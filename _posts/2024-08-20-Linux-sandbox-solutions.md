---
layout: post
author: Wei
title:  "GNU/Linux Sandboxing - A Brief Review"
summary: An incomplete review of the sandbox solutions on the GNU/Linux operating system.
categories: system-security
---

**Authors:** &nbsp;&nbsp; • Wei &nbsp;&nbsp; • Shawn Chang &nbsp;&nbsp;&nbsp;&nbsp;
**Revision:** 0.2

**Abstract:**
> _In the modern operating system, application sandbox has become an effective
security mechanism to confine the permissions of run-time programs, thereby
reducing the risk that the other applications and the core system components in
the system are affected by the program's malicious behaviors. Although the
development of sandbox solutions on the GNU/Linux operating systems falls
relatively behind compared to some proprietary systems and mobile operating
systems, many efforts have already been made by the communities, and it does
not lack practicable productions among them. This paper gives an incomplete
review of existing sandbox solutions on the GNU/Linux operating system, which
only covers the often-used or commonly-studied solutions. We hope it can be a
reference to the GNU/Linux communities, including users, package maintainers,
and application developers._

|

## Introduction

On operating systems, the sandbox is a common technique to secure the system
from being affected by application vulnerabilities by creating a restricted and
controlled environment for the runtime application, thereby isolating and
limiting the program from accessing the filesystem, system calls, and other
system resources outside of the application's running environment [^b2016sand].
The sandbox protects the system resource under attack from being accessed and
tampered with when exploitable vulnerabilities occur in the application. It
also prevents sensitive data from being stolen by the potential malicious
behaviors of the application, intentionally or unintentionally.

Regarding the GNU/Linux platform, although there are already a number of
sandbox solutions, some security experts think it lacks an effective and widely
applied implementation [^m2024linu] [^s2020rema]. This article is a general but
not complete survey of the sandbox technique on the GNU/Linux platform. It
first introduces the interfaces that need to be restricted by an application
sandbox. Then, existing sandbox solutions on the GNU/Linux operating systems
are presented and discussed. Also, there are some discussions about the
weaknesses of the sandbox on GNU/Linux throughout the text. 


## Sandbox Interfaces

Sandbox interfaces are system and hardware resources whose access permission is
to be restricted in a sandbox solution. With a filtering mechanism, the sandbox
sits between these resources outside and the application inside, thereby
allowing or disallowing access behaviors according to a set of pre-defined or
customized policies. The design of the sandbox policy follows the
least-privilege and minimized access permission principle. Usually, the
customizable policy makes it possible users, application administrators, or
developers to adjust the policy afterward according to the requirements in the
actual scenarios.

This section lists the commonly existing interfaces managed by the sandboxes on
the GNU/Linux platform and the general practices to control and restrict the
program from accessing them.

### Filesystem Access

The filesystem access, including directories and files, should be restricted to
prevent the application from reading or writing to files containing critical
and sensitive system information. By writing to sensitive files like executable
or configuration files, the attacker can inject malicious code or the backdoor
and may followingly obtain further privileges or even root permission. In
addition, some of these sensitive files, such as `/etc/machine-id`
[^s2024mach], can be used to track the user and threaten the user's privacy.

The restriction of filesystem access can be implemented by blocking the
application directly, as most MAC tools do, or creating a virtual filesystem
environment where only necessary directories or files have been mapped.

### System Call

System calls are kernel interfaces exposed to the userspace called by the
application to execute critical functions, including memory managing, file
system operating, process controlling, and other kernel-level tasks. There are
more than 400 system calls for kernel version 5.14 on all system architectures
[^t2024sysc] . Usually, an application only uses a small set of system calls.
It is necessary to limit the access of the system calls to avoid the unused
ones being misused by potential vulnerabilities.

