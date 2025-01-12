---
layout: post
title:  "Gigabit NAS (running CoreOS)"
date:   2016-11-21 16:12:00
categories: Artikel
Aliases:
  - /Artikel/gigabit-nas-coreos
---
<p>
tl;dr: I upgraded from a qnap TS-119P to a custom HTPC-like network storage
solution. This article outlines what my original reasoning was for the qnap
TS-119P, what I learnt, and with what solution precisely I replaced the qnap.
</p>

<p>
A little over two years ago, I gave a (German) presentation about my network
storage setup (see <a
href="https://media.ccc.de/v/gpn14_-_5801_-_de_-_studio_-_201406202130_-_linux-based_home_nas_-_secure">video</a>
or <a
href="https://entropia.de/images/6/6b/Gpn14_linux_based_home_nas.pdf">slides</a>).
Given that video isn’t a great consumption format when you’re pressed on time,
and given that a number of my readers might not speak German, I’ll recap the
most important points:
</p>
<ul>

<li>
<strong>I reduced the scope of the setup to storing daily backups and providing
media files via CIFS</strong>.<br>
I have come to prefer numerous smaller setups over one gigantic setup which
offers everything (think a <a href="https://omnia.turris.cz/en/">Turris
Omnia</a> acting as a router, mail server, network storage, etc.). Smaller
setups can be debugged, upgraded or rebuilt in less time. Time-boxing
activities has become very important to me as I work full time: if I can’t
imagine finishing the activity within 1 or 2 hours, I usually don’t get started
on it at all unless I’m on vacation.
</li>

<li>
<strong>Requirements: FOSS, relatively cheap, relatively low power usage,
relatively high redundancy level</strong>.<br>
I’m looking not to spend a large amount of money on hardware. Whenever I do
spend a lot, I feel pressured to get the most out of my purchase and use the
hardware for many years. However, I find it more satisfying to be able to
upgrade more frequently — just like the update this article is describing
:).<br>

With regards to redundancy, I’m not content with being able to rebuild the
system within a couple of days after a failure occurs. Instead, I want to be
able to trivially switch to a replacement system within minutes. This
requirement results in the decision to run 2 separate qnap NAS appliances with
1 hard disk each (instead of e.g. a RAID-1 setup).<br>

The decision to go with qnap as a vendor came from the fact that their devices
are pretty well supported in the Debian ecosystem: there is a network installer
for it, the custom hardware is supported by the <a
href="https://www.hellion.org.uk/qcontrol/">qcontrol</a> tool and one can build
a serial console connector.
</li>

<li>
The remainder of the points is largely related to software, and hence not
relevant for this article, as I’m keeping the software largely the same (aside
from the surrounding operating system).
</li>

</ul>

## What did not work well

<p>
Even a well-supported embedded device like the qnap TS-119P requires too much
effort to be set up:
</p>
<ol>
<li>
Setting up the network installer is cumbersome.
</li>
<li>
I contributed patches to qcontrol for setting the <a
href="https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=703888">wake on LAN
configuration on the qnap TS-119P’s controller board</a> and for <a
href="https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=703894">systemd
support in qcontrol</a>.
</li>
<li>
I contributed my first ever patch to the Linux kernel for <a
href="https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/net/ethernet/marvell/mv643xx_eth.c?id=3871c3876f8084a2f40ba3c3fc20a6bb5754d88d">wake
on LAN support for the Marvell MV643xx series chips</a>.
</li>
<li>
I ended up <a
href="http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=723177">lobbying Debian
to enable the <code>CONFIG_MARVELL_PHY</code> kernel option</a>, while
personally running a custom kernel.
</li>
</ol>

<p>
On the hardware side, to get any insight into what the device is doing, your
only input/output option is a serial console. To get easy access to that serial
console, you need to <a
href="https://www.cyrius.com/debian/kirkwood/qnap/ts-119/serial/">solder an
adapter cable for the somewhat non-standard header which they use</a>.
</p>

<p>
All of this contributed to my impression that upgrading the device would be
equally involved. Logically speaking, I know that this is unlikely since my patches
are upstream in the Linux kernel and in Debian. Nevertheless, I couldn’t help
but feel like it would be hard. As a result, I have not upgraded my device ever
since I got it working, i.e. more than two years ago.
</p>

