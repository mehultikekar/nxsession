# Persistent X sessions using NX

nxsession is a shell script that uses [NX 3](https://www.nomachine.com) to start persistent X sessions. Think of it as screen or tmux for X.

It is better than `ssh -X` because:

1. The NX session is not killed if the network disconnects.
2. It is more responsive as it reduces number of round-trips between the client and server.

It is better than VNC because:

1. It has much lower bandwidth requirement.
2. It plays well with multi-monitor setups.

At present, it has been tested on Linux host and Linux/Mac OS X viewer setups. Windows viewer is being developed.

## How to use

Say you have two machines `work` and `home`. You are at `home` and want to run programs on `work`:

```bash
home> nxsession work:3
# this starts an NX session on work with windows on home.
# by default, it opens xterm from which you launch other programs.
# once you are done, suspend the session as:
home> nxsession :3 -s
# at work, you can get back the windows as:
work> nxsession :3
```

Or, if you are at `work` and intend to continue on `home` later:

```bash
work> nxsession :3
# this starts and displays an NX session on work.
# at home, you can move all windows from work as:
home> nxsession work:3 -f
```

## Install host and client on Linux (without root access)

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

The client needs only nxproxy and libXcomp3, while the host needs all the packages except nxproxy. For a complete host+client setup, simply get all packages.

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

Before using, add the bin folder to PATH on the viewer. On the remote host, make sure the login shell adds nxsession to PATH by editing the appropriate rc file (~/.bashrc, ~/.profile, ~/.tcshrc).

## Install client on Mac OS X

Download the X2Go Client dmg from [here](http://wiki.x2go.org/doku.php/download:start) and copy nxsession into the same folder as the nxproxy executable and add the folder to PATH.

## Notes

1. By default, a new session will start with xterm, unless a `~/.nxstartup` file exists and has execute permissions.
2. If you closed your terminal and cannot open any new programs in the session, ssh to the remote machine and run `env DISPLAY=:<n> xterm &`.
3. If you left the session connected to one machine and want to access it on another, add `-f` to force the first machine to disconnect.
4. `nxsession -h` prints its help message.
