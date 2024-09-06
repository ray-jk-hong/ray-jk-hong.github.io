---
title: Linux 设备模型
categories: 
- Linux Driver
tags:
- Linux Driver
---

## kobj
1. 代表内核对象，结构体本身不单独使用，而是嵌套在其他高层结构中，用于组织成拓扑关系。
2. sysfs文件系统中一个目录对应一个kobject。
```c
struct kobject {
	const char		*name;                  /* 名字，对应sysfs下的一个目录 */
	struct list_head	entry;               /* kobject中插入的 list_head结构，用于构造双向链表 */
	struct kobject		*parent;            /* 指向当前kobject父对象的指针，体现在sys中就是包含当前kobject对象的目录对象 */
	struct kset		*kset;                    /* 当前kobject对象所属的集合 */
	struct kobj_type	*ktype;            /* 当前kobject对象的类型 */
	struct kernfs_node	*sd;              /* VFS文件系统的目录项，是设备和文件之间的桥梁，sysfs中的符号链接是通过kernfs_node内的联合体实现的 */
	struct kref		kref;                     /* kobject的引用计数，当计数为0时，回调之前注册的release方法释放该对象 */
};
```

## kset
1. kset是包含多个kobject的集合，sysfs目录下有多个子目录，那这个目录就用kset表示
2. sysfs中的设备组织结构很大程度上根据kset组织的，/sys/bus目录就是一个kset对象，在Linux设备模型中，注册设备或驱动时就将kobject添加到对应的kset中。

```c
struct kset {
    struct list_head list; /* 包含在kset内的所有kobject构成一个双向链表 */
    spinlock_t list_lock;
    struct kobject kobj;    /* 归属于该kset的所有的kobject的共有parent */
    const struct kset_uevent_ops *uevent_ops;    /* kset的uevent操作函数集，当kset中的kobject有状态变化时，会回调这个函数集，以便kset添加新的环境变量或过滤某些uevent，如果一个kobject不属于任何kset时，是不允许发送uevent的 */
} __randomize_layout;
```

## ktype
kobj_type指定了删除kobject时要调用的函数。
kobj_type指定了通过sysfs显示或修改有关kobject的信息时要处理的操作，实际是调用show/store函数；
```c
struct kobj_type {
	void (*release)(struct kobject *kobj);
	const struct sysfs_ops *sysfs_ops;
	const struct attribute_group **default_groups;
	const struct kobj_ns_type_operations *(*child_ns_type)(const struct kobject *kobj);
	const void *(*namespace)(const struct kobject *kobj);
	void (*get_ownership)(const struct kobject *kobj, kuid_t *uid, kgid_t *gid);
};
```

## kernfs_node
kernfs_node指定kobject的类型
```c
struct kernfs_node {
    struct kernfs_node	*parent;
	union {
		struct kernfs_elem_dir		dir;
		struct kernfs_elem_symlink	symlink;
		struct kernfs_elem_attr		attr;
	};
    void			*priv;
}
```

## 结构体之间的关系与意义
kset/kobject这些数据结构之间的关系如下：
![alt text](/images/驱动/kobject.webp)

这些关系其实以bus_register函数为例，会比较明朗。bus_register函数如下：
```c
int bus_register(const struct bus_type *bus)
{
    // 创建kset和kobject
    struct subsys_private *priv = kzalloc(sizeof(struct subsys_private), GFP_KERNEL); 
    struct kobject *bus_kobj = &priv->subsys.kobj;
    struct kset *kset = &priv->subsys;

    // 因为我们是在/sys/bus目录下创建，所以当前kobject的父目录是/sys/bus, 所以bus_kobj->kset = bus_kset
 	bus_kobj->kset = bus_kset; 
	bus_kobj->ktype = &bus_ktype;

    kset_register(kset); // 在/sys/bus目录下创建文件夹, 这步过后就能看到/sys/bus目录下新生成了一个文件夹, 并在文件夹下创建uvent文件
    bus_create_file(bus, &bus_attr_uevent); // 创建kobjet->sd

    kset_create_and_add("devices", NULL, bus_kobj); // 创建/sys/bus/xxx/devices目录
    kset_create_and_add("drivers", NULL, bus_kobj); // 创建/sys/bus/xxx/driver目录

    add_probe_files(bus); // 创建/sys/bus/xxx/drivers_autoprobe， /sys/bus/xxx/drivers_probe文件
}
```
仔细看bus_register函数，可以看到kset/kobject这些结构体是什么依赖关系了。

## 接口

device_create
bus_register
driver_register
device_register
device_add
class_create

## 参考
http://www.wowotech.net/device_model/13.html

https://www.cnblogs.com/LoyenWang/p/13334196.html
