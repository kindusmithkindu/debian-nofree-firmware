# debian-nofree-firmware
Unofficial solution for debian nofree firmware
============================================================

1. First, go to download the linux kernel official mainline firmware package from this link :

```
 https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/ 
```  

2. Unzip the firmware package

```
   sudo tar -zxvf linux-firmware-main.tar.gz
```
3. Move the old firmware package to the firmware.old directory

```
   sudo mv /lib/firmware /lib/firmware.old
```
4. Move the downloaded new firmware package to /lib

```
   sudo mv ~/linux-firmware-main /lib/firmware
```
5. Download all regulatory files in this repository and move them under /lib/firmware

```
   sudo cp ~/regulatory* /lib/firmware/
```
6. Update initramfs

```
   sudo update-initramfs -u -k all
```
  or, a fresh install

```
   sudo update-initramfs -d -k all && sudo update-initramfs -c -k all
```
7. At this point, you have basically solved most of the firmware problems, but there are still two small datum test packages that cannot be installed. Of course, these are optional dependencies and you can completely ignore them. Use the command of “sudo dmesg|grep fail” , you should see two error messages, please ignore them:

```
  ath10k_pci 0000:07:00.0: firmware: failed to load ath10k/pre-cal-pci-0000:07:00.0.bin
```

```
  ath10k_pci 0000:07:00.0: firmware: failed to load ath10k/cal-pci-0000:07:00.0.bin
```
8. But the firmware is updated from time to time, you can also set a “firmware update” service, move it to /etc/systemd/system, and set a “firmware.timer” based on this service

```
firmware.service:

[Unit]
Description="Use firmware script to install the Linux firmware"

[Service]
WorkingDirectory=/home/yourname/firmware
ExecStart=/bin/sh -c '/home/yourname/firmware/firmware'

```

```
firmware.timer:

[Unit]
Description="Linux firmware install"

[Timer]
OnCalendar=weekly
AccuracySec=12h
Persistent=true

[Install]
WantedBy=timers.target
```

9. Please replace yourname in the text with your personal user name, and create a new firemware directory and firmware script, and grant executable permissions

```
   sudo mkdir ~/firmware  
```

```
   sudo touch ~/firmware/firmware
```

```
   sudo chmod +x firmware
```
10. Use vim or nano to edit this firmware script

```
   sudo vim ~/firmware
```
11. Add the following code. Yourname in the text needs to be replaced with your personal username:

```
#!/bin/bash

axel -n10 https://kernel.source.codeaurora.cn/pub/scm/linux/kernel/git/firmware/linux-firmware.git/snapshot/linux-firmware-main.tar.gz
&& \
tar -zxvf /home/yourname/firmware/linux-firmware-main.tar.gz -C /lib && \
rm -rf /lib/firmware.old && mv /lib/firmware /lib/firmware.old && mv /lib/linux-firmware-main /lib/firmware && \
cp /home/yourname/firmware/regulatory/* /lib/firmware && \
update-initramfs -u -k all
```
12. Now is the time to start the service

```
  sudo systemctl enable --now firmware.service 
```

```
  sudo systemctl enable --now firmware.timer
```
13. Done