Seccomp is a security mechanism implemented in the kernel to restrict the
application process from accessing unnecessary system calls. Only the very
basic system calls such as `read()`, `write()`, `_exit()`, and `sigreturn()`
are allowed by defaut. Seccomp-bpf is an extension of Seccomp, which provides a
configurable filter design based on the Berkeley Packet Filter (BPF) rule.
Seccomp and Seccomp-bpf have been integrated and utilized by many sandbox
solutions on the GNU/Linux platform.

### Network 

A sandbox solution should be able to put the network access in the control
based on the application status or pre-defined policies. On the GNU/Linux
platform, firewall tools such as iptables and nftables can set rules based on
the process ID or user ID. For example, iptables carries out the `-m owner` option
with `--pid-owner`, `--uid-owner`, and `--gid-owner` in the rule configuration.
A higher-level sandbox solution can work similarly to an application firewall
on the network restriction.

Another viable approach is to restrict network access through network
namespace, which provides a way to create virtual networks for the separate
applications, thereby easily controlling their network accessing behaviors.

### Device

Controlling the access permissions to the devices by the application, such as
hard drive, microphone, and digital camera, is a critical sandboxing interface
considering the user's security and privacy. On GNU/Linux, considering all
devices except the network devices and the video adapter present as device
files under /dev/, the access to the devices could be controlled by setting the
permission of the device file in terms of reading, writing, and executing for
specified users and groups. During the system loading process, the default
permissions of the devices are set by udev rules. Moreover, the cgroups feature
[^t2024cgro] in the Linux kernel is also a viable tool to control and isolate
the application processes from accessing some devices. It is used by many
container solutions.

### Process

As one of the purposes of the application sandbox, the process in a sandbox
should be restricted to access or see other processes outside the box.

Inter-Process Communication (IPC) is an often-used mechanism by the operating
systems to deliver and synchronize the information between different processes.
The GNU/Linux operating system could create boundaries between different groups
of processes using IPC Namespace [^t2024ipcn]. Also, PID Namespace [^t2024pidn]
isolates process IDs. Each PID namespace has a dedicated environment, so two
processes in different PID namespaces may have the same process ID number. In
addition, cgroups is another powerful tool in isolating the application
processes.

As one of the commonly used IPC mechanisms, D-Bus is widely used for desktop
applications. D-Bus is designed as a bus system that provides inter-process
communication and process lifecycle management [^f2022dbus], and it has been
available on many often-used applications, such as Gnome Nautilus, Network
Manager, Dolphin, and KMail [^f2013dpro]. Therefore, as a sandbox solution, it
is necessary to manage the access permission to D-Bus, which can be done by
editing the configurations of the `<policy>` section in the file
`/usr/share/dbus-1/session.conf` [^f2023ddae] or modifying D-Bus service files
under the directory `/usr/share/dbus-1/services/` [^f2024ffaq].

### Window System

By design, the GUI applications running on the X Window system are able to
obtain the input of other applications without any restriction. It opens a risk
that a keylogger running behind may secretly steal sensitive information typed
by users.

Lauching different applications in separated X Window sessions can mitigate
this issue but it brings further complexity to the sandbox configuration. A
native solution is to apply the window systems based on the Wayland protocol
[^h2024wayl], which provides the isolation between GUI applications by design
[^d2024wayl]. However, as a prerequisite, the desktop environment should
support the Wayland protocol as well as the applications running on it.

Major desktop environments, including Gnome [^g2017wayl] and KDE [^k2022wayl],
have integrated their native Wayland support as the default delivery option.
XFCE's Wayland support is still working in progress [^x2024wayl]. On Gnome,
most of the commonly used native applications have been covered under the
Wayland, such as Gnome Terminal, Evince, Nautilus, etc. Also, LibreOffice and
web browsers, such as Mozilla Firefox and Chromium, have already implemented
native Wayland support.

### Audio

The audio service on the GNU/Linux platform, namely PulseAudio, follows a
user-centric security model, so it does not provide any isolation mechanisms
[^f2021puls], which poses a security concern as applications can do things like
snooping on another application's audio content, accessing the microphone and
recording the sound input, loading or unloading server modules, snooping on the
audio server's shared memory pool, and so on.

