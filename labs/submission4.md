# Lab 4 — Operating Systems & Networking (Submission)

> Branch: `feature/lab4`  
> File: `labs/submission4.md`

---

## Task 1 — Operating System Analysis (6 pts)

### 1.1 Boot Performance Analysis

#### 1.1.1 Analyze System Boot Time

**Command**
```sh
systemd-analyze
```

```bash
valeria@valeria-BBR-WAX9:~$ systemd-analyze
Startup finished in 5.306s (firmware) + 2.488s (loader) + 2.158s (kernel) + 8.283s (userspace) = 18.237s 
graphical.target reached after 8.270s in userspace
```

```sh
systemd-analyze blame
```

```bash
valeria@valeria-BBR-WAX9:~$ systemd-analyze blame
5.959s NetworkManager-wait-online.service
3.575s plymouth-quit-wait.service
1.852s snapd.seeded.service
1.758s snapd.service
 873ms gpu-manager.service
 772ms fwupd.service
 598ms docker.service
 542ms binfmt-support.service
 537ms systemd-binfmt.service
 463ms proc-sys-fs-binfmt_misc.mount
 456ms apparmor.service
 331ms dev-nvme0n1p2.device
 305ms dev-loop5.device
 305ms dev-loop1.device
 304ms dev-loop7.device
 303ms dev-loop4.device
 302ms dev-loop3.device
 302ms dev-loop0.device
 302ms dev-loop2.device
 301ms dev-loop6.device
 300ms dev-loop15.device
 295ms logrotate.service
 278ms snapd.apparmor.service
 267ms systemd-backlight@backlight:intel_backlight.service
 256ms dev-loop14.device
 249ms systemd-resolved.service
 237ms containerd.service
 235ms networkd-dispatcher.service
 222ms dev-loop13.device
 213ms systemd-timesyncd.service
 192ms systemd-oomd.service
 157ms udisks2.service
 156ms dev-loop12.device
 153ms ModemManager.service
 140ms bluetooth.service
 131ms systemd-logind.service
 131ms dev-loop11.device
 124ms accounts-daemon.service
 117ms user@1000.service
 112ms upower.service
  97ms systemd-update-utmp.service
  97ms geoclue.service
  97ms dev-loop8.device
  94ms dev-loop9.device
  88ms dev-loop10.device
  87ms systemd-udev-trigger.service
  83ms NetworkManager.service
  74ms plymouth-start.service
  66ms keyboard-setup.service
  64ms systemd-udevd.service
  62ms power-profiles-daemon.service
  61ms switcheroo-control.service
  60ms apport.service
  60ms avahi-daemon.service
  59ms systemd-journald.service
  58ms systemd-fsck@dev-disk-by\x2duuid-CA7D\x2dA5BC.service
  57ms polkit.service
  52ms e2scrub_reap.service
  51ms systemd-journal-flush.service
  46ms systemd-modules-load.service
  44ms grub-common.service
  42ms boot-efi.mount
  39ms packagekit.service
  31ms thermald.service
  31ms snap-code-220.mount
  30ms snap-bare-5.mount
  29ms snap-core20-2686.mount
  28ms gdm.service
  28ms wpa_supplicant.service
  27ms snap-core22-2216.mount
  26ms snap-core22-2292.mount
  24ms snap-firefox-7423.mount
  23ms snap-firefox-7672.mount
  23ms rsyslog.service
  22ms dev-hugepages.mount
  22ms snap-gnome\x2d42\x2d2204-202.mount
  21ms dev-mqueue.mount
  21ms systemd-tmpfiles-setup.service
  20ms snap-gnome\x2d42\x2d2204-226.mount
  19ms sys-kernel-debug.mount
  18ms snap-gtk\x2dcommon\x2dthemes-1535.mount
  18ms sys-kernel-tracing.mount
  17ms snap-snap\x2dstore-1113.mount
  17ms colord.service
  17ms systemd-remount-fs.service
  16ms snap-snap\x2dstore-1216.mount
  15ms snap-snapd-25577.mount
  14ms systemd-sysusers.service
  14ms cups.service
  14ms systemd-random-seed.service
  13ms snap-snapd-25935.mount
  13ms kmod-static-nodes.service
  13ms modprobe@configfs.service
  12ms sys-fs-fuse-connections.mount
  12ms modprobe@drm.service
  12ms snap-snapd\x2ddesktop\x2dintegration-315.mount
  12ms kerneloops.service
  12ms plymouth-read-write.service
  11ms docker.socket
  10ms snap-snapd\x2ddesktop\x2dintegration-343.mount
  10ms modprobe@fuse.service
  10ms alsa-restore.service
  10ms systemd-sysctl.service
  10ms console-setup.service
   9ms grub-initrd-fallback.service
   9ms swapfile.swap
   8ms systemd-user-sessions.service
   8ms systemd-tmpfiles-setup-dev.service
   7ms user-runtime-dir@1000.service
   5ms e2scrub_all.service
   4ms openvpn.service
   4ms systemd-rfkill.service
   4ms ufw.service
   4ms systemd-update-utmp-runlevel.service
   3ms sys-kernel-config.mount
   3ms modprobe@efi_pstore.service
   3ms rtkit-daemon.service
   2ms setvtrgb.service
 673us snapd.socket
lines 97-119/119 (END)
```

