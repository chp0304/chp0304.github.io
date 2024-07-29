---
title: Kafka安装
date: 2024-07-26 14:53:12 +0800
categories: [Kafka]
tags: [kafka]     # TAG names should always be lowercase
typora-root-url: ../../chp0304.github.io/assets/pictures
---

## 安装kafka 

### 安装依赖

- 安装java依赖

  ``` shell
  wget https://builds.openlogic.com/downloadJDK/openlogic-openjdk/8u412-b08/openlogic-openjdk-8u412-b08-linux-x64.tar.gz # 安装jdk8
  ```

- 安装zookeeper依赖

  ``` shell
  wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.4/apache-zookeeper-3.8.4-bin.tar.gz
  ```

### 安装kafka

  ``` shell
  wget https://dlcdn.apache.org/kafka/3.7.1/kafka_2.13-3.7.1.tgz
  ```

### 影响kafka性能的因素

- 磁盘吞吐量-影响生产者（建议选择固态硬盘）
- 磁盘容量
- 内存-影响消费者
- 网络延迟
- CPU