As an alternative to PulseAudio, Pipwire addresses these problems by designing
an access control mechanism in the product [^p2024pacc]. It also officially
supports the Flatpak applications [^p2024pipw] in which the sandboxing solution
is available. So far, most of the major GNU/Linux distributions have migrated
their sound system to Pipewire.


## Linux Sandbox Solutions

### Dedicated Sandbox Solutions:<br> Firejail, Bubblewrap, and Minijail

There is never a lack of arguments in the Open Source communities regarding the
weaknesses of sandbox solutions in the GNU/Linux platform. In an article
[^m2024linu], Madaidans points out that the GNU/Linux operating system is short
of an efficient sandboxing mechanism compared with other operating systems. The
article mentions two GNU/Linux sandboxing mechanisms in terms of Flatpak and
Firejail. However, Flatpak is not a dedicated sandboxing solution. It is far
more than that. As defined, Flatpak is an application building, managing, and
distributing tool on the GNU/Linux distributions platform [^f2024infl], which
employs the sandbox tool, namely Bubblewrap, to secure the applications that
have been built and packaged [^f2024undr]. In the same article, the author
brings the Bubblewrap up as a comparison to the Firejail and gives credit to
its less attack surface design.

As stated by the author, the problem with the bubblewrap is its learning curve,
which means it is difficult to learn and deploy in practice because of its
minimalized design. However, this problem can be addressed by developing the
community rule set and wrapper programs on a higher layer. In contrast, the
shortage of another sandboxing tool, Firejial, is severe. The main problem is
it uses SUID to start the sandboxed applications, which leaves the risk of
privilege escalation.

Despite the issue, in practice, Firejail still provides a relatively high level
of protection to the system by mitigating malicious behavior from the
application inside the box, especially when most of the threats derive from the
application installed according to one's threat model. Firejail covers most
sandbox interfaces using various mechanisms, including Seccomp-bpf, Linux
namespaces, Linux capabilities, cgroup, D-bus filtering, etc. Its user-friendly
command-line interface and configuration options make Firejial easier to accept
by policy developers and normal users, thereby increasing the overall security
of the running environment.

Minijail is another dedicated open-source sandbox developed by Google for
applications running on ChromeOS and Android [^g2024minj]. Like Firejial, it
provides a confinement environment utilizing Seccomp and Linux namespace.
Still, it has the same issue with Firejail, which requires root privilege to
launch the application. Additionally, as the purpose of its development,
Minijail is only officially available for ChromOS and Android as it depends on
libchrome [^m2024mgit]. Although it is possible to build the binary from the
code on other GNU/Linux systems [^g2022hgit], there is no official package
available on most desktop GNU/Linux distributions till now.

In general, the problems with these sandbox soluitions described above have
already been noticed by the community developers and discussed broadly. Many
community projects have been started in trying to resolve these issues on
Bubblwrap and Firejail. For instance, the Bubblejail sandbox based on
Bubblewrap [^b2024bgit] is an attempt to be there as an alternative to
Firejail.  Meanwhile, the Firejail community is talking about the unprivileged
sandboxing without SUID and the splitting of the SUID part of functions into a
separate binary [^u2024ufir].

### Sandbox Enabled by Package Management

Some package management systems introduced their built-in sandbox solutions
based on existing techniques for privilege confinement. Snap [^c2022intr],
managed by Canonical and used on Ubuntu, implements the sandboxing using
AppArmor [^c2023seca] and Seccomp, whereas the cross-platform package
management application Flatpak [^f2024flat] involves Bubblewrap [^f2024hood] as
mentioned before. AppImage does not have a built-in mechanism, but the sandbox
can be created using Firejail with the `--appimage` option.

By implementing the sandbox solution in package management, the complex works
of making policies are transfered from the application users to the package
maintainers who have better knowledge and understanding of the application. It
helps to create more reasonable policies, and more essentially, these policies
can be updated along with the application.

The integrated sandbox solution also reduces the deployment cost and increases
the coverage of sandboxed applications because they are usually enabled by
default.

