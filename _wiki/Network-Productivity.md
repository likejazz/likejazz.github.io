---
layout: wiki 
title: Network Productivity
tags: ["Productivity"]
last_modified_at: 2024/06/25 14:47:08
---

<!-- TOC -->

- [ssh without password](#ssh-without-password)
- [rsync](#rsync)
- [dig `+trace`](#dig-trace)
- [Port Forwarding](#port-forwarding)
  - [ssh 접속 via proxy](#ssh-접속-via-proxy)
- [nload](#nload)
- [Reverse Proxy](#reverse-proxy)
  - [nohup without Docker](#nohup-without-docker)

<!-- /TOC -->

# ssh without password
```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub -p 300XX xxx-user@10.12.XX.XX
```

# rsync
두 디렉토리 동기화

```
$ rsync -avzh \
	--dry-run \
	--progress \
	~/Documents/BACKUPZ/2020/ ./2020/
```

앞에가 src, 뒤가 dest 이며, `dry-run` 실행 구문이다. `--progress`는 `412 files to consider` 표시. 

# dig `+trace`
```console
$ dig +trace likejazz.com
```

`+trace`는 DNS 질의 과정을 root dns 부터 모두 표시해준다.

# Port Forwarding
```
$ ssh -N -L 7860:10.17.120.XX:7860 airlab-xxx@10.12.54.XX -p 30001
```

`localhost:7860`으로 접속하면 10.12.54.XX:30001의 ssh를 통해 10.17.120.XX:7860으로 연결된다.

## ssh 접속 via proxy

```
$ cat ~/.ssh/config
Host sshuser-vpn
  Hostname 10.17.119.XX
  Port 2222
  User sshuser
  IdentityFile /Users/HXXXXXXX/.ssh/id_rsa_rapids_docker
  ProxyCommand ssh -W 10.17.119.XX:2222 airlab-xxx@10.12.54.XX -p 30001

$ ssh sshuser-vpn
```

10.12.54.XX를 통해서 10.17.119.XX:2222로 ssh 접속한다.

# nload
네트워크 모니터링 `$ sudo apt install nload`

Device는 eth를 택하도록 화살표로 선택하고 `F2` 옵션에서 Unit for traffic numbers: Human Readable (Byte)에서 `TAB`으로 선택, `F5`로 저장

# Reverse Proxy
[A Reverse Proxy](https://github.com/fatedier/frp) to help you expose a local server behind a NAT.

frps:
```
# frps.toml
bindPort = 7000

$ ./frps -c frps.toml
```

frpc:
```
# frpc.toml
serverAddr = "x.x.x.x"
serverPort = 7000

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 2222

$ ./frpc -c frpc.toml
```

```shell
$ ssh -p 2222 xxx@x.x.x.x
```

## nohup without Docker
```
$ http_proxy= nohup ./frpc -c frpc.toml &
```