---
Author: Vassilis Vatikiotis
Tags: Linux, File systems
Date: 15 June 2019
Title: Parted notes
---

# SSHing and open remote GUIs

- From client `ssh -X -Y <direct-ip>. Do not go via another machine and then another machine (although this could work via local redirect -L)
- In server's sshd_config use X11UseLocalhost no (work with -Y and it's
  necessary for MacOS users?)

## References

- https://unix.stackexchange.com/questions/83949/with-ssh-x11-forwarding-ssh-x-get-cant-open-display-trying-to-run-x-app


