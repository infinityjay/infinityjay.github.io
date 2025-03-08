---
layout: post
title:  Linux - learning notes
categories:
  - Computer system
tags:
  - Linux
  - Learning notes
---

Content

{% include toc %}

# Linux Introduction

LInux is a free and freely disseminated Unix.

> Boot and Login

The boot will start many programs, which are called "daemons" in Linux

The highest privilege account is root

> Shutdown

The shutdown command is: shutdown

```shell
sync # Synchronize data from memory to hard disk;
shutdown # Shutdown command, man shutdown View help document, view related commands;
shutdown -h 10 # Shutdown after ten minutes
shutdown -h now # Shutdown immediately
shutdown -h 20:25 # Shutdown at 20:25 today
shutdown -r now # System restarts now
shutdown -r +10 # System restarts in ten minutes
reboot # Equal to system restart now
halt # Equal to immediate shutdown
```

First run `sync` to save data

## System directory structure

1. Everything is a file;

2. Root directory / , all files are mounted under this node;

![image-20210628154718753](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210628154718753.png)

Directory explanation:

```shell
/bin # bin is the abbreviation of binary, which stores the most frequently used commands
/boot # Stores some core files used when starting Linux, including some link files and image files
/dev # dev is the abbreviation of device, which stores Linux's external devices. The way to access devices in Linux is the same as the way to access files
/etc # Stores all configuration files and subdirectories required for system management
/home # User home directory, each user has his own directory, generally named after the user's account
/lib # Stores the most basic dynamic link shared library of the system, which is similar to the DLL file in Windows
/lost+found # Normally it is empty. When the system is shut down illegally, some files are stored here.
/media # Linux automatically identifies some devices such as USB flash drives, optical drives, etc., and then mounts the identified devices in this directory.
/mnt # You can temporarily mount other file systems for users. You can mount the optical drive on /mnt, and then enter this directory to view the contents of the optical drive.
/opt # The directory where additional software is installed on the host. For example, if you install a new ORACLE database, you can directly put it in this directory. It is empty by default.
/proc # This directory is a virtual directory and a mapping of the system memory. You can directly access this directory to obtain system information.
/root # System administrator directory, also known as the user home directory of the super user.
/sbin # S means super user, and it stores the favorite management programs used by the system administrator.
/srv # Stores some data that needs to be extracted after the service is started.
/sys # This is a new change in the Linux 2.6 kernel. A new file system sysfs that appeared in the 2.6 kernel is installed.
/tmp # This directory is used to store some temporary files.
/usr # Many user applications and files are stored in this directory, similar to the program files directory under Windows
/usr/bin # Applications used by system users
/usr/sbin # Advanced management programs and system daemons used by super users
/usr/src # Default prevention directory for kernel source code
/var # Stores things that are constantly expanding, such as log files, and other frequently modified directories
/run # A temporary file system that stores information since the system was started. When the system is restarted, the files in this directory should be deleted or cleared
```

# Commonly used basic commands

> Absolute path and relative path

**Absolute path:**

The path is written from the root directory /, for example: /usr/share/doc.

**Relative path:**

The path is not written from /, for example, from /usr/share/doc to /usr/share/man, it can be written as: cd ../man This is the relative path.

## Common commands for processing directories

Here are some common commands for processing directories:

- ls: List directories

* -a: List all files, including hidden files (files starting with .) (commonly used)

* -l: List long data strings, including file attributes and permissions, etc. (commonly used)

- cd: Switch directories

- pwd: Display the current directory

- mkdir: Create a new directory

- -m: Configure file permissions! Directly configure, no need to look at the default permissions (umask)~

- -p: Helps you directly create the required directories (including the parent directory) recursively!

- rmdir: delete an empty directory

* -p: delete the "empty" directory at the previous level

- cp: copy files or directories

* ```shell
[root@www ~]# cp [-adfilprsu] source file (source) destination file (destination)
[root@www ~]# cp [options] source1 source2 source3 .... directory
```

* **-a: ** equivalent to -pdr, as for pdr, please refer to the following description; (commonly used)

* **-p: ** copy the file attributes together with the file attributes, instead of using the default attributes (commonly used for backup);

* **-d: ** if the source file is a link file, copy the link file attributes instead of the file itself;

* **-r: ** recursive continuous copy, used for directory copying; (commonly used)

* **-f:** means force. If the target file already exists and cannot be opened, remove it and try again;

* **-i:** If the target file (destination) already exists, it will ask whether to proceed before overwriting (commonly used)

* **-l:** Create a link file of a hard link instead of copying the file itself.

* **-s:** Copy it as a symbolic link file (symbolic link), that is, a "shortcut" file;