#### Observations
1) Total boot time (firmware + loader + kernel + userspace): 18.237s. Graphical target reached after 8.270s in userspace.
2) Longest startup units (possible bottlenecks):
    - NetworkManager-wait-online.service (5.959s) - waiting for “the network is ready".
    - plymouth-quit-wait.service (3.575s) - waiting for the completion of splash/plymouth.
3) Additionally, docker.service ~598ms, containerd.service ~237ms — the container stack is raised at startup


#### 1.1.2 Check System Load

```sh
uptime
```

```bash
23:38:58 up 5 min,  1 user,  load average: 1.27, 0.80, 0.36
```

```sh
w
```

```bash
23:38:58 up 5 min,  1 user,  load average: 1.27, 0.80, 0.36
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
valeria  tty2     tty2             23:34    5:06   0.02s  0.00s /usr/libexec/gd
```

#### Observations
1) The system works for ~5 minutes after booting, and 1 user is active
2) Load average: 1.27 / 0.80 / 0.36 (1/5/15 min). This looks normal for the early stages of the launch.

### 1.2 Process Forensics

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6
```

```bash
    PID    PPID CMD                         %MEM %CPU
   3068    1705 ~/Downloads/<redacted>       3.6  4.8
   2206    1705 /snap/snap-store/1216/usr/b  1.7  1.5
   1882    1705 /usr/bin/gnome-shell         1.6  3.3
   3138    1705 /usr/libexec/gsd-xsettings   0.5  0.0
   3118    1882 /usr/bin/Xwayland :0 -rootl  0.4  0.0
```

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6
```

```bash
    PID    PPID CMD                         %MEM %CPU
   3068    1705 ~/Downloads/<redacted>       3.6  4.6
   1882    1705 /usr/bin/gnome-shell         1.6  3.3
   1140       1 /usr/libexec/packagekitd     0.3  2.1
   2206    1705 /snap/snap-store/1216/usr/b  1.7  1.3
    608       1 /usr/lib/snapd/snapd         0.2  0.9
```

#### Answer: What is the top memory-consuming process?
Top by %MEM: PID 3068 (/home/valeria/Downloads/<redacted>) — 3.6% MEM.

#### Observations
1) At the time of the check, the most “heavy” process in terms of both RAM and CPU is .../Downloads/<redacted> (~3.6% MEM, ~4.6–4.8% CPU).
2) From the system: gnome-shell is noticeable by CPU, snap-store by memory, and packagekitd is active (updates/packages).

### 1.3 Service Dependencies
```sh
systemctl list-dependencies
```