In addition, in contrast to the conventional packaging systems such as APT and
RPM, often this new kind of system has the mechanism to put the dependencies,
usually, the shared libraries used by the application, into the packages
[^s2023part] [^f2024depe], thereby leaving less interface of library calling to
the outside of the package. This feature is helpful in creating the sandbox
virtual environment.

Despite that, the problem within the package management system is worth being
concerned about. A study from Dunlap _et al._ in 2022 [^t2022astu] discovered
that package maintainers tend to follow the least-privilege principle on
policymaking and the security from the system level is improved, but only 58.3%
of Flatpak applications and 90.1% of Snap applications defined proper sandbox
policies. In other words, only a part of the applications had been covered
under the protection of their sandbox mechanisms. Indeed, users could override
existing policies with their own ones by means of, for instance, adding
execution options for improperly protected or unprotected Flatpak applications
or using tools such as Flatseal [^f2024fsea], it is still time-consuming and
the requirement here is to deliver the security by default.

Another weakness often argued by the communities on these all-in-on package
solutions is that the dependencies included may not be patched in time as soon
as the vulnerabilities have been discovered. Thus, they give a "false sense of
security".  Nevertheless, as Dunlap _et al._ points out [^t2022astu], this is
an issue with the maintainer but not the design of the packaging system.

The candidate solution to these issues is to standardize the packaging process
in the community. It is important to ensure every package has reasonable
policies defined in the process. Also, even if a CVE is found in only one of
these dependencies, that package must be rebuilt and published as fast as
possible, just as the ordinary GNU/Linux packaging systems do. Meanwhile, the
communities may introduce an automatic notification mechanism to inform the
package maintainers when vulnerabilities in dependencies have been reported,
according to their severity.

### MAC Tools

MAC (Mandatory Access Control) tools based on LSM, such as SELinux and
AppArmor, can also be used for the purpose of application confinement. However,
they mainly focus on restricting the file and object accessed by the
application processes, thus not the overall sandbox solutions. For example,
they do not provide a virtual environment or containerized environment.

In addition, it is difficult for normal users or package maintainers to develop
security policies or to customize the privileges exposed to the confined
applications, particularly obvious for the SELinux policy development in
contrast to the AppArmor, where many specific concepts such as SELinux users,
roles, and types, are involved. This issue is obvious in the GUI applications,
which are more complicated and probably involve more system resources. Most
GNU/Linux distributions with MAC tools enabled only have policies that cover
limited applications.

Despite that, MAC tools can work with other existing security features like
Seccomp and Capabilities to make an additional hardening layer. For instance,
Firejail has an option to run itself with AppArmor confinement enabled
[^f2024simp].

The policy development issue can be addressed by involving experts who are
familiar with both the MAC policies and the application privileges. It cost
time and efforts, but the investment could benefit all the downstream
distribution communities, just like what firejail community has done on
developing applicaiton profiles.

### Landlock

There are also other sandboxing solutions, mostly in the experiment phase, on
GNU/Linux platform which are not covered in Madaidan's article. Landlock
[^l2024land] is one of them.

Implemented in the kernel space, Landlock is designed as an unprivileged access
control mechanism in the LSM for the purpose of creating the sandbox during
application development. Since it is part of the LSM, it can be a stackable
layer upon other existing Linux security mechanisms in LSM [^l2024from] such as
SELinux, Apparmor, and other MAC tools. With the help of new system calls,
namely `landlock_create_ruleset()`, `landlock_add_rule()`, and
`landlock_restrict_self()`, Landlock creates embedded policy for an application
in its code, thereby reducing the tedious and error-prone work of policy
development on the downstream developers package and maintainer. Also, thanks
for the system call interfaces, Landlock works in an unprivileged way where the
risk of privilege escalation is further reduced.

