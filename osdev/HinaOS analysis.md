
## HinaOS init

```mermaid
graph TD;
	subgraph kernel
		kernel_main[kernel_main]
		kernel_main --> memory_init
		memory_init --> arch_init
		arch_init --> task_init_percpu
		task_init_percpu --> create_first_task
		create_first_task --> arch_init_percpu
		arch_init_percpu --> idle_task

		idle_task --> idle_task_loop
		idle_task_loop --> task_switch
		task_switch --> arch_idle
		arch_idle --> idle_task_loop
	end
	
```

`memory_init` 初始化 `free` 内存 和 `mmio` 地址。
从 `task_switch` 开始切换任务和调度

## IPC

#### `send_message`

```mermaid
graph TD;
	send_to_self -- "yes" --> return
	send_to_self -- no --> check_dst_is_ready
	check_dst_is_ready -- no --> check_non_block
	check_non_block -- yes --> return
	check_non_block -- no --> block_current
	block_current --> task_switch
	task_switch --> check_task_abort
	check_task_abort -- yes --> return
	check_task_abort -- no --> set_dst_message
	check_dst_is_ready -- yes --> set_dst_message
	set_dst_message --> task_resume
	task_resume --> return
	
```


`check_dst_is_ready` will check dest task Blocked and wait for ipc.

## syscall

```c
#define SYS_IPC 1
#define SYS_NOTIFY 2
#define SYS_SERIAL_WRITE 3
#define SYS_SERIAL_READ 4
#define SYS_TASK_CREATE 5
#define SYS_TASK_DESTROY 6
#define SYS_TASK_EXIT 7
#define SYS_TASK_SELF 8
#define SYS_PM_ALLOC 9
#define SYS_VM_MAP 10
#define SYS_VM_UNMAP 11
#define SYS_IRQ_LISTEN 12
#define SYS_IRQ_UNLISTEN 13
#define SYS_TIME 14
#define SYS_UPTIME 15
#define SYS_HINAVM 16
#define SYS_SHUTDOWN 17
```