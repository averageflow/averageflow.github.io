---
layout: post
title: rc.d scripting
date: 2021-12-02
description: FreeBSD uses the init system, and system services must be made as rc.d scripts
categories: ["FreeBSD", "rc.d"]
---

FreeBSD uses the init system, and system services must be made as rc.d scripts.

You can find more detailed information and examples in the handbook: [Practical rc.d scripting in FreeBSD](https://docs.freebsd.org/en/articles/rc-scripting/).

## CLI options choice for daemons

Find below the options we normally use for any `/etc/rc.d` script as daemon. See more options by in the terminal issuing:
```
man daemon
```

`-f`: Redirect standard input, standard output and standard error to `/dev/null`.  When this option is used together with any of the options related to file or syslog output, the standard file descriptors are first redirected to `/dev/null`, then stdout and/or stderr is redirected to a file or to syslog as specified by the other options.

`-S`: Enable syslog output.  This is implicitly applied if other syslog parameters are provided.  The default values are `daemon`, `notice`, and `daemon` for facility, priority, and tag, respectively.

`-P <supervisor pid file>`:  Write the ID of the daemon process into the file with given name using the `pidfile` functionality.  The program is executed in a spawned child process while the daemon waits until it terminates to keep the `supervisor_pidfile` locked and removes it after the process exits.  The `supervisor_pidfile` owner is the user who runs the daemon regardless of whether the `-u` option is used or not.

`-r`: Supervise and restart the program after a one-second delay if it has been terminated.

## Connecting a script to the rc.d framework

In a nutshell, [rcorder](https://www.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) takes a set of files, examines their contents, and prints a dependency-ordered list of files from the set to stdout. The point is to keep dependency information inside the files so that each file can speak for itself only. A file can specify the following information:

- the names of the "conditions" (which means services to us) it provides with `# KEYWORD`:
- the names of the "conditions" it requires with `# REQUIRE`:
- the names of the "conditions" this file should run before with `# BEFORE`:
- additional keywords that can be used to select a subset from the whole set of files (rcorder(8) can be instructed via options to include or omit the files having particular keywords listed.) with `# KEYWORD`:

As a rule of thumb, every custom made rc.d script should provide 1 daemon, and require `DAEMON` and `networking` to be initialized. Thus the first lines of your rc.d script will generally look something like:
```sh
#!/bin/sh
#
# PROVIDE: <name of this rc.d script>
# REQUIRE: DAEMON networking
# KEYWORD:
```

## Daemon load order

The order in which the daemons load is highly important. For this example, imagine you have an API application running with an rc.d script called `my_api_service`. It should run as a daemon and should be able to access the network.

This means that it should only load after `DAEMON` and `networking` are loaded and running.

You can verify the load order with:
```
service -e
```

The correct load order in this case then would be : 
```
service -e

/etc/rc.d/cleanvar
/etc/rc.d/ip6addrctl
/etc/rc.d/netif
/etc/rc.d/virecover
/etc/rc.d/motd
/etc/rc.d/os-release
/etc/rc.d/newsyslog
/etc/rc.d/syslogd
/etc/rc.d/my_api_service
/etc/rc.d/cron
```

## Binaries as daemons

In the case of binaries, we will use the daemon in a straightforward manner. We will run our service as `root` user since this will be running inside a jail, where `root` is the only existing user. Feel free to adjust to your use case.
By convention I always create a folder called `internalApplications` inside any jail, and place my binaries inside there.
See the example below for `my_api_service`:
```sh
#!/bin/sh
#
# PROVIDE: my_api_service
# REQUIRE: DAEMON networking
# KEYWORD:

. /etc/rc.subr

name="my_api_service"
rcvar="my_api_service_enable"
my_api_service_chdir="/root/internalApplications/myAPIWorkingDirectory/"
my_api_service_user="root"
my_api_service_command="/root/internalApplications/myAPIWorkingDirectory/app"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f -S ${my_api_service_command}"

load_rc_config $name
: ${my_api_service_enable:=no}

run_rc_command "$1"
```

## Shell scripts as daemons

Shell scripts that run utilities can also be daemonized. See the example below:
```sh
#!/bin/sh

# PROVIDE: my_script_service
# REQUIRE: DAEMON networking
# KEYWORD:
. /etc/rc.subr

name="my_script_service"
rcvar="my_script_service_enable"
my_script_service_user="root"
pidfile="/var/run/${name}.pid"
my_script_service_command="/root/internalApplications/my-script.sh"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f -S ${my_script_service_command}"

load_rc_config $name
: ${my_script_service_enable:=no}

run_rc_command "$1"
```

Hopefully you have become a little wiser about rc.d scripting and can "daemonize" anything that comes your way :)