```bash
 valeria@valeria-BBR-WAX9:~$ systemctl list-dependencies
default.target
● ├─accounts-daemon.service
● ├─apport.service
● ├─gdm.service
● ├─power-profiles-daemon.service
● ├─switcheroo-control.service
○ ├─systemd-update-utmp-runlevel.service
● ├─udisks2.service
● └─multi-user.target
●   ├─anacron.service
●   ├─apport.service
●   ├─avahi-daemon.service
●   ├─binfmt-support.service
●   ├─console-setup.service
●   ├─containerd.service
●   ├─cron.service
●   ├─cups-browsed.service
●   ├─cups.path
●   ├─cups.service
●   ├─dbus.service
○   ├─dmesg.service
●   ├─docker.service
○   ├─e2scrub_reap.service
○   ├─grub-common.service
○   ├─grub-initrd-fallback.service
●   ├─irqbalance.service
●   ├─kerneloops.service
●   ├─ModemManager.service
●   ├─networkd-dispatcher.service
●   ├─NetworkManager.service
●   ├─openvpn.service
●   ├─plymouth-quit-wait.service
○   ├─plymouth-quit.service
●   ├─rsyslog.service
○   ├─secureboot-db.service
●   ├─snap-bare-5.mount
●   ├─snap-code-220.mount
●   ├─snap-core20-2686.mount
●   ├─snap-core22-2216.mount
●   ├─snap-core22-2292.mount
●   ├─snap-firefox-7423.mount
●   ├─snap-firefox-7672.mount
●   ├─snap-gnome\x2d42\x2d2204-202.mount
●   ├─snap-gnome\x2d42\x2d2204-226.mount
●   ├─snap-gtk\x2dcommon\x2dthemes-1535.mount
●   ├─snap-snap\x2dstore-1113.mount
●   ├─snap-snap\x2dstore-1216.mount
●   ├─snap-snapd-25577.mount
●   ├─snap-snapd-25935.mount
●   ├─snap-snapd\x2ddesktop\x2dintegration-315.mount
●   ├─snap-snapd\x2ddesktop\x2dintegration-343.mount
●   ├─snapd.apparmor.service
○   ├─snapd.autoimport.service
○   ├─snapd.core-fixup.service
○   ├─snapd.recovery-chooser-trigger.service
●   ├─snapd.seeded.service
●   ├─snapd.service
●   ├─systemd-ask-password-wall.path
●   ├─systemd-logind.service
●   ├─systemd-oomd.service
●   ├─systemd-resolved.service
○   ├─systemd-update-utmp-runlevel.service
●   ├─systemd-user-sessions.service
●   ├─thermald.service
○   ├─ua-reboot-cmds.service
○   ├─ubuntu-advantage.service
●   ├─ufw.service
●   ├─unattended-upgrades.service
●   ├─whoopsie.path
●   ├─wpa_supplicant.service
●   ├─basic.target
●   │ ├─-.mount
○   │ ├─tmp.mount
●   │ ├─paths.target
●   │ │ ├─acpid.path
○   │ │ └─apport-autoreport.path
●   │ ├─slices.target
●   │ │ ├─-.slice
●   │ │ └─system.slice
●   │ ├─sockets.target
●   │ │ ├─acpid.socket
○   │ │ ├─apport-forward.socket
●   │ │ ├─avahi-daemon.socket
●   │ │ ├─cups.socket
●   │ │ ├─dbus.socket
●   │ │ ├─docker.socket
●   │ │ ├─snapd.socket
●   │ │ ├─systemd-initctl.socket
●   │ │ ├─systemd-journald-audit.socket
●   │ │ ├─systemd-journald-dev-log.socket
●   │ │ ├─systemd-journald.socket
●   │ │ ├─systemd-udevd-control.socket
●   │ │ ├─systemd-udevd-kernel.socket
●   │ │ └─uuidd.socket
●   │ ├─sysinit.target
●   │ │ ├─apparmor.service
●   │ │ ├─dev-hugepages.mount
●   │ │ ├─dev-mqueue.mount
●   │ │ ├─keyboard-setup.service
●   │ │ ├─kmod-static-nodes.service
●   │ │ ├─plymouth-read-write.service
●   │ │ ├─plymouth-start.service
●   │ │ ├─proc-sys-fs-binfmt_misc.automount
●   │ │ ├─setvtrgb.service
●   │ │ ├─sys-fs-fuse-connections.mount
●   │ │ ├─sys-kernel-config.mount
●   │ │ ├─sys-kernel-debug.mount
●   │ │ ├─sys-kernel-tracing.mount
○   │ │ ├─systemd-ask-password-console.path
●   │ │ ├─systemd-binfmt.service
○   │ │ ├─systemd-boot-system-token.service
●   │ │ ├─systemd-journal-flush.service
●   │ │ ├─systemd-journald.service
○   │ │ ├─systemd-machine-id-commit.service
●   │ │ ├─systemd-modules-load.service
○   │ │ ├─systemd-pstore.service
●   │ │ ├─systemd-random-seed.service
●   │ │ ├─systemd-sysctl.service
●   │ │ ├─systemd-sysusers.service
●   │ │ ├─systemd-timesyncd.service
●   │ │ ├─systemd-tmpfiles-setup-dev.service
●   │ │ ├─systemd-tmpfiles-setup.service
●   │ │ ├─systemd-udev-trigger.service
●   │ │ ├─systemd-udevd.service
●   │ │ ├─systemd-update-utmp.service
●   │ │ ├─cryptsetup.target
●   │ │ ├─local-fs.target
●   │ │ │ ├─-.mount
●   │ │ │ ├─boot-efi.mount
○   │ │ │ ├─systemd-fsck-root.service
●   │ │ │ └─systemd-remount-fs.service
●   │ │ ├─swap.target
●   │ │ │ └─swapfile.swap
●   │ │ └─veritysetup.target
●   │ └─timers.target
●   │   ├─anacron.timer
○   │   ├─apport-autoreport.timer
●   │   ├─apt-daily-upgrade.timer
●   │   ├─apt-daily.timer
●   │   ├─dpkg-db-backup.timer
●   │   ├─e2scrub_all.timer
●   │   ├─fstrim.timer
●   │   ├─fwupd-refresh.timer
●   │   ├─logrotate.timer
●   │   ├─man-db.timer
●   │   ├─motd-news.timer
○   │   ├─snapd.snap-repair.timer
●   │   ├─systemd-tmpfiles-clean.timer
○   │   ├─ua-timer.timer
●   │   ├─update-notifier-download.timer
●   │   └─update-notifier-motd.timer
●   ├─getty.target
○   │ ├─getty-static.service
○   │ └─getty@tty1.service
●   └─remote-fs.target
```

