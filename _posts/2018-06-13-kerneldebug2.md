---
layout: post
title: Kernel Modules Debugging Tips (2)
date: 2018-06-13
---

What if you encounter an issue with a kernel module that doesn't give feedback on what went wrong in the kernel log?

An example of a problem I faced along that line was when I got an error while trying to unload a module.

<pre>
$ sudo modprobe -r vkms
modprobe: FATAL: Module vkms is in use.
</pre>

Usually, this would have been easily resolved with *lsmod* by checking the **Used by** column.
Unfortunately, that's not possible all the time.

<pre>
$ lsmod | grep vkms
vkms                   16384  1
drm_kms_helper        172032  1 vkms
drm                   405504  4 drm_kms_helper,vkms
</pre>

For example, from above, we can see that **Used by** in the third column shows that one instance of *drm_kms_helper* is used by vkms,
but it doesn't give more details on what's using vkms.

I have been told that it's probably used by a part of the GUI manager.
To understand what exactly was using the module, I have stumbled upon an answer in this thread
[[1]](https://stackoverflow.com/questions/448999/is-there-a-way-to-figure-out-what-is-using-a-linux-kernel-module)
which explained how Ftrace utility could be used to shed light on the modules dependencies.

---
## Ftrace

Ftrace (Function Tracer) is a linux kernel utility that enables tracing kernel function calls.
Ftrace is not only helpful for debugging but also to understand how a module works by following the order of function calls.

Reading the documentation file [[2]](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/trace/ftrace.rst)
and this guide [[3]](https://lwn.net/Articles/365835/), as well as exploring the directory 
<code>/sys/kernel/debug/tracing</code> was helpful to understand how *ftrace* works and how to use it for debugging
(to access the directory, super-user privileges needed).

The following use cases give some overview on how to navigate *ftrace* files:

1) To check if tracing is enabled:
<div align="center"><code>$ cat /sys/kernel/debug/tracing/tracing_on</code></div>

2) By default, tracing is enabled in my system. To disable it:
<div align="center"><code>$ echo 0 > /sys/kernel/debug/tracing/tracing_on</code></div>

3) To check available tracers:
<pre>$ cat /sys/kernel/debug/tracing/available_tracers
hwlat blk mmiotrace function_graph wakeup_dl wakeup_rt wakeup function nop
</pre>

Other files with *available* prefix such as available_[events, filter_functions]
is helpful to learn about the available *events* and *functions* that can be traced.

4) By default *nop* is used as the *current_tracer*, to change the tracing function:
<div align="center"><code>$ echo function_graph > /sys/kernel/debug/tracing/current_tracer</code></div>

6) To clean trace buffer:
<div align="center"><code>$ echo > /sys/kernel/debug/tracing/trace</code></div>

---

## Ftrace through trace-cmd Tool

Since dealing with Ftrace through the debugfs filesystem is not that convenient,
there exists a user-space command-line tool for Ftrace that may be easier to use.

<div align="center"><code>$ sudo apt-get install trace-cmd</code></div>

