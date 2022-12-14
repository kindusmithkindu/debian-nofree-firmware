# Unofficial solution for debian nofree firmware

1. First, go to download the linux kernel official mainline firmware package from this link :

```
   https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/ 
```  

2. Unzip the firmware package

```
   tar -zxvf linux-firmware-main.tar.gz
```
3. Move the old firmware package under the /lib directory to the firmware.old directory

```
   sudo mv /lib/firmware /lib/firmware.old
```
4. Move the downloaded new firmware package to /lib

```
   sudo mv ~/linux-firmware-main /lib/firmware
```
5. Download all regulatory files in this repository to your personal directory, and copy them to /lib/firmware directory

```
   sudo cp ~/regulatory* /lib/firmware/
```
6. Update initramfs

```
   sudo update-initramfs -u -k all
```
   
7. or, a fresh install:

```
   sudo update-initramfs -d -k all && sudo update-initramfs -c -k all
```
8. At this point, you have basically solved most of the firmware problems, but there are still two small datum test packages that cannot be installed. Of course, these are optional dependencies and you can completely ignore them. Use the command of “sudo dmesg|grep fail” , you should see two error messages, please ignore them:

```
  ath10k_pci 0000:07:00.0: firmware: failed to load ath10k/pre-cal-pci-0000:07:00.0.bin
  ath10k_pci 0000:07:00.0: firmware: failed to load ath10k/cal-pci-0000:07:00.0.bin
```
9. Of course the firmware will be updated from time to time, you can also write a “firmware.service” service, move it to /etc/systemd/system, and  based on this service write a timer of “firmware.timer” for firmware update. Please replace yourname in the script with your personal user name

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

10. Create a new firmware directory and firmware script under your personal directory, and grant executable permissions, at the same time Create regulatory directory under the firmware directory and copy all of regulatory file in the repository to this regulatory directory
```
   mkdir ~/firmware && touch ~/firmware/firmware && chmod +x ~/firmware/firmware
```

```
   mkdir ~/firmware/regulatory && cp regulatory* ~/firmware/regulatory
```

11. Use vim or nano to edit this firmware script

```
   vim ~/firmware/firmware
```
12. Add the following code. Yourname in the script needs to be replaced with your personal username:

```
  #!/bin/bash
  axel -n10 https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/ && \
  tar -zxvf /home/yourname/firmware/linux-firmware-main.tar.gz -C /lib && \
  rm -rf /lib/firmware.old && mv /lib/firmware /lib/firmware.old && mv /lib/linux-firmware-main /lib/firmware && \
  cp /home/yourname/firmware/regulatory/* /lib/firmware && \
  update-initramfs -u -k all && rm -rf /home/yourname/firmware/linux*
```
13. Now it's the time to start the service

```
  sudo systemctl enable --now firmware.service 
```

```
  sudo systemctl enable --now firmware.timer
```
14. Done
15. If you have any questions, please go to issue to discuss
