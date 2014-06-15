[RHEL  7 is out][1], and ships with systemd, and CentOS 7 will be doing the
same shortly.. [Debian][2] and [Ubuntu][3] are planning it for their future
releases. Soon, systemd will be a de facto standard for new Linux distro
releases. But the chances of all your users being on new releases any time in
the near future could be pretty low. ([One poor soul][4] still has customers on
RHEL 4.)

So you have some choices. If you want to use any of systemd's new features,
you'll have to either abandon the users who can't or won't upgrade, or maintain
init stuff for both SysV and systemd. Or, you can use `dshimv`.

`dshimv` lets you plop your systemd service unit file into your `/etc/init.d`
just as if it were a regular SysV init script by just adding one extra line to
it.

# How it works

Any Linux developer is at least a little familiar with `#!` - two magic
characters at the beginning of a script to tell it what interpreter to use. But
nothing limits it to scripts; all `#!` does is run whatever command you give it
adding file name as an argument, followed by whatever arguments you gave when
you executed the file (So if `foo.py` has `#!/usr/bin/env python`,
`./foo.py bar` runs `/usr/bin/env python foo.py bar`). That command can do
whatever it wants with the file.

`dshimv` is a program that takes a systemd service unit file and an action. It
parses the file and gets the command associated with that action, and runs it.
So if you take a systemd service unit file, and throw a `#!` at the top
pointing to `dshimv`, you now have an executable that acts just like a SysV
Init script.

# How to use it

1. Install `dshimv` by placing the `dshimv` file in `/usr/bin`.
2. Add `#!/usr/bin/dshimv` to the first line ofyour systemd service unit file
3. Put that unit file in `/etc/init.d`
4. `chown root:root /etc/init.d/$YOUR_FILE; chmod 0700 /etc/init.d/$YOUR_FILE`
5. `service $YOUR_FILE start`

# Limitations

Obviously, anything that SysV Init isn't able to handle, `dshimv` can't do
either. You can use `dshimv` with Upstart, but while Upstart does have some
features beyond SysV that it shares with systemd, `dshimv` doesn't make any
attempt to expose that.

Additionally, `dshimv` currently uses Python's `ConfigParser` to parse
files, and that may or may not have some differences from how systemd parses in
some edge cases. For simplicity, `dshimv` also uses simple shell execution of
the specified commands, rather than the limited subset that systemd does. This
means `dshimv` is a bit more permissive about what's valid than systemd.
Finally, there is almost certainly a performance detriment to using `dshimv`
over native scripts, though only when starting/stopping/etc. a service.

[1]: http://developerblog.redhat.com/2014/06/10/red-hat-enterprise-linux-7-now-generally-available/
[2]: https://lwn.net/Articles/585363/
[3]: http://www.markshuttleworth.com/archives/1316
[4]: http://www.reddit.com/r/linux/comments/27sbwc/red_hat_enterprise_linux_7_now_generally_available/ci3x348