For more details on trace-cmd, this guide [[4]](https://lwn.net/Articles/410200/) might be helpful. 
Moreover, checking the help manual for a specific command such as: <code>$ trace-cmd record --help</code> would give more details on the
proper usage of the command.

---

## Inspecting module dependncies with trace-cmd: 

As
[[1]](https://stackoverflow.com/questions/448999/is-there-a-way-to-figure-out-what-is-using-a-linux-kernel-module)
mentioned,  *try_module_get* && *module_put* are the ones responsible for managing a module usage count (refcount).

We can see the available events related to module load and unload by inspecting <code>available_events</code>
file in the Ftrace directory:

<pre>$ cat /sys/kernel/debug/tracing/available_events | grep module_
module:module_request
module:module_put
module:module_get
module:module_free
module:module_load
</pre>

### Use Case:

So now we can use **trace-cmd** to understand the module dependencies for *vkms*:

1- Start recording all events associated with **module** keyword (ex: module_request, module_get, etc.)
<pre>$ sudo trace-cmd record -e module </pre>

2- In another terminal, install the module again. Since the vkms module depends on other modules, I would use modprobe.
<pre>$ modprobe vkms </pre>

3- To inspect the results of the event tracing, we can stop recording and run **trace-cmd** again with the **report** command.
<pre>
$ trace-cmd report
cpus=1
        modprobe-1687  [000]   102.001424: module_load: sysimgblt 
        modprobe-1687  [000]   102.001519: module_put:  sysimgblt call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.013795: module_load: sysfillrect 
        modprobe-1687  [000]   102.013812: module_put:  sysfillrect call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.023779: module_load: syscopyarea 
        modprobe-1687  [000]   102.023792: module_put:  syscopyarea call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.030766: module_load: fb_sys_fops 
        modprobe-1687  [000]   102.030776: module_put:  fb_sys_fops call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.117198: module_load: drm 
        modprobe-1687  [000]   102.118399: module_put:  drm call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.145601: module_get:  drm call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.145674: module_get:  fb_sys_fops call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.145680: module_get:  sysimgblt call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.145687: module_get:  sysfillrect call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.145754: module_get:  syscopyarea call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.149619: module_load: drm_kms_helper 
        modprobe-1687  [000]   102.150797: module_put:  drm_kms_helper call_site=do_init_module refcnt=1
        modprobe-1687  [000]   102.160087: module_get:  drm call_site=ref_module refcnt=3
        modprobe-1687  [000]   102.160090: module_get:  drm_kms_helper call_site=ref_module refcnt=2
        modprobe-1687  [000]   102.161679: module_load: vkms 
        modprobe-1687  [000]   102.161689: module_get:  drm call_site=get_filesystem refcnt=4
        modprobe-1687  [000]   102.163020: module_put:  vkms call_site=do_init_module refcnt=1
  systemd-logind-545   [000]   102.168295: module_get:  drm call_site=cdev_get refcnt=5
  systemd-logind-545   [000]   102.168296: module_get:  drm call_site=chrdev_open refcnt=6
  systemd-logind-545   [000]   102.168297: module_get:  vkms call_site=drm_stub_open refcnt=2
  systemd-logind-545   [000]   102.168298: module_put:  drm call_site=drm_stub_open refcnt=5
</pre>
We can see from the tracing output that some other modules were loaded first (sysimgblt, sysfillrect, etc.). 
That's because modprobe checks first the module directory in <code>/lib/modules/`uname -r`</code>
and load first a set of modules that the module to be installed depends on.

<pre>
$ cat /lib/modules/$(uname -r)/modules.dep | grep vkms
kernel/drivers/gpu/drm/vkms/vkms.ko: kernel/drivers/gpu/drm/drm_kms_helper.ko kernel/drivers/gpu/drm/drm.ko kernel/drivers/video/fbdev/core/fb_sys_fops.ko kernel/drivers/video/fbdev/core/syscopyarea.ko kernel/drivers/video/fbdev/core/sysfillrect.ko kernel/drivers/video/fbdev/core/sysimgblt.ko
</pre>

---

## Resolving the Problem

It appears that, **systemd-login** (which manages user logins) [[5]](https://www.freedesktop.org/software/systemd/man/systemd-logind.service.html)
has been using an instance of *vkms* and *drm* modules as can be shown bellow:

<pre>
systemd-logind-545   [000]   102.168295: module_get: drm call_site=cdev_get refcnt=5
systemd-logind-545   [000]   102.168296: module_get: drm call_site=chrdev_open refcnt=6
systemd-logind-545   [000]   102.168297: module_get: vkms call_site=drm_stub_open refcnt=2
systemd-logind-545   [000]   102.168298: module_put: drm call_site=drm_stub_open refcnt=5
</pre>

To stop *systemd-login* from using these modules we need to disable the *graphical login manager*.

Since my machine uses **systemd** instead of **init**
[[6]](https://www.systutorials.com/239880/change-systemd-boot-target-linux/)
I can switch to a text mode by changing the *runlevel* or its equivelant keyword: *target*.

<pre>
$ cat /proc/1/status | grep Name
Name:systemd
</pre>

To inspect the current target <code>get-default</code> can be used:

<pre>
$ systemctl get-default
graphical.target
</pre>

We can disable the *graphical login manager* permenantly by changing the default target to 
*multi-user* as follows:

<div align="center"><code>
$ sudo systemctl set-default multi-user.target
</code></div>

To understand the difference between *graphical* and *multiuser* targets,
**pstree** can be used to give some context on both of them:

<pre>
$ systemctl isolate multiuser.target 
$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager─┬─2*[dhclient]
        │                └─2*[{NetworkManager}]
        ├─accounts-daemon───3*[{accounts-daemon}]
        ├─agetty
        ├─avahi-daemon───avahi-daemon
        ├─cron
        ├─cups-browsed───2*[{cups-browsed}]
        ├─cupsd
        ├─dbus-daemon
        ├─dbus-daemon───dbus-daemon
        ├─evolution-addre───5*[{evolution-addre}]
        ├─evolution-calen───5*[{evolution-calen}]
        ├─evolution-sourc───4*[{evolution-sourc}]
        ├─gnome-keyring-d───3*[{gnome-keyring-d}]
        ├─goa-daemon───5*[{goa-daemon}]
        ├─goa-identity-se───3*[{goa-identity-se}]
        ├─gvfsd───3*[{gvfsd}]
        ├─gvfsd-fuse───5*[{gvfsd-fuse}]
        ├─2*[kerneloops]
        ├─networkd-dispat───{networkd-dispat}
        ├─packagekitd───3*[{packagekitd}]
        ├─polkitd───3*[{polkitd}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─snapd───6*[{snapd}]
        ├─ssh-agent
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-resolve
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        ├─whoopsie───2*[{whoopsie}]
        └─wpa_supplicant
</pre>

<pre>
$ systemctl isolate graphical.target 
$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager─┬─2*[dhclient]
        │                └─2*[{NetworkManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─at-spi-bus-laun─┬─dbus-daemon
        │                 └─4*[{at-spi-bus-laun}]
        ├─at-spi2-registr───2*[{at-spi2-registr}]
        ├─avahi-daemon───avahi-daemon
        ├─cron
        ├─cups-browsed───2*[{cups-browsed}]
        ├─cupsd
        ├─dbus-daemon
        ├─dbus-daemon───dbus-daemon
        ├─evolution-sourc───3*[{evolution-sourc}]
        ├─gdm3─┬─gdm-session-wor─┬─gdm-x-session─┬─Xorg───{Xorg}
        │      │                 │               ├─dbus-daemon
        │      │                 │               ├─gnome-session-b─┬─gnome-shell─┬─ibus-daemon─┬─ibus-dconf───{ibu+
        │      │                 │               │                 │             │             └─3*[{ibus-daemon}]
        │      │                 │               │                 │             └─11*[{gnome-shell}]
        │      │                 │               │                 ├─ssh-agent
        │      │                 │               │                 └─4*[{gnome-session-b}]
        │      │                 │               └─2*[{gdm-x-session}]
        │      │                 └─2*[{gdm-session-wor}]
        │      └─3*[{gdm3}]
        ├─gnome-keyring-d───3*[{gnome-keyring-d}]
        ├─goa-daemon───4*[{goa-daemon}]
        ├─goa-identity-se───3*[{goa-identity-se}]
        ├─gvfsd───2*[{gvfsd}]
        ├─gvfsd───3*[{gvfsd}]
        ├─gvfsd-fuse───5*[{gvfsd-fuse}]
        ├─2*[kerneloops]
        ├─networkd-dispat───{networkd-dispat}
        ├─plymouth
        ├─plymouthd
        ├─polkitd───3*[{polkitd}]
        ├─pulseaudio───2*[{pulseaudio}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─rtkit-daemon───2*[{rtkit-daemon}]
        ├─snapd───6*[{snapd}]
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-resolve
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        ├─udisksd───5*[{udisksd}]
        ├─upowerd───3*[{upowerd}]
        ├─whoopsie───2*[{whoopsie}]
        └─wpa_supplicant
</pre>

---

If we reboot now and test again modprobe with trace-cmd, we can see that systemd-login no longer uses *drm* or *vkms*.

<pre>$ trace-cmd report
cpus=1
        modprobe-1101  [000]   523.667393: module_load: sysimgblt 
        modprobe-1101  [000]   523.667459: module_put:  sysimgblt call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.674594: module_load: sysfillrect 
        modprobe-1101  [000]   523.674605: module_put:  sysfillrect call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.680929: module_load: syscopyarea 
        modprobe-1101  [000]   523.680937: module_put:  syscopyarea call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.686194: module_load: fb_sys_fops 
        modprobe-1101  [000]   523.686203: module_put:  fb_sys_fops call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.771319: module_load: drm 
        modprobe-1101  [000]   523.772248: module_put:  drm call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.799666: module_get:  drm call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.799739: module_get:  fb_sys_fops call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.799745: module_get:  sysimgblt call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.799751: module_get:  sysfillrect call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.799818: module_get:  syscopyarea call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.803201: module_load: drm_kms_helper 
        modprobe-1101  [000]   523.804125: module_put:  drm_kms_helper call_site=do_init_module refcnt=1
        modprobe-1101  [000]   523.813720: module_get:  drm call_site=ref_module refcnt=3
        modprobe-1101  [000]   523.813723: module_get:  drm_kms_helper call_site=ref_module refcnt=2
        modprobe-1101  [000]   523.815070: module_load: vkms 
        modprobe-1101  [000]   523.815081: module_get:  drm call_site=get_filesystem refcnt=4
        modprobe-1101  [000]   523.817306: module_put:  vkms call_site=do_init_module refcnt=1
</pre>

And the *Used by* entry for vkms is zero now, so we can unload the module vkms without any complains \o/
<pre>
$ lsmod | grep vkms
vkms                   16384  0
drm_kms_helper        172032  1 vkms
drm                   405504  3 drm_kms_helper,vkms
</pre>


---

### Refrences:
[1] [Stack Overflow: Is there a way to figure out what is using a Linux kernel module?](https://stackoverflow.com/questions/448999/is-there-a-way-to-figure-out-what-is-using-a-linux-kernel-module)

[2] [Kernel Documentation: FTrace - Function Tracer](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/trace/ftrace.rst)

[3] [Debugging the kernel using Ftrace - part 1](https://lwn.net/Articles/365835/)

[4] [trace-cmd: A front-end for Ftrace](https://lwn.net/Articles/410200/) 

[5] [Systemd-logind](https://www.freedesktop.org/software/systemd/man/systemd-logind.service.html)
 
[6] [How to Change Systemd Boot Target on Linux](https://www.systutorials.com/239880/change-systemd-boot-target-linux/)
