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
```

To define the context, I have added this snippet to
`/etc/asterisk/extensions.conf` which will map my user to extension 100:
```
[laptop]
exten => 100,1,Dial(SIP/ian)
```