The embedded policy moves the policy-making tasks from the sandbox solution
developers and users to the upstream application developers. The Landlock
project only provides the sandboxing mechanisms, namely Landlock system calls.
Also, users or package maintainers do not need to care about policy
configuration [^l2024from]. The absence of userspace configuration tools
simplifies the work for downstream distributions to a great extent. Only the
enabling of kernel options `CONFIG_SECURITY_LANDLOCK=y` and
`CONFIG_LSM=landlock,...` is required. So far, these kernel options have
already been the default options on many major Linux distributions, such as
Ubuntu, Fedora, openSUSE Tumbleweed, Gentoo, and Arch Linux [^l2023back] [^o2024sour].

However, the built-in policy increases the workload of the upstream application
developers. They have to define the policy and maintain this part of the code
continuously. Not all application developers are willing to do these additional
works. It is a blocker for promoting sandbox solutions in general. Up to now,
most Linux applications do not have dedicated landlock implementations.  The
developer of the Landlock project tried to push the patch of the tar command
with the sandbox implementation to its GNU Project but so far has not received
any responses [^m2021tars].

To address this issue, the third-party sandbox solutions could implement the
landlock in their code so the ruleset takes effect on the sandboxed
applications, just as Firejail does [^f2024land]. In this way, the ruleset
becomes configurable for users and profile developers.

### Virtualization and Containerization

Another common practice is employing Virtualization or Containerization
techniques as a sandbox solution in the Linux environment. Virtualization and
containerization provide a virtualized environment where the sandboxed
application runs. Since everything is in the environment, the interfaces like
system calls and file systems are self-contained in the images. Therefore, the
configuration of these interfaces is not required. Virtualization solutions
such as KVM and XEN include the entire Linux kernel in a completely isolated
runtime environment. In contrast, containerization solution like Docker, Linux
Containers (LXC), and Podman create their separate root structure and resources
with the help of the Linux namespace and organize the processes using Control
Group (cgroup) to create an isolated environment [^m2021towa].

Nevertheless, both virtualization and containerization have not been designed
as sandbox solutions from the very beginning. It is difficult for users to run
applications in those environments without tedious configurations. Users must
take care of file sharing between the host and guest system when files need to
be created or read on the host system. Moreover, the entire system installed in
the virtual environment must be updated regularly. It increases the overall
time cost of the sandbox maintenance. Performance is another non-ignorable
consideration. Even with the help of CPU Virtualization features, the
consumption of host's CPU Power, memory, and disk space still takes up a
significant portion of the system resource compared to other sandbox solutions.

More importantly, using virtualization and containerization as sandbox
solutions also has security weaknesses. Regarding virtualization, virtual
machine escape attack is always a risk. In terms of containerization, it
provides only limited protection to the application. Usually, the kernel space
processing is shared with the host system and other containers. More seriously,
container solutions like Docker require root privilege to start the daemon,
which further increases the attack interface. 

Some attempts have been made by the communities to address the above issues.
For example, QubeOS based on Xen virtualization supports running various Linux
applications in separate virtual machine environments [^q2024temp], which
reduces the complexity of creating the virtualization sandboxes. Moreover, as a
virtual machine monitor (VMM) that targets serverless computing, Firecracker
builds and manages KVM-based microVMs with a lightweight and minimalist design
to limit the attack surface [^f2024crac]. Also, the openSUSE MicroOS project
keeps a similar idea in its design but mainly targets container deployment in
the minimized VM environment [^o2022micr]. These projects can be potential
candidates for the VM-type sandboxing solution due to their ability to reduce
the consumption of the system resources. Finally, the container consumes much
fewer system resources than the virtual machine sandbox solution. Compared to
Docker, Podman is more suitable as a sandbox due to its rootless and daemonless
features [^r2024podm].


## Conclusion

So far, this article gives a general overview of the sandbox solutions on the
GNU/Linux platform and discusses the weaknesses and community efforts to
address these weaknesses. In a word, despite the various weaknesses of these
sandbox solutions, GNU/Linux provides more choices in contrast to the
proprietary operating systems. Moreover, these solutions can be combined or
stacked to achieve a higher level of protection.