<p>
The take-away is that I now try hard to:
</p>
<ol>
<li>
use <strong>standard hardware which fits well into my landscape</strong>
</li>
<li>
use a software setup which has as <strong>few non-standard
modifications</strong> as possible and which <strong>automatically updates
itself</strong>.
</li>
</ol>

## What I would like to improve

<p>
One continuous pain point was how slow the qnap TS-119P was with regards to
writing data to disk. The slowness was largely caused by full-disk encryption.
The device’s hardware accelerator turned out to be useless with
cryptsetup-luks’s comparatively small hard-coded 4K block size, resulting in
about 6 to 10 MB/s of throughput.
</p>

<p>
This resulted in me downloading files onto the SSD in my workstation and then
transferring them to the network storage. Doing these downloads in a faster
environment circumvents my somewhat irrational fears about the files becoming
unavailable while you are downloading them, and allows me to take pleasure in
how fast I’m able to download things :).
</p>

<p>
The take-away is that any new solution should be as quick as my workstation and
network, i.e. it should be able to <strong>write files to disk with gigabit
speed</strong>.
</p>

## What I can get rid of

<p>
While <a
href="https://github.com/stapelberg/zkj-nas-tools/tree/master/dramaqueen">dramaqueen</a>
and <a
href="https://github.com/stapelberg/zkj-nas-tools/tree/master/autowake">autowake</a>
worked well in principle, they turned out to no longer be very useful in my
environment: I switched from a dedicated OpenELEC-based media center box to
using <a href="https://emby.media/">emby</a> and a Chromecast. emby is also
nice to access remotely, e.g. when watching series at a friend’s place, or
watching movies while on vacation or business trips somewhere. Hence, my
storage solution needs to be running 24/7 — no more automated shutdowns and
wake-up procedures.
</p>

## What worked well

<p>
Reducing the scope drastically in terms of software setup complexity paid off.
If it weren’t for that, I probably would not have been able to complete this
upgrade within a few mornings/evenings and would likely have pushed this
project out for a long time.
</p>

## The new hardware

<p>
I researched the following components back in March, but then put the project
on hold due to time constraints and to allow myself some more time to think
about it. I finally ordered the components in August, and they still ranked
best with regards to cost / performance ratio and fulfilling my requirements.
</p>

<table width="100%" style="margin-top: 1.5em; margin-bottom: 1.5em; margin-left: 2em">
<tr>
<th>Price</th>
<th>Type</th>
<th>Article</th>
</tr>

<tr>
<td>43.49 CHF</td>
<td>Case</td>
<td><a href="http://www.silverstonetek.com/product.php?pid=413">SILVERSTONE Sugo SST-SG05BB-LITE</a></td>
</tr>

<tr>
<td>60.40 CHF</td>
<td>Mainboard</td>
<td><a href="http://www.asrock.com/mb/AMD/AM1H-ITX/">ASROCK AM1H-ITX</a></td>
</tr>

<tr>
<td>38.99 CHF</td>
<td>CPU</td>
<td>AMD Athlon 5350 (supports <a
href="https://en.wikipedia.org/wiki/AES_instruction_set">AES-NI</a>)</td>
</tr>

<tr>
<td>20.99 CHF</td>
<td>Cooler</td>
<td><a href="https://www.arctic.ac/ch_en/alpine-m1-passive.html">Alpine M1-Passive</a></td>
</tr>

<tr>
<td>32.80 CHF</td>
<td>RAM</td>
<td>KINGSTON ValueRAM, 8.0GB (KVR16N11/8)</td>
</tr>

<tr>
<td valign="top">28.70 CHF</td>
<td valign="top">PSU</td>
<td><a href="https://www.accuswiss.ch/product_info.php?info=p18088_Netzteil-fuer-Toshiba-PA3822U-1ACA--PA3822E-1AC3--19V-2-37A.html">Toshiba PA3822U-1ACA, PA3822E-1AC3, 19V 2,37A</a><br>
(<code>To19V_2.37A_5.5x2.5</code>)</td>
</tr>

<tr>
<td valign="top">0 CHF</td>
<td valign="top">System disk</td>
<td>(OCZ-AGILITY3 60G)<br>
You’ll need to do your own research.<br>
Currently, my system uses 5GB of space,<br>
so chose the smallest SSD you can find.</td>
</tr>

