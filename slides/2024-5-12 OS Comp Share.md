---
theme: serif
highlightTheme: 
mermaid:
  theme: 'base'
---
### 支持多种架构的 
### `Rust` 硬件抽象层模块—`PolyHAL`

<br />
##### 杨金博 
##### 清华大学 
##### 2024/5/12

---

<grid drag="50 90" drop="0 0">
![[微信图片_20240512164432.jpg]]
</grid>

<grid drag="45 90" drop="52 0">
欢迎加群讨论 `PolyHAL` 和硬件相关设计，如果有同学在移植 `PolyHAL` 的过程中遇到问题，可以在群里询问。
</grid>

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
报告内容
:::

- 什么是 `HAL`
<br />
- `PolyHAL` 介绍
<br />
- `PolyHAL` 运行指南
<br />
- 一些工具的指引

---

### 什么是 `HAL`

---
<!-- slide template="[[tpl-title-content-box]]" -->
::: title
##### **什么是 `HAL`**
:::

硬件抽象层（Hardware Abstraction Layer，HAL）是一种计算机系统中的软件层，它充当了硬件和操作系统之间的接口。HAL 的主要目的是隐藏底层硬件的细节，使得操作系统可以与不同类型和制造商的硬件设备交互，而无需关注具体硬件的细节。这种抽象使得操作系统更容易移植到不同的硬件平台上，并简化了软件开发过程，因为开发人员只需与 HAL 接口交互，而无需了解底层硬件的细节。

---

<!-- slide template="[[tpl-title-content-box]]" -->
::: title
##### **为什么需要 `HAL`**
:::

硬件抽象层(HAL)的存在是为了解决以下问题：

1. 硬件依赖性
<br />
2. 硬件抽象
<br />
3. 系统的可维护性、可移植性
<br />
4. 开发流程


---

### `PolyHAL` 介绍

---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
##### **`PolyHAL` 架构图**
:::


![[Pasted image 20240512124825.png]]


---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
##### **为什么要做 `PolyHAL`**
:::

- 让 `Rust` 编写的 `kernel` 聚焦内核实现
<br />
- 简化内核开发流程
<br />
- 提供一个可以复用的 `HAL` 模块
<br />
- 为 `kernel` 提供更多的架构支持



---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
##### **`PolyHAL` 硬件支持情况**
:::


|     架构      |        开发板         |
|:-------------:|:---------------------:|
|   `riscv64`   | `qemu`, `visionfive2` |
|   `aarch64`   |        `qemu`         |
|   `x86_64`    |        `qemu`         |
| `loongarch64` |   `qemu`, `2k1000`    |


---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
`PolyHAL` 启动调用顺序
:::

![[Pasted image 20240512130231.png]]

---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
`PolyHAL` 用户态调用顺序
:::

![[Pasted image 20240512130122.png]]

---

### `PolyHAL` 简单使用指南

---

<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`PolyHAL` 需要 `kernel` 提供的支持
:::

- `Rust` `alloc` 库的支持
- 页表申请和释放的接口
- 实现 `log` 模块的接口
- 各个架构的链接脚本
	- riscv64
	- aarch64
	- x86_64
	- loongarch64
- [使用 `--cfg` 传递运行的开发板](https://github.com/Byte-OS/polyhal/wiki/%E7%BC%96%E8%AF%91%E9%85%8D%E7%BD%AE)

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`PolyHAL` 可以为 `kernel` 提供的支持
:::

- 内核上下文切换 
	- `KContext`
	- `context_switch`
	- `context_switch_pt`
- 用户态上下文切换 
	- `TrapFrame`
	- `run_user_task`
- 中断处理接口

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`PolyHAL` 可以为 `kernel` 提供的支持
:::

- 封装的地址、页操作
- 封装的页表映射和切换
	- `map_page`
	- `unmap_page`
	- `translate`
- 封装的 `TLB` 操作
	- `flush_vaddr`
	- `flush_all`

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`PolyHAL` 可以为 `kernel` 提供的支持
:::

- 内置的设备树，探测可用内存范围
	- `get_mem_areas`
	- `get_dtb_`
- 完成硬件启动和初始化
- 多核启动
	- `boot_all`
	- `get_cpu_num`
	- `hart_id`

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`PolyHAL` 运行
:::

以 `rcore-tutorial-v3` 为例，我们为 `rcore-tutorial-v3` 移植了 `polyhal`

```shell
# Run on riscv64
make ARCH=riscv64 run
# Run on x86_64
make ARCH=x86_64 run
# Run on aarch64
make ARCH=aarch64 run
# Run on loongarch64
make ARCH=loongarch64 run
```

项目地址: https://github.com/yfblock/rcore-tutorial-v3-with-hal-component

---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
`PolyHAL` 运行截图
:::

![[Pasted image 20240512131752.png]]

---
<!-- slide template="[[tpl-title-chart-box]]" -->

::: title
`PolyHAL` 运行截图
:::

![[Pasted image 20240512131943.png]]
---
<!-- slide template="[[tpl-title-chart-2-box]]" -->

::: title
`PolyHAL` 运行截图
:::

::: content1
![[Pasted image 20240512132517.png]]
:::


![[Pasted image 20240512132609.png]]

---

### 一些工具的指引

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`kern-crates` 组织
:::

- `kern-crates` 助力于内核开发
- `kern-crates` 收集 `Rust` 内核模块, 也欢迎您将模块提交到这个组织下。
- `kern-crates` 协助 `Rust` 内核模块进行测试，协助您完善自己的模块。
- 链接：https://github.com/orgs/kern-crates/repositories

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
`kbuild` 内核编译工具
:::

- `kbuild` 内核编译工具可以协助您快速进行内核模块化开发
- `kbuild` 可以用 `yaml` 和 `toml` 来描述内核编译参数、`cfg`和`env`
- `kbuild` 可以简化多种架构存在时的编译
- 可以直接使用 `cargo install kbuild` 进行安装

---
<!-- slide template="[[tpl-title-content-box]]" -->

::: title
相关链接
:::
1. `PolyHAL`: https://github.com/Byte-os/polyhal
2. `PolyHAL wiki`: https://github.com/Byte-OS/polyhal/wiki
3. `PolyHAL` template: [https://github.com/Byte-OS/polyhal-template](https://github.com/Byte-OS/polyhal-template)
4. `rcore-tutorial-with-hal-component`: [https://github.com/yfblock/rcore-tutorial-v3-with-hal-component](https://github.com/yfblock/rcore-tutorial-v3-with-hal-component). 含有多个分支。
5. `ByteOS`: [https://github.com/Byte-OS/byteos](https://github.com/Byte-OS/byteos)

---

### 感谢各位聆听