No matter what operating systems they are, there is no such thing as a "true
sandbox" solution. Depending on the complexity of the application protected, it
always has some resources and interfaces exposed to the host system or other
applications to a certain extent, such as system calls, file systems, and
devices. Otherwise, the application does not function properly. These exposed
interfaces may introduce exploitable vulnerabilities and then weaken the entire
protection mechanism. Again, it is not only a "Linux thing". To evaluate the
protection level of sandbox solution on a specific operating system platform,
not only the general design needs to be examined, but also the implementation
in detail. Therefore, code-level auditing conducted by multiple parties may be
necessary, which is hard to implement on proprietary platforms.

<hr style="height:1px;border:none;color:#009BCD;background-color:#009BCD;" />

## References

[^b2016sand]: I. Borate and R. K. Chavan, "Sandboxing in linux: From smartphone
    to cloud." _International Journal of Computer Applications,_ vol.148, no.8,
    August 2016, doi:10.5120/ijca2016911256. [Online]. Available:
    <https://www.ijcaonline.org/archives/volume148/number8/25774-2016911256>

[^m2024linu]: Madaidan. "Security & Privacy Evaluations," Madaidan's
    Insecurities, March 18, 2022. Accessed: August 10, 2024. [Online].
    Available: <https://madaidans-insecurities.github.io/linux.html>

[^s2020rema]: S. Designer. "Re: major changes if gnu/linux dominates the
    desktop and/or mobile market?," Openwall, October t, 2020. Accessed: August
    10, 2024. [Online]. Available:
    <https://www.openwall.com/lists/oss-security/2020/10/05/5>

[^s2024mach]: Systemd. "machine-id(5) — Linux manual page," man7.org. Accessed:
    August 11, 2024. [Online]. Available:
    <https://man7.org/linux/man-pages/man5/machine-id.5.html>

[^t2024sysc]: The Linux Kernel Organization. "syscalls(2) — Linux manual page,"
    man7.org, May 2, 2024. Accessed: August 11, 2024. [Online]. Available:
    <https://man7.org/linux/man-pages/man2/syscalls.2.html>

[^t2024cgro]: The Linux Kernel Organization. "cgroups(7) — Linux manual page,"
    man7.org, June 15, 2024. Accessed: August 11, 2024. [Online]. Available:
    <https://man7.org/linux/man-pages/man7/cgroups.7.html>

[^t2024ipcn]: The Linux Kernel Organization. "ipc_namespaces(7) — Linux manual
    page," man7.org, May 2, 2024. Accessed: August 11, 2024. [Online].
    Available: <https://man7.org/linux/man-pages/man7/ipc_namespaces.7.html>

[^t2024pidn]: The Linux Kernel Organization. "pid_namespaces(7) — Linux manual
    page," man7.org, June 13, 2024. Accessed: August 11, 2024. [Online].
    Available: <https://man7.org/linux/man-pages/man7/pid_namespaces.7.html>

[^f2022dbus]: "dbus," freedesktop.org, February 28, 2022. Accessed:
    August 15, 2024. [Online]. Available:
    <https://www.freedesktop.org/wiki/Software/dbus/>

[^f2013dpro]: "DbusProjects," freedesktop.org, May 18, 2013. Accessed: August
    15, 2024. [Online]. Available:
    <https://www.freedesktop.org/wiki/Software/DbusProjects/>

[^f2023ddae]: "dbus-daemon — Message bus daemon," freedesktop.org,
    August 21, 2023. Accessed: August 15, 2024. [Online]. Available:
    <https://dbus.freedesktop.org/doc/dbus-daemon.1.html>

[^f2024ffaq]: Firejail. "Frequently Asked Questions," GitHub wiki.  Accessed:
    August 15, 2024.  [Online].  Available:
    <https://github.com/netblue30/firejail/wiki/Frequently-Asked-Questions#how-do-i-sandbox-applications-started-via-systemd-or-d-bus-services>

[^h2024wayl]: K. Høgsberg. "Wayland - The Wayland Protocol," freedesktop.org.
    Accessed: August 10, 2024. [Online]. Available:
    <https://wayland.freedesktop.org/docs/html/>

