# dracut-sshd-RHEL9
## dracut-sshd setup for RHEL9 / Rocky9
This setup is tested on a Rocky Linux 9 machine with encrypted disk set up during install.

### 1. Enable copr repo
```
# dnf copr enable gsauthof/dracut-sshd
```

### 2. Install dracut-sshd
```
# dnf install dracut-sshd
```

### 3. Make sure the root user is able to ssh in with publickey

Check /etc/shadow
```
# head -1 /etc/shadow
root:*::0:99999:7:::
```
Make sure the second field is _*_ and not _!_

Add your public key of choice to */root/.ssh/authorized_keys*
Check the mode of the file, should be 600

Check that you can ssh in as root!

### 4. Rebuild dracut
```
# dracut -v -f
```

### 5. Add networking parameters to kernel cmdline
To make sure the machine get networking in early boot stage we have to add kernel parameters

Edit */etc/default/grub*, change the line starting with GRUB_CMDLINE_LINUX=

```
# cat /etc/default/grub | grep GRUB_CMDLINE_LINUX
GRUB_CMDLINE_LINUX="resume=/dev/mapper/rl-swap rd.luks.uuid=luks-XXXXXXXXXXXXXXXXXXXXXXXXXXXX rd.lvm.lv=rl/root rd.lvm.lv=rl/swap rhgb quiet rd.neednet=1 ip=10.0.0.199::10.0.0.1:255.255.255.0:hostname:eno1:off"
```

The added part is rd.neednet=1 and the rest of the line.

*10.0.0.199* is the static ip of the host

*10.0.0.1* is the gateway

*hostname* is the hostname

*eno1* is the name of the network device

For more details see [Configuring IP Networking from the Kernel Command line](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_from_the_kernel_command_line)

Update grub config
```
# grub2-mkconfig -o /etc/grub2-efi.cfg
# grub2-mkconfig -o /etc/grub2.cfg
```

Reboot!

### 6. Test the setup
After reboot ssh in to your host as root from another host

```
$ ssh -i privatekey_for_root_on_host root@10.0.0.199

Welcome to the early boot SSH environment. You may type
￼
￼   systemd-tty-ask-password-agent
￼
(or press "arrow up") to unlock your disks.

This shell will terminate automatically a few seconds after the
unlocking process has succeeded and when the boot proceeds.  ￼

initramfs-ssh:/root# 
```

If you press arrow up and enter you should get a prompt to enter the LUKS decrypt key. After a correct key is entered, you will loose connection and the boot will continue as normal.

If you change the public key you need to rerun the dracut -v -f step.
If you change ip address you need to rerun the grub step to reflect the changes. 
