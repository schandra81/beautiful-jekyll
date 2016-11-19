---
layout: post
published: true
title: Important Performance Counters
date: 2016-3-21
---
Performance Counters are very important to evaluate. There are more than thousands of Performance Counters. Today I will cover three basic but very important Performance Counters.

### Processor:% Processor Time
It reports the total processor time with respect to the available capacity of the server. If counter is between 50 to 70 % consistently, investigate the process which is taking long time.

### PhysicalDisk:Avg.Disk Queue Length
It indicates wait time for processes to use disk resources. As a disk is reading and writing data some requests cannot be immediately filled, those requests are queued. If many simultaneous requests are waiting, investigate the process which is taking long time.

### PhysicalDisk:Disk Read Bytes/sec and PhysicalDisk :Disk Write Bytes/sec
It report the number of bytes read from and written to the disk, respectively. Slow SELECT queries with high physical reads and low queue lengths demonstrates under performance of Disk read and write. Index optimization can reduce this problem.

You can watch above performance counters using windows perfmon utility.
