# Next Step
Here we will install kodi-standalone-service, droidmote, tvheadend from aur

login as admin
```
sudo pacman -S avahi ntfs-3g udisks2 dosfstools gptfdisk git afpfs-ng bluez python2-pybluez libplist pulseaudio shairplay upower nss-mdns xorg-fonts

git https://aur.archlinux.org/kodi-standalone-service.git
cd kodi-standalone-service
makepkg -si

sudo systemctl enable kodi.service
```
Reboot and see how you go

Then we can droidmote if needed.
```
git clone https://aur.archlinux.org/droidmote.git
cd droidmote
makepkg -si
```
sudo systemctl enable droidmote.service
droidmote needs X11 so we need to alter the service slightly and maybe up its priority for input
Also we will create a quiet and fastboot env for Arch & Kodi

```
sudo nano /usr/lib/systemd/system/droidmote.service

[Unit]
Description=DroidMote
After=kodi.service

[Service]
EnvironmentFile=/etc/droidmote.conf
ExecStart=/usr/bin/droidmote $PORT $PASSWORD 
CPUSchedulingPolicy=SCHED_RR
CPUSchedulingPriority=99
Restart=always

[Install]
WantedBy=graphical.target

sudo nano /usr/lib/systemd/system/kodi.service

[Unit]
Description=Kodi standalone (X11)
After=remote-fs.target systemd-user-sessions.service network-online.target nss-lookup.target sound.target bluetooth.target polkit.service upower.service mysqld.service
Wants=network-online.target polkit.service upower.service
Conflicts=getty@tty1.service

[Service]
User=kodi
Group=kodi
PAMName=login
TTYPath=/dev/tty1
Environment=WINDOWING=x11
ExecStart=/usr/local/bin/kodi-standalone
Restart=on-abort
StandardInput=tty

[Install]
WantedBy=graphical.target

#

sudo nano /usr/local/bin/kodi-standalone


#!/bin/sh
/usr/bin/xinit /usr/bin/kodi-standalone -- :0 -nolisten tcp vt1 &> /dev/null


sudo chmod a+x /usr/local/bin/kodi-standalone
sudo chown kodi:kodi /usr/local/bin/kodi-standalone
#
sudo nano /boot/loader/entries/arch.conf

title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=8e9084b3-32ce-485b-8ee0-6178c8afe4f8 rw quiet loglevel=3 rd.systemd.show_status=auto rd.udev.log_priority=3 i915.fastboot=1

#
sudo nano /etc/sysctl.d/20-quiet-printk.conf
kernel.printk = 3 3 3 3

#
sudo mkdir /etc/systemd/system/getty@tty1.service.d
sudo nano /etc/systemd/system/getty@tty1.service.d/skip-prompt.conf

[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --skip-login --nonewline --noissue --autologin username --noclear %I $TERM

#

fsck
To hide fsck messages during boot, let systemd check the root filesystem. For this, replace udev hook with systemd:

HOOKS=( base systemd fsck ...) 
in /etc/mkinitcpio.conf and then run:

mkinitcpio -p linux
Now edit systemd-fsck-root.service and systemd-fsck@.service:

# systemctl edit --full systemd-fsck-root.service
# systemctl edit --full systemd-fsck@.service
Configuring StandardOutput and StandardError like this:

(...)

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-fsck
StandardOutput=null
StandardError=journal+console
TimeoutSec=0

#
```
Then we will enable the avahi service
```
sudo systemctl disable systemd-resolved.service
sudo systemctl enable avahi-daemon.service

sudo nano /etc/nsswitch.conf

# Name Service Switch configuration file.
# See nsswitch.conf(5) for details.

passwd: files mymachines systemd
group: files mymachines systemd
shadow: files

publickey: files

hosts: files mymachines myhostname resolve [!UNAVAIL=return] mdns_minimal [NOTFOUND=return] dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

netgroup: files

sudo nano /etc/mkinitcpio.conf

Edit HOOKS Line
##   NOTE: If you have /usr on a separate partition, you MUST include the
#    usr, fsck and shutdown hooks.
HOOKS=(base systemd autodetect modconf block filesystems keyboard fsck)

sudo mkinitcpio -p linux

sudo systemctl edit --full systemd-fsck-root.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-fsck
StandardOutput=null
StandardError=journal+console
TimeoutSec=0


sudo systemctl edit --full systemd-fsck@.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-fsck %f
StandardOutput=null
StandardError=journal+console
TimeoutSec=0

sudo usermod -a -G audio,optical,uucp,video,kodi admin
sudo usermod -a -G admin kodi
sudo chmod -R g+rwx /var/lib/kodi
sudo chmod -R g+rwx /home/admin

git clone https://aur.archlinux.org/tvheadend.git
cd tvheadend
makepkg -si
sudo systemctl enable tvheadend.service
sudo chmod -R g+rwx /home/hts

cd ..
git clone https://aur.archlinux.org/kodi-addon-pvr-hts.git
cd kodi-addon-pvr-hts
makepkg -si

cd ..
git clone https://aur.archlinux.org/google-chrome.git 
cd google-chrome
makepkg -si


#
/usr/share/kodi/addons/skin.estuary/xml/DialogButtonMenu.xml
Delete to remove exit button as that will just exit kodi and hang in X11

					<item>
						<label>$LOCALIZE[13012]</label>
						<onclick>Quit()</onclick>
						<visible>System.ShowExitButton</visible>
					</item>

wget https://github.com/StuartIanNaylor/k-a-s/blob/master/plugin.program.chrome.launcher-master.zip?raw=true
```
Install from zip the chrome launcher enable the tvheadend addon and configure.
https://tvheadend.org/ if you require install and setup config
http://hostname:9981/ the admin setup should be found here

