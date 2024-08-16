---
title: PowerManage
categories: 
- Linux
tags:
- Linux PowerManage
---

## wakelock接口


## Suspend流程
```c [include/linux/suspend.h]
#define PM_HIBERNATION_PREPARE	0x0001 /* Going to hibernate */
#define PM_POST_HIBERNATION	0x0002 /* Hibernation finished */
#define PM_SUSPEND_PREPARE	0x0003 /* Going to suspend the system */
#define PM_POST_SUSPEND		0x0004 /* Suspend finished */
#define PM_RESTORE_PREPARE	0x0005 /* Going to restore a saved image */
#define PM_POST_RESTORE		0x0006 /* Restore failed */
```
register_pm_notifier接口注册监听上述几个状态。
在进入睡眠之前会发送PM_SUSPEND_PREPARE， 等睡眠结束（进入睡眠失败或者从睡眠中唤醒的时候）会再发送PM_POST_SUSPEND


### Suspend流程参考
https://www.wowotech.net/pm_subsystem/suspend_and_resume.html


