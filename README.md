# Abulia
abulia is a bash-script that aims to assist exploitation of netcat-based
remote code execution.

abulia requires:
- netcat to be installed on both the host and target system; the target system's netcat implementation will need to support the `-e` flag.
- read-write access on the current directory on the remote system
- the target system to be executing `netcat -e /bin/sh -vlp 8888`; this will have to be done by utilising a different exploit. (any port may be used; `8888` is used as an example here.)

## Usage
### Regular Invocation
The simplst way to invoke abulia is:
```bash
./abulia 192.0.2.0 8888
```
abulia will attempt to connect to the given host before:
- copying over `payloads/abuliarc` to `$PWD/.abulia/abuliarc` on the host system over port 8889
- open an additional netcat instance on the host system in order to connect over port 8890
- performs a reverse shell upgrade over the newly established netcat connection; this instance then sources the `abuliarc` on the host system.

### Invoke Payloads
Any additional options on the command line will be interpreted as payloads:
```bash
./abulia 192.0.2.0 8888 whoami binpack/binpack-x86_64 +binmake
```
This will:
- transfer `payloads/whoami` to `$PWD/.abulia/whoami` on the remote system
- transfer `payloads/binpack/binpack-x86_64` to `$PWD/.abulia/binpack-x86_64` on the remote system
- transfer `payloads/binmake` to `$PWD/.abulia/binmake` on the remote system **and then execute it**.
This should provide a pretty clear overview of how payloads work.
*At the moment, abulia only searches for payloads relative to the `payloads/`
directory; this has yet to be fixed.*

**Notice:** abulia uploads payloads in chronological order from left to
right; if payload B depends on payload A, then `./abulia 192.0.2.0 8888 +B A` will produce erroneous results.

abulia uses `dd` and `netcat` for file transmission, and can handle payloads of arbitrary size (including 200MiB+ wordlists).

## Payload Overview
### understand
the `understand` payload shows a fancy header displaying information such as disk usage, `uname`, SSH configuration, etc.

TODO:
- display more useful information such as:
- systemctl output;
- autodetect users and groups on the system;
- autodetect security features such as `hidepid` and non-readable `/var/log` files;

### binmake + binpack
The `binpack` files found in `payloads/binpack/` are `.tar.xz` files
containing precompiled static binaries for x86_64, x86 and ARM architecture
GNU/Linux operating systems. They were stolen from [https://github.com/andrew-d/static-binaries/](andrew-d/static-binaries).

`binmake` depends on one or more of each `binpack-$ARCH` to have been transferred to the target system.
`binmake` will create a new subdirectory `$PWD/.abulia/binpack-$ARCH` before unpacking the statically linked binaries found in the pack.
It will then modify the `$PATH` variable in `$PWD/.abulia/abuliarc`.

## Todo
abulia has several issues that need to be fixed:
- abulia does not yet support other ways of attaining a reverse shell (nmap, lua, irb, etc.)- it currently relies on `python` and `pty`.
- abulia does not yet support other ways of initiating connections other than netcat; no support for `socat` or gnu/awk+network-based shelling.
- abulia produces one or two lines of garbage on the screen when achieving a reverse shell; this has yet to be fixed.
- abulia relies on some ugly hacks and `timing` to keep working; this is ugly and should ideally be fixed.
- exiting the abulia shell renders your terminal garbage, requiring you to close it.