* **-u:** If the destination is older than the source, then upgrade the destination!

- rm: Remove files or directories

- * -f: It means force. Ignore non-existent files and no warning message will appear;

* -i: Interactive mode. The user will be asked whether to act before deleting

* -r: Recursive deletion! Most commonly used for deleting directories! This is a very dangerous option! ! !

- mv: move files and directories, or modify the names of files and directories

* ```shell
[root@www ~]# mv [-fiu] source destination
[root@www ~]# mv [options] source1 source2 source3 .... directory
```

* -f: force means if the target file already exists, it will be directly overwritten without asking;

* -i: if the target file (destination) already exists, it will ask whether to overwrite!

* -u: if the target file already exists and the source is newer, it will be upgraded (update)

You can use *man [command]* to view the usage documentation of each command, such as: man cp

## File attributes

The Linux system is a typical multi-user system, and different users are in different positions and have different permissions. In order to protect the security of the system, the Linux system has different regulations for different users to access the same file (including directory files).

In Linux, we can use the `ll` or `ls –l` command to display the attributes of a file and the user and group to which the file belongs.

![image-20210628162708948](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210628162708948.png)

In the example, the first attribute of the boot file is represented by "d". "d" means that the file is a directory file in Linux.

In Linux, the first character represents whether the file is a directory, file, or link file, etc.:

- When it is [ **d** ], it is a directory
- When it is [ **-** ], it is a file;
- If it is [ **l** ], it means a link file;
- If it is [ **b** ], it means an interface device (random access device) for storage in the device file;
- If it is [ **c** ], it means a serial port device in the device file, such as a keyboard or mouse (one-time read device).

The following characters are grouped in groups of three, and all are combinations of the three parameters of "rwx".

Among them, [r] stands for readable (read), [w] stands for writable (write), and [x] stands for executable (execute).

It should be noted that the positions of these three permissions will not change. If there is no permission, a minus sign [-] will appear.

The attributes of each file are determined by the 10 characters in the first part on the left (as shown below): ![image-20210628163137981](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210628163137981.png)

From left to right, these numbers are used to represent.

The 0th bit determines the file type, and the 1st to 3rd bits determine the permissions that the owner (the owner of the file) has on the file. Bits 4-6 determine the permissions that the group (users in the same group as the owner) has for the file, and bits 7-9 determine the permissions that other users have for the file.

Among them:

Bits 1, 4, and 7 represent read permissions. If they are represented by the "r" character, they have read permissions, and if they are represented by the "-" character, they do not have read permissions;

Bits 2, 5, and 8 represent write permissions. If they are represented by the "w" character, they have write permissions, and if they are represented by the "-" character, they do not have write permissions;

Bits 3, 6, and 9 represent executable permissions. If they are represented by the "x" character, they have execute permissions, and if they are represented by the "-" character, they do not have execute permissions.

For a file, it has a specific owner, that is, the user who owns the file.

At the same time, in the Linux system, users are classified by group, and a user belongs to one or more groups.

Users other than the file owner can be divided into users in the same group as the file owner and other users.

Therefore, the Linux system specifies different file access permissions according to the file owner, users in the same group as the file owner, and other users.

In the above example, the boot file is a directory file, and the owner and group are both root.

> Modify file attributes

**1. chgrp: Change the file group**

```shell
chgrp [-R] group name file name
```

-R: Recursively change the file group, that is, when changing the group of a directory file, if you add the -R parameter, the group of all files in the directory will be changed.

**2. chown: Change the file owner, and you can also change the file group at the same time**

```shell
chown [–R] owner name file name
chown [-R] owner name: group name file name
```

==**3. chmod: Change 9 file attributes**==

```shell
chmod [-R] xyz file or directory
```

There are two ways to set Linux file attributes, one is a number and the other is a symbol.

There are nine basic permissions for Linux files, namely owner/group/others, each with its own read/write/execute permissions.

Let's review the data just mentioned: the file permission characters are: "-rwxrwxrwx", these nine permissions are in groups of three! Among them, we can use numbers to represent each permission. The score comparison table of each permission is as follows:

```shell
r:4 w:2 x:1
Readable, writable and executable: 7
Readable, writable and non-executable: 6

chmod 777 filename # Grant all users read, write and execute permissions
```

The scores of the three permissions (r/w/x) of each identity (owner/group/others) need to be accumulated. For example, when the permission is: [-rwxrwx---], the score is:

- owner = rwx = 4+2+1 = 7
- group = rwx = 4+2+1 = 7
- others= --- = 0+0+0 = 0

```shell
chmod 770 fil

