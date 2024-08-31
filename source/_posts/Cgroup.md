---
title: Cgroup
categories: 
- Linux Cgroup
tags:
- Linux Cgroup
---

## 相关结构体
```c
struct task_struct {
    struct css_set *cgroups;
    struct list_head cg_list;
}
```

## 在线显示

{% plantuml %}
    AAA->BBB : hello
{% endplantuml %}


## preview用
```plantuml
AAA->BBB : hello
```
