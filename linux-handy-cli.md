---
Author: Vassilis Vatikiotis
Tags: React Javascript
Date: 21 August 2018
Title: Linux Handy CLI Commands
---

# Linux Handy CLI Commands

## Find stuff

- `find . -type f -print0 | xargs -0 mv -t/path/to/target-dir`
- `find . -type d -name .Trash -print | xargs -I DIR echo ls DIR/cur`

## Mail shit

- When I need to remove some spam messages from the postfix queue:

  `postqueue -p | tail -n +2 | grep -v '^ *(' | awk 'BEGIN { RS = "" }{ if ($7 == "info@micr.com") print $1}' | postsuper -d -`

Ref: http://www.postfix.org/postsuper.1.html

- Sometimes mails from MAILER-DAEMON will be produced, as a result of spam attacks. In that case, check for sender == MAILER-DAEMON and 1st recipient == info@micr.com, in our example:

  `postqueue -p | tail -n +2 | grep -v '^ *(' | awk 'BEGIN { RS = "" }{ if ($8 == "info@micr.com" && $7 == "MAILER-DAEMON" print $1}' | postsuper -d -`

- CHECK ALSO THE ACTIVE QUEUE: There may be left over spams, ready to go

- Sometimes it's necessary to put active and/or deferred message on hold, clean up the exclamation mark (!), delete them and then release them.
- Put all messages in deferred queue on hold:

  `postsuper -h ALL deferred`

- Delete the as per above instructions and
- Release all message in deferred queue:

  `postsuper -H ALL`

* Check every n minutes queue size and email if exceeds a limit

  > ```bash
  > #!/bin/sh
  >
  > postqueue -p | awk '
  >   BEGIN { limit = 10240 }
  >   /^-- .+ Kbytes in .+ Request/ { queue_len = $5}
  >   END {
  >      if (queue_len > limit) {
  >          print "Mail queue size is too large: ", queue_len;
  >          system("echo \"Message queue size is $queue_len\" | mail -s \"Mail message queue too long\" root");
  >      }
  >   }
  > '
  > ```

  Ref: [Alert-of-unusually-large-queue](http://postfix.1071664.n5.nabble.com/Alert-of-unusually-large-queue-td51546.html)
