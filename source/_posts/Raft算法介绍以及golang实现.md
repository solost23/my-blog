---
title: Raft算法介绍以及golang实现
tags:
    - Algorithm
---

Raft 是 consoul 和 etcd 的的核心算法，是一种一致性算法，Raft 提供了一种在计算系统集群中分布状态机的通用方法，确保集群中的每个节点都同意一系列相同的状态转换，Raft 有许多开源参考实现，具有 Go，C++，Java 和 Scala 中的完整规范实现，一个 Raft 集群包含若干个服务节点，通常是 5 个，这允许整个系统容忍 2 个节点的失败，每个节点处于以下三种状态之一。

- Follower:所有节点都以`Follower`的状态开始。如果没收到`Leader`消息则会变成`Candidate`状态。
- Candidate:会向其它节点“拉选票”，如果得到大部分的票则成为`Leader`，这个过程就叫做`Leader Election`。
- Leader: 所有对系统的修改都会先经过`Leader`。

### <center>Raft一致性算法</center>

`Raft`通过选举出一个`Leader`来简化日志副本的管理，例如，日志项只允许从`Leader`流向`Follower`。基于`Leader`的方法，`Raft`可以分解成三个子问题:

- Leader Election: 原来的`Leader`挂掉后，必须选出一个`New Leader`。
- Log Replication：`Leader`从客户端接收日志，并复制到整个集群中。
- Safety:如果有任意的`Server`将日志项回放到了状态机中，那么其他`Server`只会回放相同的日志项。

### <center>Raft一致性算法动画演示</center>

[Raft一致性算法动画演示](http://thesecretlivesofdata.com/raft/)

动画主要包含三部分:

- 介绍简单版的领导者选举和日志复制的过程。
- 介绍详细版的领导者选举和日志复制的过程。
- 介绍如果遇到网络分区(脑裂)，Raft算法是如何恢复网络一致的。

### <center>Leader Election</center>

Reft 使用一种心跳机制来发出领导人选举，当服务器启动时，节点都是 Follower（跟随者）身份，如果一个跟随者在一段时间里没有收到任何消息，也就是选举超时，然后它就会认为系统中没有可用的领导者，然后开始进行选举以选出新的领导者，要开始一次选举过程，Follower 会给当前 term+1 并且转换成 Candidate 状态，然后它会并行的向集群中的其他服务器节点发送请求投票的 RPCs 来给自己投票。候选人的状态维持直到发生以下任何一个条件时:

- 他自己赢得了这次选举。
- 其他的服务器成为领导者。
- 一段时间后没有任何服务器获胜。

### <center>Log Replication</center>

当选出 Leader 后，它会开始接收客户端请求，每个请求会带有一个指令，可以被回访到状态机中，Leader 会把指令追加成一个 log entry，然后通过 appendEntries RPC 并行地发送给其它的 Server，当该 entry 被多数 Server 复制后，Leader 会把 entry 回放到状态机中，然后把结果返回给客户端。当 Follow 宕机或者运行较慢时，Leader 会无限地重发 AppendEntries 给这些 Follower，直到所有的 Follower 都复制到了该 log entry。raft 的 log replication 要保证如果两个 log entry 有相同的 index 和 term，那么它们存储相同的指令。Leader 在一个指定的 term 和 index 下，只会创建一个 log entry。

### <center>Golang实现Raft选举</center>

[Raft算法单机版](https://github.com/Solost23/Raft)

[Raft算法分布式(RPC)版](https://github.com/Solost23/RaftRPC)