# net-mpris

Control a remote D-Bus session as if it was running on the default D-Bus session bus on your machine.
D-Bus only works on \*nix systems, so this isn't going to work on Windows.


## What it does?

Once everything is correctly setup, you'll be able to control remote MPRIS supported media players
as if that remote media player was running on your machine. The media player will show up and be
controllable from your desktop environment's sound applet. Such as here we have an instance of
mps-youtube playing some music on my Raspberry Pi and me able to control it via my main machine
from my Cinnamon's default sound applet:

<img src="https://i.imgur.com/C1lfhcx.png">

mps-youtube interfaces with MPRIS specification and allows for controls such as Play/Pause, Stop,
Previous/Next track and Seeking. I can do all of these from my Cinnamon's sound applet.

One can also use multimedia keys on their keyboard (or set custom hotkeys) to control playback on the
remote media player instance.


## Setting up D-Bus and net-mpris

You'll need to go through these steps before you can use net-mpris.

Setting up the environment isn't so trivial. You'll need to make these arrangements before a machine
can mimic a remote D-Bus session.

### Server

On the main machine which will actually be running the media player (such as the Raspberry Pi in the
example in above section), you'll need to install D-Bus:
```
$ sudo apt install dbus
```

Once installed, you'll need to modify your D-Bus session configuration file in `/usr/share/dbus-1/session.conf`
so that D-Bus can bind to a TCP socket. [This answer on stackoverflow](https://stackoverflow.com/a/13275973/6554943)
tells you how to do that.

Next you'll need to set your `DBUS_SESSION_BUS_ADDRESS` environment variable telling media players to
bind to this specific session bus. You can do this for the current terminal with:
```
$ eval $(dbus-launch --auto-syntax)
```

If you get this error when running the above command, where 55556 is the port number you set in your
`session.conf` above:
```
dbus-daemon[2964]: Failed to start message bus: Failed to bind socket "*:55556": Address already in use
```

You'll need to kill the previous D-Bus session with:
```
$ fuser -k 55556/tcp
```

and then run the above command again and it shouldn't error this time.

You can now run any MPRIS supported media player from the same terminal such as:
```
$ spotify
```
or
```
$ mpsyt
```
and they are ready to be controlled from another machine via the TCP port you assigned in your D-Bus's
`session.conf`!


### Client

On the client machine (the one which will gain controls to the media player running on the server),
you'll need to install [playerctl](https://github.com/acrisci/playerctl)
([64-bit releases](https://github.com/acrisci/playerctl/releases) and
[Raspberry Pi releases](http://raspbian.raspberrypi.org/raspbian/pool/main/p/playerctl/))

and `dbus-python` from PyPI:
```
$ pip3 install dbus-python
```

If `dbus-python` complains about installing a development version, download
[the tarball](https://files.pythonhosted.org/packages/3f/e7/4edb582d1ffd5ac3c84188deea32e960b5c8c0fe1da56ce70224f85ce542/dbus-python-1.2.8.tar.gz)
and run:
```
$ pip3 install -e .
```

in the extracted directory and it should now be installed ok.

Now, let's make sure everything is setup correctly so far.

Point the D-Bus session bus on your client machine to the on running on the server:
```
$ export DBUS_SESSION_BUS_ADDRESS='tcp:host=192.168.1.5,port=55556,family=ipv4
```
(Replace the IP address and port where D-Bus is listening on the host machine)

Now run:
```
$ playerctl -l
```

If this shows the name of the media player you ran on the host machine, like it does for me:
```
mps-youtube.instance15990
```
then everything is good to go!

```
$ git clone https://github.com/ritiek/net-mpris
$ cd net-mpris
$ python3 netmpris.py 192.168.1.5 -p 55556
```
and you should have the media player in your sound applet and should also be controllable from multimedia
keys on your keyboard!


## Alternate ways?

To be honest, I feel like I'm missing something out and that something like this should already be
possible with some hacking on D-Bus itself. We probably don't need this code to act a middle man between
the local and remotely running MPRIS controller. Anyway, I've yet to discover if something like that is
possible. I'm up for all thoughts!


## License

I'm GPL fan but since most of the code here is taken from [mps-youtube](https://github.com/mps-youtube/mps-youtube)
which is GPL licensed, it makes some sense to also keep this GPL to avoid any potential conflics.