```sh
systemctl list-dependencies multi-user.target
```

```bash
 valeria@valeria-BBR-WAX9:~$ systemctl list-dependencies multi-user.target
multi-user.target
● ├─anacron.service
● ├─apport.service
● ├─avahi-daemon.service
● ├─binfmt-support.service
● ├─console-setup.service
● ├─containerd.service
● ├─cron.service
● ├─cups-browsed.service
● ├─cups.path
● ├─cups.service
● ├─dbus.service
○ ├─dmesg.service
● ├─docker.service
○ ├─e2scrub_reap.service
○ ├─grub-common.service
○ ├─grub-initrd-fallback.service
● ├─irqbalance.service
● ├─kerneloops.service
● ├─ModemManager.service
● ├─networkd-dispatcher.service
● ├─NetworkManager.service
● ├─openvpn.service
● ├─plymouth-quit-wait.service
○ ├─plymouth-quit.service
● ├─rsyslog.service
○ ├─secureboot-db.service
● ├─snap-bare-5.mount
● ├─snap-code-220.mount
● ├─snap-core20-2686.mount
● ├─snap-core22-2216.mount
● ├─snap-core22-2292.mount
● ├─snap-firefox-7423.mount
● ├─snap-firefox-7672.mount
● ├─snap-gnome\x2d42\x2d2204-202.mount
● ├─snap-gnome\x2d42\x2d2204-226.mount
● ├─snap-gtk\x2dcommon\x2dthemes-1535.mount
● ├─snap-snap\x2dstore-1113.mount
● ├─snap-snap\x2dstore-1216.mount
● ├─snap-snapd-25577.mount
● ├─snap-snapd-25935.mount
● ├─snap-snapd\x2ddesktop\x2dintegration-315.mount
● ├─snap-snapd\x2ddesktop\x2dintegration-343.mount
● ├─snapd.apparmor.service
○ ├─snapd.autoimport.service
○ ├─snapd.core-fixup.service
○ ├─snapd.recovery-chooser-trigger.service
● ├─snapd.seeded.service
● ├─snapd.service
● ├─systemd-ask-password-wall.path
● ├─systemd-logind.service
● ├─systemd-oomd.service
● ├─systemd-resolved.service
○ ├─systemd-update-utmp-runlevel.service
● ├─systemd-user-sessions.service
● ├─thermald.service
○ ├─ua-reboot-cmds.service
○ ├─ubuntu-advantage.service
● ├─ufw.service
● ├─unattended-upgrades.service
● ├─whoopsie.path
● ├─wpa_supplicant.service
● ├─basic.target
● │ ├─-.mount
○ │ ├─tmp.mount
● │ ├─paths.target
● │ │ ├─acpid.path
○ │ │ └─apport-autoreport.path
● │ ├─slices.target
● │ │ ├─-.slice
● │ │ └─system.slice
● │ ├─sockets.target
● │ │ ├─acpid.socket
○ │ │ ├─apport-forward.socket
● │ │ ├─avahi-daemon.socket
● │ │ ├─cups.socket
● │ │ ├─dbus.socket
● │ │ ├─docker.socket
● │ │ ├─snapd.socket
● │ │ ├─systemd-initctl.socket
● │ │ ├─systemd-journald-audit.socket
● │ │ ├─systemd-journald-dev-log.socket
● │ │ ├─systemd-journald.socket
● │ │ ├─systemd-udevd-control.socket
● │ │ ├─systemd-udevd-kernel.socket
● │ │ └─uuidd.socket
● │ ├─sysinit.target
● │ │ ├─apparmor.service
● │ │ ├─dev-hugepages.mount
● │ │ ├─dev-mqueue.mount
● │ │ ├─keyboard-setup.service
● │ │ ├─kmod-static-nodes.service
● │ │ ├─plymouth-read-write.service
● │ │ ├─plymouth-start.service
● │ │ ├─proc-sys-fs-binfmt_misc.automount
● │ │ ├─setvtrgb.service
● │ │ ├─sys-fs-fuse-connections.mount
● │ │ ├─sys-kernel-config.mount
● │ │ ├─sys-kernel-debug.mount
● │ │ ├─sys-kernel-tracing.mount
○ │ │ ├─systemd-ask-password-console.path
● │ │ ├─systemd-binfmt.service
○ │ │ ├─systemd-boot-system-token.service
● │ │ ├─systemd-journal-flush.service
● │ │ ├─systemd-journald.service
○ │ │ ├─systemd-machine-id-commit.service
● │ │ ├─systemd-modules-load.service
○ │ │ ├─systemd-pstore.service
● │ │ ├─systemd-random-seed.service
● │ │ ├─systemd-sysctl.service
● │ │ ├─systemd-sysusers.service
● │ │ ├─systemd-timesyncd.service
● │ │ ├─systemd-tmpfiles-setup-dev.service
● │ │ ├─systemd-tmpfiles-setup.service
● │ │ ├─systemd-udev-trigger.service
● │ │ ├─systemd-udevd.service
● │ │ ├─systemd-update-utmp.service
● │ │ ├─cryptsetup.target
● │ │ ├─local-fs.target
● │ │ │ ├─-.mount
● │ │ │ ├─boot-efi.mount
○ │ │ │ ├─systemd-fsck-root.service
● │ │ │ └─systemd-remount-fs.service
● │ │ ├─swap.target
● │ │ │ └─swapfile.swap
● │ │ └─veritysetup.target
● │ └─timers.target
● │   ├─anacron.timer
○ │   ├─apport-autoreport.timer
● │   ├─apt-daily-upgrade.timer
● │   ├─apt-daily.timer
● │   ├─dpkg-db-backup.timer
● │   ├─e2scrub_all.timer
● │   ├─fstrim.timer
● │   ├─fwupd-refresh.timer
● │   ├─logrotate.timer
● │   ├─man-db.timer
● │   ├─motd-news.timer
○ │   ├─snapd.snap-repair.timer
● │   ├─systemd-tmpfiles-clean.timer
○ │   ├─ua-timer.timer
● │   ├─update-notifier-download.timer
● │   └─update-notifier-motd.timer
● ├─getty.target
○ │ ├─getty-static.service
○ │ └─getty@tty1.service
● └─remote-fs.target

```

