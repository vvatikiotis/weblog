---
Author: Vassilis Vatikiotis
Tags: Linux, Virtualisation, oVirt
Date: 17 June 2019
Title: Installing oVirt self Hosted
---

# Installing oVirt self Hosted

Install iVirt packages
1. `# yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release43.rpm`
2. `# yum update`

Chapter 7: Enterprise Linux Hosts Installing Enterprise Linux Hosts
3. `# yum install cockpit-ovirt-dashboard`
4. `# systemctl enable cockpit.socket`
5. `# systemctl start cockpit.socket`
6. Login at `https://HostFQDNorIP:9090` (with admin account)

7. `# systemctl enable ntpd`
8. `# systemctl start ntpd`

Self Hosted
Chapter 2: Deploying the Self-Hosted Engine

9. DNS, forward and reverse should be setup

# ALL THE ABOVE ARE NULL