[^d2024wayl]: Debian. "Wayland," debian.org, May 1, 2024. Accessed:
    August 12, 2024. [Online]. Available: <https://wiki.debian.org/Wayland>

[^g2017wayl]: The GNOME Project. "Wayland," gnome.org, May 12, 2017. Accessed:
    August 14, 2024. [Online]. Available:
    <https://wiki.gnome.org/Initiatives/Wayland>

[^k2022wayl]: "KWin/Wayland," KDE Community Wiki, February 4, 2022. Accessed:
    August 14, 2024. [Online]. Available: <https://community.kde.org/KWin/Wayland>

[^x2024wayl]: "Xfce Wayland Development Roadmap", Xfce Developer Wiki, March
    18, 2024. Accessed: August 14, 2024. [Online]. Available:
    <https://wiki.xfce.org/releng/wayland_roadmap>

[^f2021puls]: "Access Control – Development Documentation – PulseAudio,"
    freedesktop.org, May 7, 2021. Accessed: August 14, 2024. [Online].
    Available:
    <https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/Developer/AccessControl/>

[^p2024pacc]: "PipeWire: Access Control," Pipewire Document. Accessed: August
    14, 2024. [Online]. Available: <https://docs.pipewire.org/page_access.html>

[^p2024pipw]: "Pipewire," Pipewire, 2022. Accessed: August 14, 2024. [Online].
    Available: <https://pipewire.org>

[^f2024infl]: Flatpak Team. "Introduction to Flatpak," Flatpak documentation,
    June 30, 2024. Accessed: August 16, 2024. [Online]. Available:
    <https://docs.flatpak.org/en/latest/introduction.html#terminology>

[^f2024undr]: Flatpak Team. "Under the Hood," Flatpak documentation, July
    24, 2024. Accessed: August 16, 2024. [Online]. Available:
    <https://docs.flatpak.org/en/latest/under-the-hood.html>

[^g2024minj]: Google. "./ minijail," Github page. Accessed: August 16, 2024.
    [Online]. Available: <https://google.github.io/minijail/>

[^m2024mgit]: "Minijail," Google Git Repositories on Chromium, March 22, 2024.
    Accessed: August 16, 2024. [Online]. Available:
    <https://chromium.googlesource.com/chromiumos/platform/minijail#historical-notes>

[^g2022hgit]: "Hacking on Minijail," Google Git repository on Chromium, August
    30, 2022.  Accessed: August 17, 2024. [Online]. Available:
    <https://chromium.googlesource.com/chromiumos/platform/minijail/+/refs/heads/main/HACKING.md>

[^b2024bgit]: "Bubblejail," Github repository, May 26, 2024. Accessed: August
    9, 2024. [Online]. Available: https://github.com/igo95862/bubblejail/>

[^u2024ufir]: "unprivileged firejail," Github issues, January 19, 2024.
    Accessed: August 9, 2024. [Online]. Available:
    <https://github.com/netblue30/firejail/issues/5157>

[^c2022intr]: Canonical Ubuntu. "Introduction to snaps," ubuntu.com,
    March 22, 2022. Accessed: August 10, 2024. [Online]. Available:
    <https://ubuntu.com/core/services/guide/snaps-intro>

[^c2023seca]: Canonical Ubuntu. "Security and sandboxing," ubuntu.com, February
    9, 2023. Accessed: August 10, 2024. [Online].
    Available: <https://ubuntu.com/core/docs/security-and-sandboxing>

[^f2024flat]: "Flatpak," Flatpak. Accessed: August 10, 2024. [Online].
    Available: <https://flatpak.org>

[^f2024fsea]: M. A. Lahaye. _Flatseal_. (2.2.0). Flathub. Accessed: August 18, 2024.
    [Online]. Available: <https://flathub.org/apps/com.github.tchx84.Flatseal>

