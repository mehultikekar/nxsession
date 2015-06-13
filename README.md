# Persistent X sessions using NX

nxsession uses [NX 3](https://www.nomachine.com) and bash scripts to start and manage persistent X sessions. Think of it as screen or tmux for X.

It is better than `ssh -X` because:

1. The NX session is not killed if the network disconnects.
2. It is more responsive as it reduces number of round-trips between the client and server.

It is better than VNC because:

1. It has much lower bandwidth requirement.
2. It plays well with multi-monitor setups.

The code is just two bash scripts - [nxviewer](nxviewer) and [nxhost](nxhost) for the viewer and host respectively. At present, it has been tested on Linux host and Linux/Mac OS X viewer setups. Windows viewer is being developed.

## How to use

Say you have two machines `work` and `home`. You are at `home` and want to run programs on `work`:

```bash
home> nxviewer connect work:3
# this starts an NX session on work with windows on home.
# by default, it opens xterm from which you launch other programs.
# once you are done, suspend the session as:
home> nxviewer suspend :3
# at work, you can get back the windows as:
work> nxviewer connect :3
```

Or, if you are at `work` and intend to continue on `home` later:

```bash
work> nxviewer connect :3
# this starts and displays an NX session on work.
# at home, you can move all windows from work as:
home> nxviewer connect work:3 -f
```

## Install host and viewer on Linux (without root access)

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

The viewer needs only nxproxy and libXcomp3, while the host needs all the packages except nxproxy. For a complete host+viewer setup, simply get all packages.

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

We will remove the wrappers and move the actual executables in their place. The wrappers are probably needed for a full-fledged nxclient-nxserver setup or for x2go. Also, add [rpath](http://en.wikipedia.org/wiki/Rpath) to the executables so that they can find their libraries. Finally, copy nxviewer and nxhost scripts into the bin folder.

```bash
cd usr/bin
mv ../lib64/nx/bin/* .
rm -r ../lib64/nx
patchelf --set-rpath '$ORIGIN/../lib64' nxagent
patchelf --set-rpath '$ORIGIN/../lib64' nxproxy
cp <path to repo>/nxviewer .
cp <path to repo>/nxhost .
```

Before using, add the bin folder to PATH on the viewer. On the host, make sure the *login* shell adds the bin folder to PATH by editing the appropriate rc file (~/.bashrc, ~/.profile, ~/.tcshrc).

## Install viewer on Mac OS X

Download the X2Go Client dmg from [here](http://wiki.x2go.org/doku.php/download:start) and copy nxviewer into the same folder as the nxproxy executable and add the folder to PATH. We have tried to use only common UNIX commands that should be installed on OS X by default. If that is not the case, please open an issue.

## Notes

1. A new session executes `~/.nx/xstartup` file. If it fails, it will try `xterm`.
2. If you closed your terminal and cannot open any new programs in the session, ssh to the remote machine and run `env DISPLAY=:<display> xterm &`.
3. If you left the session connected to one machine and want to access it on another, add `-f` to force the first machine to disconnect.
4. Run `nxviewer -h` to see all available commands and options.
