---
author: cloudjjcc
title: redis 底层数据结构
date: 2019-05-20
description: redis 底层数据结构
tags: [redis]
categories: [NoSQL]
---

## SDS

SDS是Redis中存储string的底层数据结构，定义如下：
``` c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

```
* 对比c语言字符串，SDS实现为动态字符数组，可以高效的进行strlen,append等操作，并且是二进制安全的
* Redis用不同类型的结构体（主要区别在于`len`和`alloc`字段）存储不同大小的string，使用attribute ((packed))来实现紧凑的内存布局，从而节省内存消耗。

## hash


## skiplist
skiplist(跳表)是zset的底层数据结构之一，其结构定义如下：
``` c
/* ZSETs use a specialized version of Skiplists */
// 跳表节点
typedef struct zskiplistNode {
    sds ele;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;
// 跳表
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
// 跳表实现的zset 结构
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

### 跳表结构
跳表实现为多层链表结构，其顶层链表节点比底层少，可以起到索引的作用。
![alt 跳表结构](skiplist.webp)

### 新增数据


### 查找数据


