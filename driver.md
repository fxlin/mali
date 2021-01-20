# the Linux kernel driver 

*(Much of these info is based through guess work & code reading)*

[TOC]

## Acronyms 

Kbdev -- kbase device. This corresponds to a GPU
Kctx -- a GPU context (?)
GP -- geometry processor
PP -- pixel processor
group -- A render group, i.e. all core sharing the same Mali MMU. see `struct mali_group`
kbase -- the kernel driver instance for midgard 
TLstream -- timeline stream (for trace record)
js - job slot. As exposed by GPUs
Jd - job dispatcher (in the driver)
AS - address space (for GPU)
LPU -- Logical Processing Unit. For timeline display only (?)

## Register access 

### Overview

There are 3 main types of registers (CPU Control, Job Control, MMU Management).
GPU_CTRL: GPU Control
JOB_CTRL: JOB Control
MMU_MGMT: MMU Control

Accordingly, Mali has three types of commands.
GPU_COMAMND: GPU-related (e.g. soft reset, performance counter sample, etc.)
JS_COMMAND: Job-related (e.g. start or stop processing a job chain, etc.)
AS_COMMAND: MMU-related (e.g. MMU lock, broadcast, etc.)

### Code
`Mali_kbas_device_hw_.c`

* kbase_reg_write()
* kbase_reg_read()

**Reg definition**: `Mali_kbase_gpu_regmap.h` 

### Reg map
![regmap](regmap.png)


## Memory management  

#### Memory group manager

On hikey960, this is in mali_kbase_native_mgm.[c|h]
```
* kbase_native_mgm_dev - Native memory group manager device
 * An implementation of the memory group manager interface that is intended for
 * internal use when no platform-specific memory group manager is available.
 * It ignores the specified group ID and delegates to the kernel's physical
 * memory allocation and freeing functions.
```

Note this code is generic (drivers/base), not quite Mali specific. 
In odroid-xu4 v4.15, there's not such a component (??)
Absent in Hikey960 android kernel (4.19)

Seems the "root" of mm allocation. 

`memory_group_manager.h`
struct memory_group_manager_device - Device structure for a memory group
Since it is a "device structure"? Caan there be such hardware?
It defines a bunch of memory_group_manager_ops to be implemented… plat specific?
	
Kbdev->mgm_dev

#### Memory pool vs. memory group manager

One pool may contain pages from a specific memory group

Group id? 

A physical memory group ID. The meaning of this is defined by the systems integrator (???)

A memory group ID to be passed to a platform-specific memory group manager


* Pool -- a frontend of memory allocation
* Memory group manager -- the backend; platform specific. May or may not present (if not, just a sw impl?)


cf: drivers/memory_group_manager/XXX

"An example mem group manager" This seems a sw-only implementation. 

From kernel configuration: "This option enables an example implementation of a memory group manager for **allocation and release of pages for memory pools managed by Mali GPU device drivers.**"

**The example** instance will be init'd during "device probe time". Because it emulates a hw device (??)


There's also "physical mem group manager"? See the devicetree binding `memory_group_manager.txt`

 

 