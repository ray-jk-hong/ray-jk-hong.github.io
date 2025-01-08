---
title: Linux-链表
categories: 
- Linux链表
tags:
- Linux链表
---

## llist_xx接口
分使用场景无需锁保护的以NULL结尾的单链表。实现文件为 lib/llist.c 和 include/linux/llist.h，后者包含导出的函数和便利宏。

(1) 不需要加锁的情况：
如果有多个生产者和多个消费者，则可以在生产者中使用 llist_add，在消费者中同时使用 llist_del_all，无需锁定。此外，如果单个消费者多个生产者，单个消费者可以使用llist_del_first，
而多个生产者同时使用 llist_add，无需任何锁定。

(2) 需要加锁的情况：
如果我们有多个消费者，其中一个消费者使用 llist_del_first, 其他消费者使用 llist_del_first 或 llist_del_all，则需要锁。这是因为 llist_del_first 依赖于 list->first->next 不会改变，
但是没有锁保护，如果在删除操作中间发生抢占并且被抢占，  
则无法确定 list->first 与之前导致 llist_del_first 中的 cmpxchg 成功的相同。 例如，当一个消费者正在进行 llist_del_first 操作时，
另一个消费者中的 llist_del_first、llist_add、llist_add（或 llist_del_all、llist_add、llist_add）序列可能会导致违规。


