---
layout: post
published: true
title: Kill a Running PHP Cron Job
date: '2015-11-21'
---
If you started a long-running PHP script as a cron job and wish to terminate it for whatever reason.  Here is what you can do.
## Step 1: Run the following command to find the Process ID
```
ps -ef | grep php
```
"ps -ef" is basically to show all (-e) processes in full details (-f) and the output is pipe (with ’|’) through “grep php” to find all processes with the word “php”.  From the output, you should be able to identify the process and its ID.

## Step 2: Kill it with the following command
```
kill -9 <pid>
```



