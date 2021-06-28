---
layout: post
title: HPC性能测试部署阿里云避坑指南
---

# 一、Host_manage部分

## cron包的使用

* 项目地址

https://pkg.go.dev/github.com/robfig/cron@v1.2.0

* import格式

Import ("github.com/robfig/cron")

* 表达式

```go
c := cron.New()
c.AddFunc("0 30 * * * *", func() { fmt.Println("Every hour on the half hour") })
c.AddFunc("@hourly",      func() { fmt.Println("Every hour") })
c.AddFunc("@every 1h30m", func() { fmt.Println("Every hour thirty") })
c.Start()
```

六个字符分别代表：

```json
Field name   | Mandatory? | Allowed values  | Allowed special characters
----------   | ---------- | --------------  | --------------------------
Seconds      | Yes        | 0-59            | * / , -
Minutes      | Yes        | 0-59            | * / , -
Hours        | Yes        | 0-23            | * / , -
Day of month | Yes        | 1-31            | * / , - ?
Month        | Yes        | 1-12 or JAN-DEC | * / , -
Day of week  | Yes        | 0-6 or SUN-SAT  | * / , - ?
```

*：每一个；

？：与*的含义相反，代表并非每周或者每月的周期

，：单列出来那几个数值

/：0 3-59/15 11 * * * 代表每天11时第3分钟到59分钟每隔15分钟循环一次

## 使用gin包接收返回数据







## 协程goroutine





## statuspage

将数据点上报到statuspage进行实时监控。

* 在上报数据的时候需要在header中加上apikey的信息；
* 时间戳的格式需要转换成为unixTime的格式，时间需要从现在的时间往前推到程序执行的时间；
* 传入的数据应该为float格式，但是API接口没有说明；





# 二、client部分

## 程序刚开始运行就直接卡住

返回主机信息的时候利用session创建powershell，执行命令，在关闭session的时候会卡住不动。

* 利用元数据metadata接口

直接通过http方法访问阿里云metadata，可以返回mac地址、private IP、实例ID

* 利用cpu包（见下）

可以返回逻辑CPU以及物理CPU信息

## 性能剖析--pprof





## 错误分析（panic、defer、recover）





## 返回主机cpu信息的cpu包

* 项目地址

https://pkg.go.dev/github.com/shirou/gopsutil

* import格式

import（""github.com/shirou/gopsutil/v3/cpu""）

* 函数

[func Counts(logical bool) (int, error)](https://pkg.go.dev/github.com/shirou/gopsutil@v3.21.4+incompatible/cpu#Counts)

输入：true/false

返回：logical CPU数量 / 物理CPU数量

# 三、部署在Windows系统上的问题

## 1、打包镜像

利用packer打包自定义镜像，需要下载packer，然后构造json文件进行打包，命令为packer build alicloud.json

```json
{
    "variables": {
      "access_key": "XXX",
      "secret_key": "XXX"
    },
    "builders": [{
      "type":"alicloud-ecs",
      "access_key":"{{user `access_key`}}",
      "secret_key":"{{user `secret_key`}}",
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

  与Linux的区别，不是使用ssh通信而是使用winrm通信，需要配置相关winrm参数，但是阿里云API上找不到。username与password就是用镜像创建主机的用户名与密码。

* line25

  "provisioners"部分可以参考 packer  alicloud 的官方文档，可以用 powershell设置命令行；



## 2、设置开机程序自动启动

* 1. 取消开机CTRL+ALT+DEL

<1>开始->管理工具->本地安全策略->"本地策略"和"安全选项"展开"交互式登陆"；
<2>双击"不需要按CTRL+ALT+DEL"选中启用；

* 2. 设置自动登录账户

<1>开始->运行 control userpasswords2，弹出用户帐号对话框。
<2>取消“要使用本机，用户必须输入密码”。
<3>按下Ctrl+Shift+A，弹出“自动登录”对话框，设置好用户帐号和密码。

* 3.设置程序自动启动

将程序快捷方式放在startup文件夹中，"C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"



## 3、阿里云主机上程序会无故终止执行

**解决方案1: client端加上状态更新，服务端创建协程进行检查**

Step1：客户端发送数据

* 服务端创建主机时，status值为0，在客户端运行完命令行参数之后，程序一般都能正常启动，可以在客户端运行完命令行参数之后加一个POST方法向服务端上报，将status置为2，表明程序开始运行;



 Step2：服务端接收数据

* 服务端可以使用POST方法接收参数，并且以private IP作为标识修改数据库中status值为2；



 Step3：服务端创建协程检测

* 在服务端main函数中增加一个协程，每隔5分钟检查数据库中status依旧为 0 的记录，将当前时间与记录中的时间对比，超过20min可以视为客户端程序没有成功启动，然后执行删除主机（主机删除成功后在数据库中删除该条记录）并且重新创建主机。

goroutine，协程

time包中的Ticker循环执行





































