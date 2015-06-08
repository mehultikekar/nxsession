# Replacement for VNC, ssh -X

nxsession is a shell script that uses [NX 3](https://www.nomachine.com) libraries to start persistent X sessions. Think of it as screen/tmux for X.

## How to use

Say you have two machines `work` and `home`. You are at `home` and want to run programs on `work`:

```bash
home> nxsession 3 -h work
# this will start an xterm from which you can launch other programs
# once you are done, suspend the session as:
home> nxsession 3 -s
# at work, you can get back the windows as:
work> nxsession 3
```

If you left the session running other one machine and want to access it on another, add `-f` to force the first machine to disconnect.

## Install without sudo access

[x2go](http://wiki.x2go.org/doku.php/download:start) maintains compiled binaries for all NX libraries. For example, rpm's for RHEL are [here](http://packages.x2go.org/epel) Download rpm’s for the appropriate version of RHEL.

1. libNX_X11-6
2. libNX_Xcomposite1
3. libNX_Xdamage1
4. libNX_Xdmcp6
5. libNX_Xext6
6. libNX_Xfixes3
7. libNX_Xinerama1
8. libNX_Xpm4
9. libNX_Xrandr2
10. libNX_Xrender1
11. libNX_Xtst6
12. libXcomp3
13. libXcompext3
14. libXcompshad3
15. nxagent
16. nxproxy

The `home` computer needs only nxproxy and libXcomp3. `work` needs all the packages.

Extract the rpm into a folder:

```bash
mkdir nx
cd nx
for rpm in <downloads>/*.rpm; do
    rpm2cpio $rpm | cpio -idmv
done
```

You should get folders inside `nx/` as follows:

```
etc -> unused
usr ->
  bin -> wrappers for executables
  lib64 -> libraries
    nx ->
      bin -> actual executables
  share -> man pages, docs, etc.
```

We will remove the wrappers and move the actual executables in their place. The wrappers are probably needed for a full-fledged nxclient-nxserver setup or for x2go. Also, add [rpath](http://en.wikipedia.org/wiki/Rpath) to the executables so that they can find their libraries. Finally, copy the provided [nxsession](nxsession) script into the bin folder.

```bash
cd usr/bin
mv ../lib64/nx/bin/* .
rm -r ../lib64/nx
patchelf --set-rpath '$ORIGIN/../lib64' nxagent
patchelf --set-rpath '$ORIGIN/../lib64' nxproxy
cp <path to repo>/nxsession .
```

Before using, add the bin folder to PATH.

## Notes

1. By default, a new session will start with xterm, unless a `~/.nxstartup` file exists and has execute permissions.
2. If you closed your terminal and cannot open any new programs in the session, ssh to the remote machine and run `env DISPLAY=:<n> xterm &`.
3. Read comments in the [nxsession](nxsession) script for its usage.
