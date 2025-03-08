---
title:  HPC performance test
categories:
  - Cloud computing
tags:
  - Ali cloud
  - Performance test
  - Project experience
---
Content

{% include toc %}

# 1. Host_manage part

## Use of cron package

* Project address

https://pkg.go.dev/github.com/robfig/cron@v1.2.0

* Import format

Import ("github.com/robfig/cron")

* Expression

```go
c := cron.New()
c.AddFunc("0 30 * * * *", func() { fmt.Println("Every hour on the half hour") })
c.AddFunc("@hourly", func() { fmt.Println("Every hour") })
c.AddFunc("@every 1h30m", func() { fmt.Println("Every hour thirty") })
c.Start()
```

The six characters represent:

```json
Field name | Mandatory? | Allowed values ​​| Allowed special characters
---------- | ---------- | --------------- | ----------------------------
Seconds | Yes | 0-59 | * / , -
Minutes | Yes | 0-59 | * / , -
Hours | Yes | 0-23 | * / , -
Day of month | Yes | 1-31 | * / , - ?
Month | Yes | 1-12 or JAN-DEC | * / , -
Day of week | Yes | 0-6 or SUN-SAT | * / , - ?
```

*: each;

? : Contrary to the meaning of *, it represents a cycle other than weekly or monthly

,: List the values ​​separately

/: 0 3-59/15 11 * * * represents a cycle every 15 minutes from the 3rd minute to the 59th minute at 11 o'clock every day

## Use gin package to receive return data

## Goroutine

## statuspage

Report data points to statuspage for real-time monitoring.

* When reporting data, you need to add apikey information in the header;

* The timestamp format needs to be converted to unixTime format, and the time needs to be pushed forward from the current time to the time when the program is executed;

* The data passed in should be in float format, but the API interface does not specify it;

# 2. Client part

## The program gets stuck as soon as it starts running

When returning host information, use session to create powershell and execute commands. It will get stuck when closing the session.

* Using metadata metadata interface

Accessing Alibaba Cloud metadata directly through http method can return mac address, private IP, instance ID

* Using cpu package (see below)

Can return logical CPU and physical CPU information

## Performance analysis--pprof

## Error analysis (panic, defer, recover)

## cpu package that returns host cpu information

* Project address

https://pkg.go.dev/github.com/shirou/gopsutil

* Import format

import（""github.com/shirou/gopsutil/v3/cpu"")

* Function

[func Counts(logical bool) (int, error)](https://pkg.go.dev/github.com/shirou/gopsutil@v3.21.4+incompatible/cpu#Counts)

Input: true/false

Return: logical CPU number / physical CPU number

# 3. Problems with deployment on Windows systems

## 1. Packaging images

To use packer to package custom images, you need to download packer and then construct a json file for packaging. The command is packer build alicloud.json

```json
{
"variables": {
"access_key": "XXX",
"secret_key": "XXX"
},
"builders": [{
"type":"alicloud-ecs",
"access_key":"user access_key",
"secret_key":"user secret_key",
"region":"cn-beijing",
"zone_id": "cn-beijing-h",
"image_name":"xxx",
"source_image":"xxx",
"ssh_username":"root",
"instance_type":"xxx",
"internet_charge_type":"PayByTraffic",
"io_optimized":"true", 
"image_force_delete":"true",
 "communicator": "winrm",
 "winrm_port": 5985,
 "winrm_username": "Administrator",
 "winrm_password": "xxx",
 "user_data_file": "winrm_enable_userdata.ps1"
 }],
 "provisioners": [
 {
 "type": "file",
 "source":"/Users/user/my_git/hpc-client_aliyun/etc/file_list.json",
 "destination": "C:\\Users\\Administrator\\Desktop\\test\\file_list.json"
 },
 {
 "type": "file",
 "source":"/Users/user/my_git/hpc-client_aliyun/etc/us3config.json",
 "destination": "C:\\Users\\Administrator\\Desktop\\test\\us3config.json"
 },
 {
 "type": "file",
"source":"/Users/user/my_git/hpc-client_aliyun/build/hpc-client.exe",
"destination": "C:\\Users\\Administrator\\Desktop\\test\\hpc-client.exe"
},
{
"type": "file",
"source":"/Users/user/my_git/hpc-client_aliyun/build/win_download.exe",
"destination": "C:\\Users\\Administrator\\Desktop\\test\\win_download.exe"
}

]
}
```

* Line 19 - line 23

The difference from Linux is that it uses winrm communication instead of ssh communication. It is necessary to configure the relevant winrm parameters, but it cannot be found on the Alibaba Cloud API. Username and password are the username and password of the host created with the image.

* line25

For the "provisioners" part, please refer to the official documentation of packer alicloud. You can use powershell to set the command line;

## 2. Set the startup program to start automatically

* 1. Cancel the startup CTRL+ALT+DEL

<1>Start->Administrative Tools->Local Security Policy->"Local Policies" and "Security Options" expand "Interactive Login";

<2>Double-click "Do not need to press CTRL+ALT+DEL" and select Enable;

* 2. Set the automatic login account

<1>Start->Run control userpasswords2, and the user account dialog box pops up.
<2>Cancel "To use this computer, the user must enter a password."
<3>Press Ctrl+Shift+A, the "Automatic Login" dialog box pops up, and set the user account and password.

* 3. Set the program to start automatically

Put the program shortcut in the startup folder, "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"

## 3. The program on the Alibaba Cloud host will terminate execution for no reason

**Solution 1: The client adds a status update, and the server creates a coroutine for inspection**

Step 1: The client sends data

* When the server creates the host, the status value is 0. After the client runs the command line parameters, the program can generally start normally. After the client runs the command line parameters, you can add a POST method to report to the server and set the status to 2 to indicate that the program has started running;

Step 2: The server receives data

* The server can use the POST method to receive parameters, and use the private IP as an identifier to modify the status value in the database to 2;

Step 3: The server creates a coroutine detection

* Add a coroutine in the server main function, and check the status in the database every 5 minutes to see if it is still 0 The current time is compared with the time in the record. If it exceeds 20 minutes, it can be regarded as that the client program has not been successfully started. Then the host is deleted (the record is deleted in the database after the host is successfully deleted) and the host is recreated.

goroutine, coroutine

Ticker loop execution in the time package