#### Observations
1) default.target includes the graphical part (gdm.service) and leads to multi-user.target
2) multi-user.target pulls a typical “basic set” of services: network (NetworkManager), logs (rsyslog), jobs (cron), containers (docker/containerd), snap (snapd and mount’s), security (ufw, apparmor)

### 1.4 User Sessions

```sh
who -a
```

```bash
           system boot  2026-02-22 23:33
           run-level 5  2026-02-22 23:34
valeria  + tty2         2026-02-22 23:34 00:10        1794 (tty2)
```

```sh
last -n 5
```

```bash
valeria  tty2         tty2             Sun Feb 22 23:34   still logged in
reboot   system boot  6.8.0-94-generic Sun Feb 22 23:33   still running
valeria  tty2         tty2             Sat Feb 14 15:37 - down  (1+07:28)
reboot   system boot  6.8.0-83-generic Sat Feb 14 15:36 - 23:06 (1+07:29)
valeria  tty2         tty2             Tue Dec  9 22:15 - down  (52+16:29)

wtmp begins Wed Nov 22 00:22:00 2023
```

#### Observations
1) Current system boot: 2026-02-22 23:33, runlevel 5 (graphical).
2) User valeria is logged in on tty2 and remains in the system.
3) The history shows several previous boot/login entries (you can see the change in kernel version: 6.8.0-83-generic → 6.8.0-94-generic).

