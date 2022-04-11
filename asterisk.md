# Asterisk PBX
### Installation
I'm installing Asterisk on my personal machine, which runs Arch Linux, so these
instructions will have some specifics that will not apply to other operating
systems.

As of this writing, Asterisk is not available as an officially maintained Arch
package, but it is available to build from source via the
[AUR](https://aur.archlinux.org/packages/asterisk). I'm building Asterisk
19.2.0-1, and I am automating the process with an AUR helper,
[yay](https://aur.archlinux.org/packages/yay). (There are many such helpers for
installing packages from the AUR; they are not required, this is just the one I
like to use.)

Some tutorials (including the one I'm following, as well as the Arch [wiki
page](https://wiki.archlinux.org/title/Asterisk#Configuration) for Asterisk)
recommend doing an initial setup with the deprecated `sip` module, citing its
ease of setup over its successor, `pjsip`. This is an issue for newer versions
of Asterisk, as `sip` was deprecated as of version 17. In order to make the
necessary change to include the `sip` module in my installation, I run the
following command:
```
$ yay --editmenu -S asterisk
```
This allows me to edit the PKGBUILD file fetched from the AUR, where I can
specify the inclusion of the `sip` module on line 174:
```
./menuselect/menuselect --disable BUILD_NATIVE --enable chan_sip
```
With this change in place, yay can then run through the rest of the
build/install process normally.

### Initial configuration
As `sip` is not included by default in Asterisk 19, a small change must be made
to `/etc/asterisk/modules.conf` -- ensure that the line `noload = chan_sip.so`
is commented out, if you plan to use `sip` as I do. Additionally, I have also
added the following to the same file to prevent any weird conflicts:
```
noload = chan_pjsip.so
noload = res_pjsip.so
```

The only additional configuration needed at this point is to define a SIP user,
and the context which will map an extension to the user. In
`/etc/asterisk/sip.conf` I have added the following:
```
[ian]
type=friend
username=ian
secret=applebees
disallow=all
allow=ulaw
host=dynamic
context=laptop
qualify=yes

[aj]
type=friend
username=aj
secret=prius
disallow=all
allow=ulaw
host=dynamic
context=laptop
qualify=yes
```

To define the context, I have added this snippet to
`/etc/asterisk/extensions.conf` which will map my users to extensions 100 and
101:
```
[laptop]
exten => 100,1,Dial(SIP/ian)
exten => 101,1,Dial(SIP/aj)
```

### Placing a test call
With some SIP users and a context configured, a test call can be made with the
use of a softphone. I used `zoiper` and `twinkle` which were both available as
AUR packages. Other possible options include Linphone and X-lite; there is
another, simply called Telephone, available for Mac.

Of course, the first step is to ensure that Asterisk is up and running. Since I
am using systemd:
```
# systemctl start asterisk
```

To connect to Asterisk once it's running:
```
# asterisk -r
```

As the `sip` module had to be explicitly included in the build process and also
enabled in `modules.conf`, it's a good idea to check to see if it is indeed
loaded. In the Asterisk console, running `module show like sip` should hopefully
produce an output that includes `chan_sip.so`, like this:
```
Module                         Description                              Use Count  Status      Support Level
app_adsiprog.so                Asterisk ADSI Programming Application    0          Running        deprecated
chan_sip.so                    Session Initiation Protocol (SIP)        0          Running        deprecated
2 modules loaded
```

Next, each SIP user can be configured and registered via softphone. The setup
will depend on which softphone is used, but it should be fairly straightforward.
I provided the credentials defined in `sip.conf` and used `localhost` for the
domain. Successful registration of SIP peers should produce an output similar to
this in the Asterisk console:
```
[Apr 10 23:51:42] NOTICE[2846]: chan_sip.c:25009 handle_response_peerpoke: Peer 'aj' is now Reachable. (1ms / 2000ms)
[Apr 10 23:51:57] NOTICE[2846]: chan_sip.c:25009 handle_response_peerpoke: Peer 'ian' is now Reachable. (1ms / 2000ms)
```

This is also verifiable by running `sip show peers` which returns this output:
```
Name/username             Host                                    Dyn Forcerport Comedia    ACL Port     Status      Description                      
aj/aj                     127.0.0.1                                D  Auto (No)  No             52119    OK (1 ms)                                    
ian/ian                   127.0.0.1                                D  Auto (No)  No             5061     OK (1 ms)                                    
2 sip peers [Monitored: 2 online, 0 offline Unmonitored: 0 online, 0 offline]
```

At this point, the only thing left to do is have one SIP peer place a call to
the other. In my softphone, I simply dialed the other party's extension, and the
call initiated. 
