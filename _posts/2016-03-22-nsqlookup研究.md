---
layout:     post
title:      "nsq研究"
subtitle:   "nsqlookup模块"
date:       2016-03-22
author:     "Lucifd"
header-img: "img/header-post/2016-03-20-01.jpg"
tags:
    - MQ
    - Golang
---

> 版权声明：本文为博主原创文章，未经博主允许不得转载。

# nsqlookup研究

## 功能简介

* 提供客户端统一连接的入口
* 统一操作Topic与channel的关系
* 监控nsqd状态

## 包简介

* nsqlookup
	* client_v1.go (在net.Conn的基础上封装了一个包含远程连接信息的PeerInfo对象)
	* context.go （当前环境上下文，既nsqlook实例）
	* http.go（提供了一些操作的Http API）
	* logger.go（日志输出接口）
	* lookup_protocol_v1.go （与nsqd通信的协议操作）
	* nsqlookupd.go
	* options.go （配置文件操作）
	* registration_db.go （逻辑存储，保存channel、topic之间的关系，每个topic的Producer的关系）
	* tcp.go （对client进行校验、分配处理）
	

## 各文件详解

各个文件解释，由入口文件逐渐向下讲，nsq建议使用二进制包进行安装、使用，而二进制包的运行文件入口在apps包下。

### apps/nsqlookup/nsqlookup.go
此文件是nsqlookup的入口程序，首先设定了nsqloookup的可配置参数提供用户根据自身情况配置nsqlookup[具体配置信息nsq官网有介绍](http://nsq.io/components/nsqlookupd.html)

```
    flagSet.Parse(os.Args[1:])

	if *showVersion {
		fmt.Println(version.String("nsqlookupd"))
		return
	}

	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)

	var cfg map[string]interface{}
	if *config != "" {
		_, err := toml.DecodeFile(*config, &cfg)
		if err != nil {
			log.Fatalf("ERROR: failed to load config file %s - %s", *config, err.Error())
		}
	}

	opts := nsqlookupd.NewNSQLookupdOptions()
	options.Resolve(opts, flagSet, cfg)
	daemon := nsqlookupd.NewNSQLookupd(opts)

	daemon.Main()
	<-signalChan
	daemon.Exit()
```

运行流程

* `--config`指定了一个本地配置文件地址，若存在将其读取放入map中
* 创建配置文件，`nsqlookupd.NewNSQLookupdOptions()`返回的opt存在系统指定的默认值。`options.Resolve(opts, flagSet, cfg)`将终端的配置信息覆盖至opt对象上（即未指定的选项使用系统默认的选项）。
* 创建nsqloopd对象开启监听
* 创建一个chan用于接收退出信号，`<-signalChan`一直阻塞知道接收到退出信号，执行`daemo.Exit()`安全退出

### nsqlookup/nsqlookup.go

```
type NSQLookupd struct {
	sync.RWMutex
	opts         *nsqlookupdOptions // 参数配置对象
	tcpListener  net.Listener
	httpListener net.Listener
	waitGroup    util.WaitGroupWrapper
	DB           *RegistrationDB
}
```

waitGroup中的Wrap方法在每次调用时都会自增，保证了退出程序时所有流程能够正常结束，不会产生状态丢失的问题。

```
func (w *WaitGroupWrapper) Wrap(cb func()) {
	w.Add(1)
	go func() {
		cb()
		w.Done()
	}()
}

func (l *NSQLookupd) Exit() {
	if l.tcpListener != nil {
		l.tcpListener.Close()
	}

	if l.httpListener != nil {
		l.httpListener.Close()
	}
	l.waitGroup.Wait()
}
```

以下两段代码开启两个goroutine分别处理tcp和http

```
l.waitGroup.Wrap(func() {
		protocol.TCPServer(tcpListener, tcpServer, l.opts.Logger)
	})
// 中间省略
l.waitGroup.Wrap(func() {
		http_api.Serve(httpListener, httpServer, l.opts.Logger, "HTTP")
	})
```

### tcp.go

这个类十分简单，每当client连接上后会进行一次4个字节的Magic校验，