### 1.5 Memory Analysis

```sh
free -h
```

```bash
               total        used        free      shared  buff/cache   available
Mem:            15Gi       1.5Gi        11Gi       223Mi       2.3Gi        13Gi
Swap:          2.0Gi          0B       2.0Gi
```

```sh
cat /proc/meminfo | grep -e MemTotal -e SwapTotal -e MemAvailable
```

```bash
MemTotal:       16111952 kB
MemAvailable:   13981616 kB
SwapTotal:       2097148 kB
```

#### Observations
1) RAM: 15Gi total, ~13Gi available — the system is almost not loaded by memory.
2) Swap: 2.0Gi total, 0B used — swap is not used (normal with large available RAM)

## Task 2 — Networking Analysis (4 pts)

### 2.1 Network Path Tracing

#### 2.1.1 Traceroute Execution

**Command**
```sh
traceroute github.com
```

``` bash
traceroute to github.com (140.82.121.3), 30 hops max, 60 byte packets
 1  _gateway (192.168.0.XXX)  4.625 ms  4.760 ms  4.895 ms
 2  10.76.0.XXX (10.76.0.XXX)  32.522 ms  32.488 ms  32.453 ms
 3  192.168.126.XXX (192.168.126.XXX)  32.422 ms  32.389 ms  32.354 ms
 4  77.37.250.223 (77.37.250.223)  32.340 ms  32.307 ms  32.277 ms
 5  87.226.221.70 (87.226.221.70)  15.182 ms  15.319 ms  15.512 ms
 6  185.140.148.29 (185.140.148.29)  37.398 ms *  33.138 ms
 7  netnod-ix-ge-b-sth-1500.inter.link (194.68.128.180)  129.505 ms  129.436 ms  129.339 ms
 8  * * *
 9  r3-ber1-de.as5405.net (94.103.180.2)  4373.697 ms  4405.831 ms  4429.501 ms
10  94.103.180.7 (94.103.180.7)  3857.440 ms  3857.402 ms  3857.363 ms
11  * * *
12  r3-fra3-de.as5405.net (94.103.180.54)  503.050 ms  2242.093 ms  2241.974 ms
13  r1-fra3-de.as5405.net (94.103.180.24)  56.847 ms  50.816 ms  118.001 ms
14  * cust-sid435.r1-fra3-de.as5405.net (45.153.82.39)  116.329 ms cust-sid436.fra3-de.as5405.net (45.153.82.37)  1949.980 ms
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

#### Insights
1) The target resolves to IP 140.82.121.3 (GitHub).
2) The first hops are the local network/provider (192.168.x.x, 10.x.x.x), followed by public routers.
3) Starting with hop 8, * * * appears (ICMP/UDP responses can be filtered or rate-limited).
4) The trace did not reach the final host (many * after the 14th hop), probably due to traceroute traffic filtering.

#### 2.1.2 DNS Resolution Check

```sh
dig github.com
```

```bash
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14418
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;github.com.      IN  A

