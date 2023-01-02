# ansible-proxmox-ve-setup
Role to configure basic settings for Proxmox VE.

## Features
Disable the Enterprise Repo in favor of the Community Repo.  
Disable the Subscription Nag with an APT hook to reapply after updates.  
Install the Dark Theme with an APT hook to reapply after updates.

## Usage
playbook.yaml
```
---
- name: Play Name
  hosts: MyHosts
  
  roles:
    - role: notmycloud.proxmox_ve_setup
      vars:
        proxmox_ve_enterprise: # Set to true if you want to keep the enterprise repo
        proxmox_ve_lib_file: # Override the default location of the subscription nag file
```

## TODO

- [ ] APT Disable extra languages
- [ ] Periodic Update of Package Lists
- [ ] Periodic download of updates (Not automatic install)
- [ ] Force APT over IPv4 only
- [ ] Pveam Update for container list
- [ ] ZFS Auto Snapshot?
- [ ] Ensure ksmtuned (ksm-control-daemon) is enabled and optimise according to ram size. /etc/ksmtuned.conf
  - Up to 16GB RAM
    - KSM_THRES_COEF=50
    - KSM_SLEEP_MSEC=80
  - Up to 32GB RAM
    - KSM_THRES_COEF=40
    - KSM_SLEEP_MSEC=60
  - Up to 64GB RAM
    - KSM_THRES_COEF=30
    - KSM_SLEEP_MSEC=40
  - Up to 128GB RAM
    - KSM_THRES_COEF=20
    - KSM_SLEEP_MSEC=20
  - More than 128GB RAM
    - KSM_THRES_COEF=10
    - KSM_SLEEP_MSEC=10
- [ ] Disable rpcbind - Not needed on NFSv4 only servers
- [ ] Get and set server timezone $(curl https://ipapi.co/timezone) outputs UNIX Timezone name for the public IP.
- [ ] Enable NTP
- [ ] Fail2Ban
- [ ] https://github.com/stuvusIT/systemd-journald
- [ ] Optimize [/etc/vzdump.conf](https://pve.proxmox.com/pve-docs/chapter-vzdump.html#:~:text=point%20of%20view.-,Configuration,-Global%20configuration%20is). Use a template, options specified below will be opionionated.
  - `bwlimit: 0` Some reports that manually setting this to zero improves performance
  - `compress: zstd` ZStandard is currently the best available compressor
  - `ionice: 5` Range from 0-8, default 7. Lower increases the IO prioroity
  - `notes-template: {{ cluster }}.{{ node }}.{{ guestname }}.{{ vmid }}`
- [ ] Sysctl Optimizations
  - [ ] Memory optimizations
    - `vm.swappiness=10`
    - `vm.min_free_kbytes=1048576`
    - `vm.nr_hugepages=72`
  - [ ] TCP Optimizations
    - `net.core.default_qdisc=fq`
    - `net.ipv4.tcp_congestion_control=bbr`
    - `net.ipv4.tcp_fastopen=3`
  - [ ] Network Optimizations
    - `net.core.netdev_max_backlog=8192`
    - `net.core.optmem_max=8192`
    - `net.core.rmem_max=16777216`
    - `net.core.somaxconn=8151`
    - `net.core.wmem_max=16777216`
    - `net.ipv4.conf.all.accept_redirects = 0`
    - `net.ipv4.conf.all.accept_source_route = 0`
    - `net.ipv4.conf.all.log_martians = 0`
    - `net.ipv4.conf.all.rp_filter = 1`
    - `net.ipv4.conf.all.secure_redirects = 0`
    - `net.ipv4.conf.all.send_redirects = 0`
    - `net.ipv4.conf.default.accept_redirects = 0`
    - `net.ipv4.conf.default.accept_source_route = 0`
    - `net.ipv4.conf.default.log_martians = 0`
    - `net.ipv4.conf.default.rp_filter = 1`
    - `net.ipv4.conf.default.secure_redirects = 0`
    - `net.ipv4.conf.default.send_redirects = 0`
    - `net.ipv4.icmp_echo_ignore_broadcasts = 1`
    - `net.ipv4.icmp_ignore_bogus_error_responses = 1`
    - `net.ipv4.ip_local_port_range=1024 65535`
    - `net.ipv4.tcp_base_mss = 1024`
    - `net.ipv4.tcp_challenge_ack_limit = 999999999`
    - `net.ipv4.tcp_fin_timeout=10`
    - `net.ipv4.tcp_keepalive_intvl=30`
    - `net.ipv4.tcp_keepalive_probes=3`
    - `net.ipv4.tcp_keepalive_time=240`
    - `net.ipv4.tcp_limit_output_bytes=65536`
    - `net.ipv4.tcp_max_syn_backlog=8192`
    - `net.ipv4.tcp_max_tw_buckets = 1440000`
    - `net.ipv4.tcp_mtu_probing = 1`
    - `net.ipv4.tcp_rfc1337=1`
    - `net.ipv4.tcp_rmem=8192 87380 16777216`
    - `net.ipv4.tcp_sack=1`
    - `net.ipv4.tcp_slow_start_after_idle=0`
    - `net.ipv4.tcp_syn_retries=3`
    - `net.ipv4.tcp_synack_retries = 2`
    - `net.ipv4.tcp_tw_recycle = 0`
    - `net.ipv4.tcp_tw_reuse = 0`
    - `net.ipv4.tcp_wmem=8192 65536 16777216`
    - `net.netfilter.nf_conntrack_generic_timeout = 60`
    - `net.netfilter.nf_conntrack_helper=0`
    - `net.netfilter.nf_conntrack_max = 524288`
    - `net.netfilter.nf_conntrack_tcp_timeout_established = 28800`
    - `net.unix.max_dgram_qlen = 4096`
- [ ] ZFS ARC Optimizations /etc/modprobe.d/99-zfs-arc.conf
  - `options zfs zfs_arc_min=` 1/16th RAM or minimum 1GB
  - `options zfs zfs_arc_max=` 1/8th RAM
- [ ] Ensure `/etc/network/interfaces` sources `.d` configs
  - `echo "source /etc/network/interfaces.d/*" >> /etc/network/interfaces`
- [ ] Enable IOMMU
  - Intel `sed -i 's/quiet/quiet intel_iommu=on iommu=pt/g' /etc/default/grub`
  - AMD `sed -i 's/quiet/quiet amd_iommu=on iommu=pt/g' /etc/default/grub`
  - `/etc/modules`
    - vfio
    - vfio_iommu_type1
    - vfio_pci
    - vfio_virqfd
- [ ] Blacklist GFX modules `/etc/modprobe.d/blacklist-gfx.conf`
  - blacklist nouveau
  - blacklist lbm-nouveau
  - options nouveau modeset=0
  - blacklist amdgpu
  - blacklist radeon
  - blacklist nvidia
  - blacklist nvidiafb
- [ ] Propogate Settings
  - update-initramfs -u -k all
  - update-grub
  - pve-efiboot-tool refresh
  

## Support
For support, please raise an issue and provide the following items
- Sample task/playbook to replicate your issue
- Resultant file that is created.
- If modifying an existing file, please provide the unmodified version as well.
