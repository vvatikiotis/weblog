---
author: Vassilis Vatikiotis
title: Notes on mbox to maildir migration
tags:
  - linux
---

# Notes on mbox to maildir migration

- The `mb2md` command takes absolutes paths.
- `mb2md -s mbox-path -d maildir-path`.
- The `maildir-path` is created.
- Maildir folder structure is:

```picture
Maildir folder
|
|-- new
|-- cur
|-- tmp
```

- It _seems_ safe to convert mboxes to maildirs while the MTA is active and delivering mails to the same folder. **Need to double check this!**.
- Need to deactivate or update users' `.procmailrc`.
- The MTA builds the Maildir folder structure for the INBOX, i.e. we don't need to do anything.
- No need to create the Maildir base folder as well as the Sent/Drafts/Trash folders by hand. IMAP takes care of this.
- Maildir folders are named with a dot e.g. .Sent or .Trash. This is controllled by the IMAP server (dovecot).
- The `subscriptions` file lists all the folders monitored by IMAP.
