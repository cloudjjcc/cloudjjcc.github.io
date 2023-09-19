---
author: cloudjjcc
title: go程序优雅退出
date: 2019-05-08
description: go程序如何进行优雅退出?
tags: [golang,服务端]
categories: [golang,服务端]
---

## 进程在哪些情况下会退出？
正常退出，程序crash，收到kill信号（如CTRL+C会向进程发送SIGINT信号，或者用kill命令发送信号）

## kill PID 与 kill -9 PID 的区别
+ kill PID 发送的是SIGTERM信号，可以被程序捕获
+ kill -9 PID 发送的是SIGKILL信号，不能被捕获，进程会被强制退出

## 优雅退出原理
通过监听可被捕获的退出信号，如：SIGTERM，SIGINT等，在程序退出前完成收尾工作。

## go语言中信号处理
go语言中可通过以下代码监听退出信号：
``` golang
sigCh := make(chan os.Signal, 1)
signal.Notify(sigCh, syscall.SIGTERM, syscall.SIGINT)
<-sigCh
// 执行程序收尾工作。。。
```

## http server 优雅退出
可以通过`Shutdown`方法进行优雅退出
``` golang
// Shutdown gracefully shuts down the server without interrupting any
// active connections. Shutdown works by first closing all open
// listeners, then closing all idle connections, and then waiting
// indefinitely for connections to return to idle and then shut down.
// If the provided context expires before the shutdown is complete,
// Shutdown returns the context's error, otherwise it returns any
// error returned from closing the Server's underlying Listener(s).
//
// When Shutdown is called, Serve, ListenAndServe, and
// ListenAndServeTLS immediately return ErrServerClosed. Make sure the
// program doesn't exit and waits instead for Shutdown to return.
//
// Shutdown does not attempt to close nor wait for hijacked
// connections such as WebSockets. The caller of Shutdown should
// separately notify such long-lived connections of shutdown and wait
// for them to close, if desired. See RegisterOnShutdown for a way to
// register shutdown notification functions.
//
// Once Shutdown has been called on a server, it may not be reused;
// future calls to methods such as Serve will return ErrServerClosed.
func (srv *Server) Shutdown(ctx context.Context) error {...}
```
## gRPC server 优雅退出
相应的gRPC server 也提供了优雅关闭的方法：
``` golang
// GracefulStop stops the gRPC server gracefully. It stops the server from
// accepting new connections and RPCs and blocks until all the pending RPCs are
// finished.
func (s *Server) GracefulStop() {...}
```

## k8s pod 优雅退出
我们先来看Pod的退出流程：
1. Pod 被删除，状态置为 Terminating。
2. kube-proxy 更新转发规则，将 Pod 从 service 的 endpoint 列表中摘除掉，新的流量不再转发到该 Pod。
3. 如果 Pod 配置了 preStop Hook ，将会执行。
4. kubelet 对 Pod 中各个 container 发送 SIGTERM 信号以通知容器进程开始优雅停止。
5. 等待容器进程完全停止，如果在 terminationGracePeriodSeconds 内 (默认 30s) 还未完全停止，就发送 SIGKILL 信号强制杀死进程。
6. 所有容器进程终止，清理 Pod 资源。

要想实现Pod的优雅退出，需要我们的业务程序监听SIGTERM信号，并保证在terminationGracePeriodSeconds内完成退出，否则进程会被强制退出