<tr>
<td>225.37 CHF</td>
<td colspan="2"><strong>total sum</strong></td>
</tr>
</table>

<p>
For the qnap TS-119P, I paid 226 CHF, so my new solution is a tad more
expensive. However, I had the OCZ-AGILITY3 lying around from a warranty
exchange, so bottomline, I paid less than what I had paid for the previous
solution.
</p>

<p>
I haven’t measured this myself, but according to the internet, the setups have
the following power consumption (without disks):
<ul>
<li>
The qnap TS-119P uses ≈7W, i.e. ≈60 CHF/year for electricity.
</li>
<li>
The AM1H-ITX / Athlon 5350 setup uses ≈20W, i.e. ≈77 CHF/year for electricity.
</li>
</ul>

<p>
In terms of fitting well into my hardware landscape, this system does a much
better job than the qnap. Instead of having to solder a custom serial port
adapter, I can simply connect a USB keyboard and an HDMI or DisplayPort monitor
and I’m done.
</p>

<p>
Further, any linux distribution can easily be installed from a bootable USB
drive, without the need for any custom tools or ports.
</p>

### Full-disk encryption performance

<pre>
# cryptsetup benchmark
# Tests are approximate using memory only (no storage IO).
PBKDF2-sha1       338687 iterations per second
PBKDF2-sha256     228348 iterations per second
PBKDF2-sha512     138847 iterations per second
PBKDF2-ripemd160  246375 iterations per second
PBKDF2-whirlpool   84891 iterations per second
#  Algorithm | Key |  Encryption |  Decryption
     aes-cbc   128b   468.8 MiB/s  1040.9 MiB/s
     aes-cbc   256b   366.4 MiB/s   885.8 MiB/s
     aes-xts   256b   850.8 MiB/s   843.9 MiB/s
     aes-xts   512b   725.0 MiB/s   740.6 MiB/s
</pre>

### Network performance

<p>
As the old qnap TS-119P would only sustain gigabit performance using IPv4 (with
TCP checksum offloading), I was naturally relieved to see that the new solution
can send packets at gigabit line rate using both protocols, IPv4 and IPv6. I
ran the following tests inside a docker container (<code>docker run --net=host
-t -i debian:latest</code>):
</p>

<pre>
# nc 10.0.0.76 3000 | dd of=/dev/null bs=5M
0+55391 records in
0+55391 records out
416109464 bytes (416 MB) copied, 3.55637 s, 117 MB/s
</pre>

<pre>
# nc 2001:db8::225:8aff:fe5d:53a9 3000 | dd of=/dev/null bs=5M
0+275127 records in
0+275127 records out
629802884 bytes (630 MB) copied, 5.45907 s, 115 MB/s
</pre>

<p>
The CPU was >90% idle using <code>netcat-traditional</code>.<br>
The CPU was >70% idle using <code>netcat-openbsd</code>.
</p>

### End-to-end throughput

<p>
Reading/writing to a disk which uses cryptsetup-luks full-disk encryption with
the <code>aes-cbc-essiv:sha256</code> cipher, these are the resulting speeds:
</p>

<p>
Reading a file from a CIFS mount works at gigabit throughput, without any
tuning:
</p>
<pre>
311+1 records in
311+1 records out
1632440260 bytes (1,6 GB) copied, 13,9396 s, 117 MB/s
</pre>

<p>
Writing works at almost gigabit throughput:
</p>
<pre>
1160+1 records in
1160+1 records out
6082701588 bytes (6,1 GB) copied, 58,0304 s, 105 MB/s
</pre>

<p>
During rsync+ssh backups, the CPU is never 100% maxed out, and data is sent to
the NAS at 65 MB/s.
</p>

## The new software setup

<p>
Given that I wanted to use a software setup which has as few non-standard
modifications as possible and automatically updates itself, I was curious to
see if I could carry this to the extreme by using <a
href="https://coreos.com/">CoreOS</a>.
</p>

<p>
If you’re unfamiliar with it, CoreOS is a Linux distribution which is intended
to be used in clusters on individual nodes. It updates itself automatically
(using <a href="https://github.com/google/omaha">omaha, Google’s updater behind
ChromeOS</a>) and comes as a largely read-only system without a package
manager. You deploy software using <a href="https://www.docker.com/">Docker</a>
and configure the setup using <a
href="https://coreos.com/os/docs/latest/cloud-config.html">cloud-config</a>.
</p>

