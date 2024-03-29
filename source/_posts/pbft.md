---
title: pbft
date: 2021-02-08 11:48:55
tags: consensus
category: 区块链
---
# PBFT 共识算法

## 简介

实用拜占庭容错（Practical Byzantine Fault Tolerance，PBFT）算法是一种共识协议，它允许分布式系统在存在拜占庭错误的情况下达成对一系列有序交易的共识。在拜占庭错误中，系统的某个组件会恶意地行为并向其他组件发送任意消息。PBFT 在对系统节点行为做出一定假设的情况下，保证了安全性和活性。

## PBFT 算法流程

PBFT 算法包含以下四个主要阶段：

1. **客户端请求阶段**：客户端向所有备份节点发送请求。
2. **预备阶段**：每个备份节点收到客户端请求后，在与其他备份节点的交互中达成对请求的预备共识。
3. **提交阶段**：备份节点将已经预备的请求提交给其他备份节点，以达成提交共识。
4. **响应阶段**：备份节点将响应发送给客户端。

## PBFT 算法流程图

![PBFT Algorithm Flowchart](img/pbft.png)

## 总结

PBFT 算法是一种典型的拜占庭容错算法，它在分布式系统中的应用十分广泛。虽然 PBFT 算法需要在每个节点之间进行频繁的通信，但由于其保证了系统的安全性和活性，所以在一些对安全性和可靠性要求较高的场景中，PBFT 算法仍然是一种非常实用的共识算法。
