### 2024-6-22

我看了一下 那个 fork 过去自己做 `riscv64` 的分支，[https://github.com/Ivan-Velickovic/microkit/](https://github.com/Ivan-Velickovic/microkit/) 但是那边没有指定可用的 `rust-sel4` 版本，直接使用这个分支会导致缺少符号。

目前判断可能是 microkit 和 rust-sel4 中的 rust-microkit 版本不一致。
看了那个作者自己添加的 riscv 支持，工作量比较大。

且比较紧迫的事情是移植到 aarch64.

### 2024-6-26

sel4 中 install 到 /opt/seL4 中的东西 都是由 cmake 指定安装的，几乎都是 libsel4 中的文件原封不动的搬过去。

```cmake
    # Install kernel.elf to bin/kernel.elf
    install(TARGETS kernel.elf RUNTIME DESTINATION bin)
    # Install all libsel4 headers to libsel4/include
    install(
        DIRECTORY
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/include/"
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/arch_include/${KernelArch}/"
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/sel4_arch_include/${KernelSel4Arch}/"
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/sel4_plat_include/${KernelPlatform}/"
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/mode_include/${KernelWordSize}/"
            "${CMAKE_CURRENT_BINARY_DIR}/libsel4/include/"
            "${CMAKE_CURRENT_BINARY_DIR}/libsel4/arch_include/${KernelArch}/"
            "${CMAKE_CURRENT_BINARY_DIR}/libsel4/sel4_arch_include/${KernelSel4Arch}/"
            # The following directories install the autoconf headers
            "${CMAKE_CURRENT_BINARY_DIR}/gen_config/"
            "${CMAKE_CURRENT_BINARY_DIR}/libsel4/gen_config/"
            "${CMAKE_CURRENT_BINARY_DIR}/libsel4/autoconf/"
        DESTINATION libsel4/include
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.pbf"
        PATTERN "api/syscall.xml"
        PATTERN "api/syscall.xsd"
        PATTERN "gen_config.json"
    )
    # Manually install object API files with non-conflicting names
    install(
        FILES "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/include/interfaces/sel4.xml"
        DESTINATION libsel4/include/interfaces
        RENAME object-api.xml
    )
    install(
        FILES
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/arch_include/${KernelArch}/interfaces/sel4arch.xml"
        DESTINATION libsel4/include/interfaces
        RENAME object-api-arch.xml
    )
    install(
        FILES
            "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/sel4_arch_include/${KernelSel4Arch}/interfaces/sel4arch.xml"
        DESTINATION libsel4/include/interfaces
        RENAME object-api-sel4-arch.xml
    )
    # Install libsel4 sources to libsel4/src
    install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/libsel4/src/" DESTINATION libsel4/src)
    # Install additional support files
    if(DEFINED KernelDTBPath)
        install(FILES ${KernelDTBPath} DESTINATION support)
    endif()
    if(DEFINED platform_yaml)
        install(FILES ${platform_yaml} DESTINATION support)
    endif()
    if(DEFINED platform_json)
        install(FILES ${platform_json} DESTINATION support)
    endif()

```

sel4-kernel-loader

```rust
let kernel_entry = unsafe { mem::transmute::<usize, KernelEntry>(payload_info.kernel_image.virt_entry) };

let ui_p_reg_start = payload_info.user_image.phys_addr_range.start;
let ui_p_reg_end = payload_info.user_image.phys_addr_range.end;
let pv_offset = 0_usize.wrapping_sub(payload_info.user_image.phys_to_virt_offset) as isize;
let v_entry = payload_info.user_image.virt_entry;

let (dtb_addr_p, dtb_size) = match &payload_info.fdt_phys_addr_range {
	Some(region) => (region.start, region.len()),
	None => (0, 0),
};

let hart_id = per_core.hart_id;

switch_page_tables();

sel4_cfg_if! {
	if #[sel4_cfg(MAX_NUM_NODES = "1")] {
		(kernel_entry)(
			ui_p_reg_start,
			ui_p_reg_end,
			pv_offset,
			v_entry,
			dtb_addr_p,
			dtb_size,
		)
	} else {
		(kernel_entry)(
			ui_p_reg_start,
			ui_p_reg_end,
			pv_offset,
			v_entry,
			dtb_addr_p,
			dtb_size,
			hart_id,
			core_id,
		)
	}
}
```

sel4 `_start` in head.S