<p>
I have been successfully using CoreOS for a few years in virtual machine setups
such as the one for <a href="https://robustirc.net/">the RobustIRC network</a>.
</p>

<p>
The cloud-config file I came up with can be found in Appendix A. You can pass
it to the <a
href="https://coreos.com/os/docs/latest/installing-to-disk.html">CoreOS
installer’s <code>-c</code> flag</a>. Personally, I installed CoreOS by booting
from a <a href="https://grml.org/">grml live linux USB key</a>, then <a
href="https://coreos.com/os/docs/latest/installing-to-disk.html">running the
CoreOS installer</a>.
</p>

<p>
In order to update the cloud-config file after installing CoreOS, you can use
the following commands:
</p>
<pre>
midna $ scp cloud-config.storage.yaml core@10.252:/tmp/
storage $ sudo cp /tmp/cloud-config.storage.yaml /var/lib/coreos-install/user_data
storage $ sudo coreos-cloudinit --from-file=/var/lib/coreos-install/user_data
</pre>

### Dockerfiles: rrsync and samba

<p>
Since neither rsync nor samba directly provide Docker containers, I had to whip
up the following small Dockerfiles which install the latest versions from
Debian jessie.
</p>

<p>
Of course, this means that I need to rebuild these two containers regularly,
but I also can easily roll them back in case an update broke, and the rest of
the operating system updates independently of these mission-critical pieces.
</p>

<p>
Eventually, I’m looking to enable auto-build for these Docker containers so
that the Docker hub rebuilds the images when necessary, and the updates are
picked up either manually when time-critical, or automatically by virtue of
CoreOS rebooting to update itself.
</p>

rrsync:

```dockerfile
FROM debian:jessie
RUN apt-get update \
  && apt-get install -y rsync \
  && gunzip -c /usr/share/doc/rsync/scripts/rrsync.gz > /usr/bin/rrsync \
  && chmod +x /usr/bin/rrsync
ENTRYPOINT ["/usr/bin/rrsync"]
```

samba:

```dockerfile
FROM debian:jessie
RUN apt-get update && apt-get install -y samba
ADD smb.conf /etc/samba/smb.conf
EXPOSE 137 138 139 445
CMD ["/usr/sbin/smbd", "-FS"]
```

### Appendix A: cloud-config

