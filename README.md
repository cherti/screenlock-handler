# screenlock-handler

`screenlock-handler` is intended to handle logind-screenlock- and suspend-events and lock your screen with a screenlocker of your choice.

## Usage

`screenlock-handler [screenlock-command]`

If `screenlock-command` is ommitted, `screenlock-handler` will instead use the freedesktop.org-screensaver, which is available via DBus under `org.freedesktop.ScreenSaver`, if provided by any software.

If run inside a session (e.g. via `screenlock-handler &` in your window manager config or shell-profile-files or similar), `screenlock-handler` will lock this specific session, whereas when run outside of a session (e.g. via a systemd-user-unit) it will lock all sessions, including those created after it was started.

It will trigger both on lock-signals as well as locking the system on suspend.

## Requirements

`screenlock-handler` requires either a DBus-available  screensaver or a  provided screenlock command.
If it is used with a screenlock-command, it requires a command that will exit once the screen is locked. The reason for this is that `screenlock-handler` will delay suspend until the screen is locked to ensure that the suspended system is locked instead of the screenlocker taking effect only after it woke up.
Therefore, `screenlock-handler` needs to tell when the screenlocking-procedure is completed so that this lock can be released. Currently, the screenlock-command exiting is the only reliable and sufficiently general way to achive this.

Screenlockers that implement this behavior, as far as the author is aware are:

  * `i3lock` (used *without* the `-n` or `--nofork` flag)

Furthermore, as screenlock-handler binds to the `systemd-logind`-API, it requires either `logind` or a `logind` replacement.


## running as a systemd-user unit

The recommended way of running `screenlock-handler` is as a systemd-user unit, as this provides automated restart capabilities should `screenlock-handler` fail. Given that a screenlock is typically intended as a security feature, this is a reasonable precaution to take.

To do so, create a file `~/.config/systemd/user/screenlock-handler.service` containing

    [Unit]
    Description='Lock screen'

    [Service]
    Environment=DISPLAY=:0
	Restart=always
    ExecStart=/path/to/screenlock-handler [screenlock-command]

and make sure to start it once the graphical environement is available (as we need to pass the `DISPLAY`-variable to the environement such that it is inherited by our screenlock-handler and the screenlocker in the process)

Now ensure that you start this unit, either by starting `systemctl --user start screenlock-handler.service` or by adding an appropriate install-section to the unit and enabling it.


## FAQ

### What's the difference to xss-lock?

You might be well aware of the tool xss-lock.
`screenlock-handler` is a superset of xss-lock, as xss-lock can only be used inside of a session, whereas `screenlock-handler` can also be used outside of a session, such as a systemd-user unit.
(`screenlock-handler` was actually written exactly to fix `xss-lock`'s incapability to be used in systemd-user units.)
