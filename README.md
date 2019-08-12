# screenlock-handler

`screenlock-handler` is intended to handle systemd-logind-screenlock- and suspend-events and lock your screen with a screenlocker of your choice.

## Usage

`screenlock-handler [screenlock-command]`

If `screenlock-command` is ommitted, `screenlock-handler` will instead use the freedesktop.org-screensaver, which is available via its DBus API, if implemented by something.

It will trigger both on lock-signals as well as locking the system on suspend.

If run inside a session (e.g. via `screenlock-handler &` in your window manager config, shell-profile-files or similar), `screenlock-handler` will lock this specific session, whereas when run outside of a session (e.g. via a systemd-user-unit) it will lock all sessions, including those created after it was started.

You can then run `loginctl lock-session` (or `loginctl lock-sessions`, if you want them all and have sufficient privileges) anywhere from your system and your session will get locked.

## Requirements

  * `systemd-logind` or API-equivalent (likely given when using a modern linux)

One of:

  * screenlocker that implements the freedesktop.org-API
  * screenlock command, that exits once the screen is locked<sup>[1](#Footnotes)</sup> (for example `i3lock`)

## running as a systemd-user unit

The recommended way of running `screenlock-handler` is as a systemd-user unit, as this provides automated restart capabilities should `screenlock-handler` fail. Given that a screenlock is typically intended as a security feature, this is a reasonable precaution to take.

To do so, create a file `~/.config/systemd/user/screenlock-handler.service` containing

    [Unit]
    Description='Lock screen'

    [Service]
    Environment=DISPLAY=:0
	Restart=always
    ExecStart=/path/to/screenlock-handler [screenlock-command]

and make sure to start it once the graphical environement is available (as we need to pass the `DISPLAY`-variable to the environement such that it is inherited by our screenlock-handler and the screenlocker in the process, which is not particularly beautiful, admittedly, but it works and allows running inside of a systemd-user unit with a `Restart=always` directive).

Now ensure that you start this unit, either by starting `systemctl --user start screenlock-handler.service` or by adding an appropriate install-section to the unit and enabling it.


## Credit

`screenlock-handler` was parented by [grawity's systemd-lock-handler](https://github.com/grawity/code/blob/master/desktop/systemd-lock-handler).

## FAQ

### What's the difference to xss-lock?

You might be well aware of the tool xss-lock.
`screenlock-handler` is a superset of xss-lock, as xss-lock can only be used inside of a session, whereas `screenlock-handler` can also be used outside of a session, such as a systemd-user unit.
(`screenlock-handler` was actually written exactly to fix `xss-lock`'s incapability to be used in systemd-user units.)


## Footnotes

<sup>1</sup>: If it is used with a screenlock-command, it requires a command that will exit once the screen is locked. The reason for this is that `screenlock-handler` will delay suspend until the screen is locked to ensure that the suspended system is actually locked before going to sleep instead of risking that the screenlocker only takes effect after the system woke up again (visible through an open desktop when waking up before the screenlock suddenly kicks in). Therefore, `screenlock-handler` needs to tell when the screenlocking-procedure is completed so that this lock can be released. Currently, the screenlock-command exiting is the only reliable and sufficiently general way to achive this. Currently, `i3lock` (used *without* the `-n` or `--nofork` flag) is the only screenlocker known to the author that implements this behavior. If the screenlock does not provide this behavior and its main process stays present, the sleep inhibition is released once its standard timeout is reached.[^](#Requirements)