```yaml
#cloud-config

hostname: "storage"

ssh_authorized_keys:
  - ssh-rsa AAAAB… michael@midna

write_files:
  - path: /etc/ssl/certs/r.zekjur.net.crt
    content: |
      -----BEGIN CERTIFICATE-----
      MIIDYjCCAko…
      -----END CERTIFICATE-----
  - path: /var/lib/ip6tables/rules-save
    permissions: 0644
    owner: root:root
    content: |
      # Generated by ip6tables-save v1.4.14 on Fri Aug 26 19:57:51 2016
      *filter
      :INPUT DROP [0:0]
      :FORWARD ACCEPT [0:0]
      :OUTPUT ACCEPT [0:0]
      -A INPUT -p ipv6-icmp -m comment --comment "IPv6 needs ICMPv6 to work" -j ACCEPT
      -A INPUT -m state --state RELATED,ESTABLISHED -m comment --comment "Allow packets for outgoing connections" -j ACCEPT
      -A INPUT -s fe80::/10 -d fe80::/10 -m comment --comment "Allow link-local traffic" -j ACCEPT
      -A INPUT -s 2001:db8::/32 -m comment --comment "local traffic" -j ACCEPT
      -A INPUT -p tcp -m tcp --dport 22 -m comment --comment "SSH" -j ACCEPT
      COMMIT
      # Completed on Fri Aug 26 19:57:51 2016
  - path: /root/.ssh/authorized_keys
    permissions: 0600
    owner: root:root
    content: |
      command="/bin/docker run -i -e SSH_ORIGINAL_COMMAND -v /srv/backup/midna:/srv/backup/midna stapelberg/rsync /srv/backup/midna" ssh-rsa AAAAB… root@midna
      command="/bin/docker run -i -e SSH_ORIGINAL_COMMAND -v /srv/backup/scan2drive:/srv/backup/scan2drive stapelberg/rsync /srv/backup/scan2drive" ssh-rsa AAAAB… root@scan2drive
      command="/bin/docker run -i -e SSH_ORIGINAL_COMMAND -v /srv/backup/alp.zekjur.net:/srv/backup/alp.zekjur.net stapelberg/rsync /srv/backup/alp.zekjur.net" ssh-rsa AAAAB… root@alp

coreos:
  update:
    reboot-strategy: "reboot"
  locksmith:
    window_start: 01:00 # UTC, i.e. 02:00 CET or 03:00 CEST
    window_length: 2h  # i.e. until 03:00 CET or 04:00 CEST
  units:
    - name: ip6tables-restore.service
      enable: true

    - name: 00-enp2s0.network
      runtime: true
      content: |
        [Match]
        Name=enp2s0

        [Network]
        DNS=10.0.0.1
        Address=10.0.0.252/24
        Gateway=10.0.0.1
        IPv6Token=0:0:0:0:10::252

    - name: systemd-networkd-wait-online.service
      command: start
      drop-ins:
        - name: "10-interface.conf"
          content: |
            [Service]
            ExecStart=
            ExecStart=/usr/lib/systemd/systemd-networkd-wait-online \
	      --interface=enp2s0

    - name: unlock.service
      command: start
      content: |
        [Unit]
        Description=unlock hard drive
        Wants=network.target
        After=systemd-networkd-wait-online.service
        Before=samba.service
        
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        # Wait until the host is actually reachable.
        ExecStart=/bin/sh -c "c=0; while [ $c -lt 5 ]; do \
	    /bin/ping6 -n -c 1 r.zekjur.net && break; \
	    c=$((c+1)); \
	    sleep 1; \
	  done"
        ExecStart=/bin/sh -c "(echo -n my_local_secret && \
	  wget \
	    --retry-connrefused \
	    --ca-directory=/dev/null \
	    --ca-certificate=/etc/ssl/certs/r.zekjur.net.crt \
	    -qO- https://r.zekjur.net/sdb2_crypt) \
	  | /sbin/cryptsetup --key-file=- luksOpen /dev/sdb2 sdb2_crypt"
        ExecStart=/bin/mount /dev/mapper/sdb2_crypt /srv

    - name: samba.service
      command: start
      content: |
        [Unit]
        Description=samba server
        After=docker.service srv.mount
        Requires=docker.service srv.mount

        [Service]
        Restart=always
        StartLimitInterval=0

        # Always pull the latest version (bleeding edge).
        ExecStartPre=-/usr/bin/docker pull stapelberg/samba:latest

        # Set up samba users (cannot be done in the (public) Dockerfile
        # because users/passwords are sensitive information).
        ExecStartPre=-/usr/bin/docker kill smb
        ExecStartPre=-/usr/bin/docker rm smb
        ExecStartPre=-/usr/bin/docker rm smb-prep
        ExecStartPre=/usr/bin/docker run --name smb-prep stapelberg/samba \
	  adduser --quiet --disabled-password --gecos "" --uid 29901 michael
        ExecStartPre=/usr/bin/docker commit smb-prep smb-prepared
        ExecStartPre=/usr/bin/docker rm smb-prep
        ExecStartPre=/usr/bin/docker run --name smb-prep smb-prepared \
	  /bin/sh -c "echo my_password | tee - | smbpasswd -a -s michael"
        ExecStartPre=/usr/bin/docker commit smb-prep smb-prepared

        ExecStart=/usr/bin/docker run \
          -p 137:137 \
          -p 138:138 \
          -p 139:139 \
          -p 445:445 \
          --tmpfs=/run \
          -v /srv/data:/srv/data \
          --name smb \
          -t \
          smb-prepared \
            /usr/sbin/smbd -FS

    - name: emby.service
      command: start
      content: |
        [Unit]
        Description=emby
        After=docker.service srv.mount
        Requires=docker.service srv.mount

        [Service]
        Restart=always
        StartLimitInterval=0

        # Always pull the latest version (bleeding edge).
        ExecStartPre=-/usr/bin/docker pull emby/embyserver

        ExecStart=/usr/bin/docker run \
          --rm \
          --net=host \
          -v /srv/data/movies:/srv/data/movies:ro \
          -v /srv/data/series:/srv/data/series:ro \
          -v /srv/emby:/config \
          emby/embyserver

```
