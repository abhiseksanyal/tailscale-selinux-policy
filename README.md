# tailscale-selinux-policy
SELinux policy for Tailscale

# Supported environment
1. Fedora 37

# Supported features
1. Service Start and Stop
2. Tailscale SSH

# Steps to build
## Pre-requisites
```shell
dnf install selinux-policy-devel
```

## Clone the policy source
```shell
git clone git@github.com:abhiseksanyal/tailscale-selinux-policy.git
cd tailscale-selinux-policy
make -f /usr/share/selinux/devel/Makefile tailscaled.pp
```
This will create the policy file "tailscaled.pp"

# Steps to test

Environment tested on
- Tailscale 1.34.2
- Fedora 37
- Kernel ?

## 1 - Check context of an unconfined Tailscale service
```shell
ps -q $(pidof tailscaled) -o pid,label,comm
```
Output will be something like
```text
    PID LABEL                                     COMMAND
 221929 system_u:system_r:unconfined_service_t:s0 tailscaled
```
NOTE: You can also run something like  ```ps -eafZ```

Stop tailscale service
```shell
sudo systemctl stop tailscaled
```

## 2 - Load the SELinux policy
```shell
sudo semodule -i tailscaled.pp
```

## 3 - Set the context for Tailscale files
```shell
sudo restorecon /usr/sbin/tailscaled
sudo restorecon /lib/systemd/system/tailscaled.service
sudo restorecon -R /var/lib/tailscale
sudo restorecon -R /var/run/tailscale
```
This is required only once, until Tailscale is reinstalled
NOTE: Ignore restorecon error, if it fails to find "/var/run/tailscale"

## 4 - Start Tailscale and check the context again
Start tailscale service
```shell
sudo systemctl start tailscaled
```

Check the context
```shell
ps -q $(pidof tailscaled) -o pid,label,comm
```
Output will be something like
```text
    PID LABEL                             COMMAND
 222705 system_u:system_r:tailscaled_t:s0 tailscaled
```
NOTE: You can also run something like  ```ps -eafZ```

Tailscale service is now running as a confined service with a context of "tailscaled_t"

# Steps to revert and unload the SELinux policy
1. Stop tailscale service
2. Unload the SELinux policy using the following command
```shell
sudo semodule -r tailscaled
```
3. Restore the context for Tailscale files
```shell
sudo restorecon /usr/sbin/tailscaled
sudo restorecon /lib/systemd/system/tailscaled.service
sudo restorecon -R /var/lib/tailscale
sudo restorecon -R /var/run/tailscale
```
NOTE: Ignore restorecon error, if it fails to find "/var/run/tailscale"

4. Start tailscale service 
