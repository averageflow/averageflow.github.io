---
layout: post
title: Practical rc.d scripting and Go
date: 2021-08-31
---

FreeBSD uses the init system, and system services must be made as rc.d scripts.

You can find more detailed information and examples in the handbook: [Practical rc.d scripting](https://docs.freebsd.org/en/articles/rc-scripting/).

## CLI options choice for daemons

Find below the options we normally use for any `/etc/rc.d` script as daemon. See more options by in the terminal issuing:
man daemon

`-f`: Redirect standard input, standard output and standard error to `/dev/null`.  When this option is used together with any of the options related to file or `syslog` output, the standard file descriptors are first redirected to `/dev/null`, then `stdout` and/or `stderr` is redirected to a file or to `syslog` as specified by the other options.

`-S`: Enable `syslog` output.  This is implicitly applied if other `syslog` parameters are provided. The default values are `daemon`, `notice`, and `daemon` for facility, priority, and tag, respectively.

`-P` <supervisor pid file>:  Write the ID of the daemon process into the file with given name using the `pidfile` functionality.  The program is executed in a spawned child process while the daemon waits until it terminates to keep the `supervisor_pidfile` locked and removes it after the process exits.  The `supervisor_pidfile` owner is the user who runs the daemon regardless of whether the `-u` option is used or not.

`-r`: Supervise and restart the program after a one-second delay if it has been terminated.
Connecting a script to the rc.d framework

In a nutshell, `rcorder` takes a set of files, examines their contents, and prints a dependency-ordered list of files from the set to stdout. The point is to keep dependency information inside the files so that each file can speak for itself only. A file can specify the following information:

* the names of the "conditions" (which means services to us) it provides with `# KEYWORD`:

* the names of the "conditions" it requires with `# REQUIRE`:

* the names of the "conditions" this file should run before with `# BEFORE`:

* additional keywords that can be used to select a subset from the whole set of files (`rcorder` can be instructed via options to include or omit the files having particular keywords listed.) with # KEYWORD:

As a rule of thumb, every custom made rc.d script should provide 1 daemon, and require `DAEMON` and `networking` to be initialized. Thus the first lines of your rc.d script will generally look something like:

```sh
#!/bin/sh
#
# PROVIDE: <name of this rc.d script>
# REQUIRE: DAEMON networking
# KEYWORD:
```

## Daemon load order

The order in which the daemons load is highly important. For this example, our `my_service` rc.d script should run as a daemon and should be able to access the network.

This means that it should only load after `DAEMON` and `networking` are loaded and running.

You can verify the load order with:

```sh
service -e
```

The correct load order in this case then would be: 

```sh
service -e

/etc/rc.d/cleanvar
/etc/rc.d/ip6addrctl
/etc/rc.d/netif
/etc/rc.d/virecover
/etc/rc.d/motd
/etc/rc.d/os-release
/etc/rc.d/newsyslog
/etc/rc.d/syslogd
/etc/rc.d/my_service
/etc/rc.d/cron
```

## Go applications as daemons

In the case of Go binaries, I will use the daemon in a straightforward manner. See  the example below for one of my Go applications, running as a compiled binary. I assume `/root/internalApplications/my-service-folder` is the folder where the service is located and should be running from:

```sh
#!/bin/sh
#
# PROVIDE: my_service
# REQUIRE: DAEMON networking
# KEYWORD:

. /etc/rc.subr

name="my_service"
rcvar="my_service_enable"
my_service_chdir="/root/internalApplications/my-service-folder"
my_service_user="root"
my_service_command="/root/internalApplications/my-service-folder/app"
pidfile="/var/run/${name}.pid"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f -S ${my_service_command}"

load_rc_config $name
: ${my_service_enable:=no}

run_rc_command "$1"
```

 
## Shell scripts as daemons

Shell scripts that run utilities can also be daemonized. See the example below I use for Jaeger:

```sh
#!/bin/sh

# PROVIDE: jaeger_tracer
# REQUIRE: DAEMON networking
# KEYWORD:
. /etc/rc.subr

name="jaeger_tracer"
rcvar="jaeger_tracer_enable"
jaeger_tracer_user="root"
pidfile="/var/run/${name}.pid"
jaeger_tracer_command="/root/internalApplications/run-jaeger.sh"
command="/usr/sbin/daemon"
command_args="-P ${pidfile} -r -f -S ${jaeger_tracer_command}"

load_rc_config $name
: ${jaeger_tracer_enable:=no}

run_rc_command "$1"
```

## Conclusion

Once the rc.d script is created then you should make it executable with 

```sh
chmod +x /etc/rc.d/my_service
```

and you can then use it as a system service whose output is in `/var/log/messages`.

```sh
service my_service status
service my_service enable
service my_service start
service my_service restart
service my_service stop
```

