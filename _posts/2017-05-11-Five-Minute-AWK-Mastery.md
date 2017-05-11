---
layout: post
title: "Five Minute AWK Mastery"
date: 2017-05-11 00:30
categories: awk linux scripting
---
I see alot of people thinking AWK is a hard skill to pickup. This is not the case, to effectively use AWK you only need to know a few things.

For the below examples I will be using `df -h` as the data source

The output:

```
 pbove@PHIL:/mnt/c/Users/pbove$ df -h
 Filesystem      Size  Used Avail Use% Mounted on
 rootfs          112G  106G  5.2G  96% /
 tmpfs           112G  106G  5.2G  96% /run
 none            112G  106G  5.2G  96% /run/lock
 none            112G  106G  5.2G  96% /run/shm
 none            112G  106G  5.2G  96% /run/user
```

1. Basic AWK Syntax:

	The basic syntax of you basic AWK one liner is as follows:
	`$>program | awk '{print $1}'`
	This will take the output from "program" and pass it to the AWK interpreter as the input. The `'{}'` block surrounded by single quotes is an AWK "operation block" that will be executed on every line that is passed through the `|`, `print $1` uses the "print" function in AWK to print the built in variable $1.

	Running this on our `df -h` output would get us this:
	```	
	pbove@PHIL:/mnt/c/Users/pbove$ df -h | awk '{print $1}'
	Filesystem
	rotfs
	tmpfs
	none
	none
	none
	```
	You can also use operators to determine whether the  block should run:

	```
	pbove@PHIL:/mnt/c/Users/pbove$ df -h | awk '$1 == "Filesystem" {print $1}'
	Filesystem
	```


2. AWK Built-in Variables:

	As you may have guessed from the previous outputs `$1` in AWK refers to the first field in the current line, there is a `$d` for every field in a line. An AWK field is by default delimited by one tab or space.


	$0 is the entire line.

	NR is the amount of records read.
	```
	df -h | awk 'NR==2 {print $5}'
	96%
	```
	FNR is the amount of records read in the current file/input.
	```
	df -h | awk 'NR==2 {print $5}'
	96%
	```
The difference between the NR and FNR variable do not matter when you are taking one input "file" from the pipe, however you need to be careful if you pull in multiple files because NR will keep counting while FNR resets per file.


There you go, this should arm with with enough AWK knowledge to be useful. If you are interested in some more in depth reading I recommend  "The AWK Programming Language" written by the creators of AWK if you can snag it for a reasonable price.
