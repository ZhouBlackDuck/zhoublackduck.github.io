---
date: 2025-04-21
categories:
  - 记录
  - 学习
tags:
  - docker
title: Docker笔记（一）
---

### Docker Swarm网络

![v2-0768daac56f8c22cd46de8b73b316c65](Docker笔记（一）/v2-0768daac56f8c22cd46de8b73b316c65.jpg)

**bridge**网络**docker_gwbridge**，每个节点拥有一个，用于和外界通信

**overlay**网络**ingress**，集群拥有一个，用于集群节点之间通信

内部容器**ingress-sbox**绑定在这两个网络上，用于转发流量

节点**node**通过**docker_gwbridge**与外网连通