;; ANSWER SECTION:
github.com.    104  IN  A  140.82.121.3

;; Query time: 88 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Mon Feb 23 00:21:20 MSK 2026
;; MSG SIZE  rcvd: 55
```

#### Analysis
1) Answer: A-record → 140.82.121.3, TTL 104s
2) DNS resolver: 127.0.0.53 (local stub-resolver systemd-resolved)
3) Request time: 88 ms

### 2.2 Packet Capture (DNS)

```sh
sudo timeout 10 tcpdump -c 5 -i any 'port 53' -nn
```

```bash
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
00:22:26.235121 wlp0s20f3 Out IP 192.168.0.XXX.60784 > 77.37.251.33.53: 18966+ [1au] AAAA? connectivity-check.ubuntu.com. (58)
00:22:26.254402 wlp0s20f3 In  IP 77.37.251.33.53 > 192.168.0.XXX.60784: 18966 12/0/1 AAAA 2001:67c:1562::23, AAAA 2001:67c:1562::24, AAAA 2620:2d:4000:1::97, AAAA 2620:2d:4002:1::197, AAAA 2620:2d:4000:1::2b, AAAA 2620:2d:4000:1::96, AAAA 2620:2d:4000:1::23, AAAA 2620:2d:4002:1::198, AAAA 2620:2d:4002:1::196, AAAA 2620:2d:4000:1::2a, AAAA 2620:2d:4000:1::22, AAAA 2620:2d:4000:1::98 (394)

2 packets captured
2 packets received by filter
0 packets dropped by kernel
```

#### One example DNS query from capture

`AAAA? connectivity-check.ubuntu.com. from 192.168.0.XXX:60784 to DNS server 77.37.251.33:53`

#### Observations
1) A typical DNS query/response has been captured
2) The request was for AAAA (IPv6 addresses) for connectivity-check.ubuntu.com
3) The response returned a set of IPv6 addresses (multiple AAAA-records)

### 2.3 Reverse DNS (PTR lookups)

```sh
dig -x 8.8.4.4
```

```bash
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> -x 8.8.4.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17715
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;4.4.8.8.in-addr.arpa.    IN  PTR

;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.  18251  IN  PTR  dns.google.

;; Query time: 10 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Mon Feb 23 00:23:17 MSK 2026
;; MSG SIZE  rcvd: 73
```

#### Analysis
1) Answer: PTR-record → dns.google. TTL 18251s
2) DNS resolver: 127.0.0.53 (local stub-resolver systemd-resolved)
3) Request time: 10 ms


```sh
dig -x 1.1.2.2
```

```bash
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> -x 1.1.2.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 9505
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;2.2.1.1.in-addr.arpa.    IN  PTR

;; AUTHORITY SECTION:
1.in-addr.arpa.    1046  IN  SOA  ns.apnic.net. read-txt-record-of-zone-first-dns-admin.apnic.net. 23597 7200 1800 604800 3600

;; Query time: 468 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Mon Feb 23 00:24:07 MSK 2026
;; MSG SIZE  rcvd: 137
```

#### Comparison
1) 8.8.4.4 successfully resolves to dns.google. (PTR is present).
2) 1.1.2.2 returns NXDOMAIN — there is no PTR record for this address (or it is not delegated), so the reverse name is not found
3) The response time for 1.1.2.2 is significantly higher (468 ms) compared to 8.8.4.4 (10 ms), which may be due to the specific routing/cache/authoritative servers for the zone.

### Security Notes (sanitization)
1) Private/local IP addresses were sanitized by replacing the last octet with XXX (e.g., 192.168.0.XXX, 10.76.0.XXX)
2) In tcpdump, the local IP was sanitized as required (last octet replaced with XXX)
3) A potentially sensitive local process path/name was redacted in the process list (~/Downloads/<redacted>)