[^f2024hood]: Flatpak Team. "Under the Hood - Flatpak documentation,"
    flatpak.org, July 24, 2024. Accessed: August 12, 2024. [Online]. Available:
    <https://docs.flatpak.org/en/latest/under-the-hood.html#underlying-technologies>

[^s2023part]: "Adding parts - Snapcraft documentation," Canonical Snapcraft,
    August 11, 2023. Accessed: August 12, 2024. [Online]. Available:
    <https://snapcraft.io/docs/adding-parts>

[^f2024depe]: Flatpak Team. "Dependencies - Flatpak documentation,"
    flatpak.org, July 22, 2024. Accessed: August 12, 2024. [Online]. Available:
    <https://docs.flatpak.org/en/latest/dependencies.html#bundling>

[^t2022astu]: T. Dunlap, W. Enck, and B. Reaves, "A Study of Application
    Sandbox Policies in Linux," in _SACMAT '22: Proceedings of the 27th ACM on
    Symposium on Access Control Models and Technologies,_ 2022, pp. 19-30, doi:
    10.1145/3532105.3535016. [Online]. Available:
    <https://dl.acm.org/doi/10.1145/3532105.3535016>

[^f2024simp]: "Firejail - Linux namespaces sandbox program," Github repository,
    July 31, 2024. Accessed: August 13, 2024. [Online]. Available:
    <https://github.com/netblue30/firejail/blob/master/src/man/firejail.1.in>

[^l2023back]: M. Salaün, Conference Presentation, Topic: "Backward and forward
    compatibility for security features (illustrated with Landlock)," FOSDEM,
    February 4, 2023.  Accessed: August 13, 2024. [Online]. Available:
    <https://landlock.io/talks/2023-02-04_rust-landlock-fosdem.pdf>

[^o2024sour]: "kernel-source/config/x86_64/default," Github repository.
    Accessed: August 16, 2024. [Online]. Available:
    <https://github.com/openSUSE/kernel-source/blob/master/config/x86_64/default>

[^l2024land]: "Landlock: unprivileged access control," landlock.io. Accessed:
    August 19,2024. [Online]. Available: <https://landlock.io/>

[^l2024from]: M. Salaün. "Landlock: From a security mechanism idea to a widely
    available implementation," landlock.io. Accessed: August 19, 2024.
    [Online].  Available:
    <https://landlock.io/talks/2024-06-06_landlock-article.pdf>

[^m2021tars]: M. Salaün. "[PATCH v1 0/1] Landlock Support," GNU Mailinglist,
    April 7, 2021.  Accessed: August 13, 2024. [Online]. Available:
    <https://lists.gnu.org/archive/html/bug-tar/2021-04/msg00001.html>

[^f2024land]: "README - Landlock support," Github repository, July 13, 2024.
    Accessed: August 17, 2024. [Online]. Available:
    <https://github.com/netblue30/firejail?tab=readme-ov-file#landlock-support>

[^m2021towa]: M. Reeves, D. J. Tian, A. Bianchi and Z. B. Celik, "Towards
    Improving Container Security by Preventing Runtime Escapes," 2021 IEEE
    Secure Development Conference (SecDev), Atlanta, GA, USA, 2021, pp. 38-46,
    doi: 10.1109/SecDev51306.2021.00022.

[^q2024temp]: "Templates - Qubes OS," Qubes OS. Accessed: August 16, 2024.
    [Online]. Available: <https://www.qubes-os.org/doc/templates/>

[^f2024crac]: "Firecracker - Secure and fast microVMs for serverless
    computing," Firecracker.  Accessed: August 23, 2024. [Online]. Available:
    <https://firecracker-microvm.github.io/>

[^o2022micr]: "Portal:MicroOS," openSUSE Wiki, June 15, 2022. Accessed: August
    23, 2024. [Online]. Available: <https://en.opensuse.org/Portal:MicroOS>

[^r2024podm]: "What is Podman?," Red Hat, June 20, 2024. Accessed:
    August 16, 2024. [Online]. Available:
    <https://www.redhat.com/en/topics/containers/what-is-podman>

