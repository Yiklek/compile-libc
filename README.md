# Compile libc/libm GitHub Actions

支持编译打包 libc、libm 等库的 GitHub Action 工作流。

## 支持的库

| 库类型 | 描述 | 包含 |
|--------|------|------|
| musl   | 轻量级 libc，适合嵌入式系统 | libc, libm |
| glibc  | GNU libc，Linux 标准库 | libc, libm, libdl, libpthread 等 |
| newlib | 嵌入式系统 libc，适合裸机开发 | libc, libm |

## glibc 编译依赖

编译 glibc 需要先编译以下依赖库：

| 依赖库 | 版本 | 用途 |
|--------|------|------|
| GMP | 6.3.0 | 任意精度数学库 |
| MPFR | 4.2.1 | 多精度浮点运算 |
| MPC | 1.3.1 | 复数运算 |

## 支持的架构

- x86_64 (AMD64)
- aarch64 (ARM64)
- i686 (x86)
- riscv64 (RISC-V)

## 使用方法

### 手动触发编译

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 选择 **Compile libc/libmater** 工作流
3. 点击 **Run workflow** 按钮
4. 填写参数：
   - **libc_type**: 选择要编译的库类型
   - **version**: 指定版本号
   - **target_arch**: 选择目标架构
   - **target_os**: 目标操作系统
   - **release**: 是否创建 GitHub Release

### 常用版本参考

#### musl
- 1.2.5 (最新稳定版)
- 1.2.4
- 1.2.3

#### glibc
- 2.39
- 2.38
- 2.37

#### newlib
- 4.3.0
- 4.2.0

## 示例

### 编译 musl (包含 libc + libm)

```
libc_type: musl
version: 1.2.5
target_arch: aarch64
```

输出：`musl-1.2.5-aarch64.tar.gz`，包含：
- `libc.a` / `libc.so`
- `libm.a` / `libm.so`

### 编译 glibc (完整库套件 + libm)

```
libc_type: glibc
version: 2.39
target_arch: x86_64
release: true
```

输出：`glibc-2.39-x86_64.tar.gz`，包含：
- `libc.a` / `libc.so`
- `libm.a` / `libm.so`
- `libdl`, `libpthread`, `librt` 等

编译顺序：
1. GMP 6.3.0
2. MPFR 4.2.1 (依赖 GMP)
3. MPC 1.3.1 (依赖 GMP + MPFR)
4. glibc 2.39

### 编译 newlib for riscv64 (包含 libc + libm)

```
libc_type: newlib
version: 4.3.0
target_arch: riscv64
target_os: none
```

输出：`newlib-4.3.0-riscv64.tar.gz`，包含：
- `libc.a`
- `libm.a`
- `libsyscalls.a` 等

## 输出文件

编译完成后会生成：
- **Artifact**: `libctype-version-arch.tar.gz` (保留30天)
- **Release**: 如果启用了 release 选项，会创建对应的 GitHub Release

## 跨平台编译

工作流自动支持交叉编译，无需额外配置目标工具链。

## 库结构示例

### musl 输出结构
```
output/
└── usr/local/
    ├── include/
    │   ├── stdio.h
    │   ├── math.h
    │   └── ...
    └── lib/
        ├── libc.a
        ├── libc.so
        ├── libm.a
        └── libm.so
```

### glibc 输出结构
```
output/
└── usr/
    ├── include/
    │   ├── stdio.h
    │   ├── math.h
    │   ├── pthread.h
    │   └── ...
    └── lib64/ (或 lib/)
        ├── libc.a
        ├── libc.so
        ├── libm.a
        ├── libm.so
        ├── libdl.so
        ├── libpthread.so
        └── ...
```

## 注意事项

1. glibc 编译时间较长（约30-60分钟），请耐心等待
2. 确保版本号正确，否则编译会失败
3. 建议先使用低版本测试，确保环境正常
4. glibc 编译需要额外的磁盘空间（约5-10GB）
5. 所有 libc 实现都会自动编译 libm（数学库）
