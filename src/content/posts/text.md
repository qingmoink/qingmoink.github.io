---
title: 安卓启动链：从硅片到桌面的接力
published: 2026-10-20
description: '安卓系统启动全流程：从硬件初始化到可信环境构建'
image: './picture\androidboot.jpg'
tags: ["安卓","系统启动"]
category: '技术研究'
draft: false 
lang: 'zh_CN'
---
# 安卓系统启动全流程：从硬件初始化到可信环境构建
:::note[速览：启动链全景]

<div style="border: 1px solid var(--line-divider); border-radius: 16px; padding: 24px; margin: 24px 0; background-color: var(--card-bg);">

  <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 1.25rem; color: var(--text-90);">一次启动，五段接力</h3>

  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 24px;">
    安卓设备的启动并非单一过程，而是由<strong>硬件初始化、内核加载、服务孵化、可信验证</strong>与<strong>桌面启动</strong>五个阶段串联而成的信任传递链。每一阶段都为下一阶段背书，任何环节被篡改都将导致启动失败。
  </p>

  <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 16px;">
    
<!-- 卡片 1 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">信任起点</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">BootROM</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">芯片固化，不可改写</div>
    </div>

<!-- 卡片 2 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">引导阶段</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">PBL → SBL → ABL</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">三级接力，逐级校验</div>
    </div>

<!-- 卡片 3 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">用户空间首进程</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">init</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">解析 .rc 脚本拉起服务</div>
    </div>

<!-- 卡片 4 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">应用底座</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">Zygote</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">fork+COW 预加载框架</div>
    </div>

<!-- 卡片 5 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">可信框架</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">Verified Boot</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">AVB 2.0 + dm-verity</div>
    </div>

<!-- 卡片 6 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 20px;">
      <div style="color: var(--text-60); font-size: 0.85rem; margin-bottom: 8px;">内存页新规</div>
      <div style="color: var(--text-90); font-size: 1.15rem; font-weight: bold; margin-bottom: 8px;">16KB</div>
      <div style="color: var(--text-75); font-size: 0.9rem;">2025年11月强制适配</div>
    </div>

  </div>
</div>

:::

## 一、硬件初始化与启动链构建：从固件到内核的接力

### 1.1 BootROM信任根锚定与第一阶段引导

安卓系统的启动始于芯片固化的BootROM代码，这段代码由芯片厂商写入，无法被用户修改或刷写，是整个启动链的信任起点。设备上电后，CPU从预定义地址读取BootROM代码并执行，首先完成最基本的硬件初始化，包括设置初始运行环境、初始化堆栈指针和启用内部SRAM，随后探测启动介质，根据硬件引脚状态判断从eMMC、UFS或SPI Flash加载下一阶段程序，最后验证并加载Primary Bootloader（PBL）。这一阶段的验证逻辑采用非对称加密技术，通过内置公钥校验PBL签名，若验证失败则设备无法继续启动，只能通过JTAG或紧急下载模式（EDL）进行救砖操作，体现了硬件信任根（Root of Trust）的不可篡改特性。

:::note[深度解读：从固件到内核]
启动始于芯片固化的**BootROM**代码，由芯片厂商写入，用户无法修改或刷写。设备上电后，CPU从预定义地址读取并执行，完成基础硬件初始化、堆栈指针设置与内部SRAM启用。
:::

:::warning[关键机制]
通过内置公钥校验PBL签名，验证失败则设备无法继续启动，只能通过JTAG或紧急下载模式（EDL）救砖，体现硬件信任根的不可篡改特性
:::

### 1.2 多级引导机制与硬件初始化协同

安卓采用多级引导机制，主要包括PBL、SBL（Secondary Bootloader）和ABL（Android Bootloader）三个阶段，这种设计既保障了启动安全性，又能满足不同阶段的差异化需求。PBL作为芯片ROM中的代码，主要完成基础硬件初始化，加载并验证SBL；SBL负责更复杂的硬件初始化，包括DDR初始化、时钟树配置、显示和USB Gadget初始化，同时建立安全环境，初始化TrustZone；ABL则是Android特定的引导程序，基于LittleKernel或U-Boot实现，负责加载Linux内核并传递启动参数。这三个阶段形成了完整的信任传递链，每个阶段都对下一阶段的镜像进行签名校验，确保启动链的完整性。

:::note[深度解读：三级引导]
<div style="border: 1px solid var(--line-divider); border-radius: 16px; padding: 24px; margin: 24px 0; background-color: var(--card-bg);">

  <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 1.25rem; color: var(--text-90);">三级引导：PBL、SBL 与 ABL</h3>
  
  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 24px;">
    安卓采用多级引导机制，三个阶段各司其职，形成完整的信任传递链，每个阶段都对下一阶段镜像进行签名校验。
  </p>

  <!-- 列表容器 -->
  <div style="display: flex; flex-direction: column; gap: 16px;">

<!-- 卡片 1 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 16px; display: flex; align-items: flex-start; gap: 16px;">
      <div style="background-color: #3b82f6; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">1</div>
      <div>
        <div style="color: var(--text-90); font-size: 1.05rem; font-weight: bold; margin-bottom: 8px;">PBL (Primary Bootloader)</div>
        <div style="color: var(--text-60); font-size: 0.9rem; line-height: 1.5;">芯片 ROM 中的代码，完成基础硬件初始化，加载并验证 SBL</div>
      </div>
    </div>

<!-- 卡片 2 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 16px; display: flex; align-items: flex-start; gap: 16px;">
      <div style="background-color: #3b82f6; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">2</div>
      <div>
        <div style="color: var(--text-90); font-size: 1.05rem; font-weight: bold; margin-bottom: 8px;">SBL (Secondary Bootloader)</div>
        <div style="color: var(--text-60); font-size: 0.9rem; line-height: 1.5;">DDR 初始化、时钟树配置、显示与 USB Gadget 初始化，建立 TrustZone 安全环境</div>
      </div>
    </div>

<!-- 卡片 3 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="border-radius: 12px; padding: 16px; display: flex; align-items: flex-start; gap: 16px;">
      <div style="background-color: #3b82f6; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">3</div>
      <div>
        <div style="color: var(--text-90); font-size: 1.05rem; font-weight: bold; margin-bottom: 8px;">ABL (Android Bootloader)</div>
        <div style="color: var(--text-60); font-size: 0.9rem; line-height: 1.5;">基于 LittleKernel 或 U-Boot 实现，加载 Linux 内核并传递启动参数</div>
      </div>
    </div>

  </div>
</div>
:::

### 1.3 内核加载与初始化环境搭建

当SBL完成硬件初始化后，会加载并执行ABL代码，ABL负责加载Linux内核镜像和初始RAM磁盘（initramfs），并通过ATAG或设备树（DTB）传递启动参数。加载完成后，ABL会关闭缓存和MMU，设置CPU寄存器，最终通过BX指令跳转到内核入口地址，将控制权交给Linux内核。这一过程实现了从硬件引导到操作系统内核的平稳过渡，为后续内核初始化奠定了基础。

:::note[内核跳转：从引导到操作系统]
**SBL**完成硬件初始化后**加载并执行ABL代码**。ABL负责**加载Linux内核镜像与初始RAM磁盘（initramfs）**，并通过**ATAG**或 **设备树（DTB）** 传递启动参数。
:::

:::tip[交接动作]
ABL关闭缓存和MMU，设置CPU寄存器，最终通过BX指令跳转到内核入口地址，将控制权平稳交给Linux内核。
:::

**表1：安卓启动关键时间节点梳理**

| 时间点 | 核心动作 | 依赖条件 |
| :--- | :--- | :--- |
| 设备上电 | BootROM执行 | 芯片固化代码生效 |
| PBL加载 | 验证SBL签名 | 内置公钥可用 |
| SBL初始化 | DDR/时钟/TrustZone初始化 | 硬件引脚配置正确 |
| ABL加载 | 加载内核与initramfs | 设备树参数正确传递 |
| 内核跳转 | 跳转到内核入口 | 缓存/MMU已关闭 |

## 二、内核初始化与系统服务孵化：从裸机到Android桌面

### 2.1 内核初始化与驱动加载

内核启动后，首先建立核心数据结构，初始化进程调度器、内存管理单元（MMU）和中断控制器等子系统。随后，内核会按照设备树或ACPI表的描述，探测并初始化CPU、内存、存储、显示、输入设备等关键硬件驱动。驱动初始化的串行或并行方式对启动速度有显著影响，串行初始化虽然稳定性更好，但耗时较长，而并行初始化虽然能加快启动速度，但也可能带来驱动竞态问题。初始化完成后，内核会挂载一个初始的、内存中的根文件系统（rootfs），其中包含了启动后续阶段所必需的最基本工具，最后执行第一个用户空间进程init，标志着内核初始化阶段的结束。

:::note[深度解读：内核到桌面]
<div style="border: 1px solid var(--line-divider); border-radius: 16px; padding: 24px; margin: 24px 0; background-color: var(--card-bg);">

  <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 1.25rem; color: var(--text-90);">内核初始化与驱动加载</h3>
  
  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 16px;">
    内核启动后建立核心数据结构，初始化<strong>进程调度器、MMU 与中断控制器</strong>。随后按设备树或 ACPI 表探测并初始化 CPU、内存、存储、显示、输入等关键硬件驱动。
  </p>

  <!-- 标签组 1 -->
  <div style="display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 12px;">
    <span class="bg-blue-50 dark:bg-blue-900/30 text-blue-600 dark:text-blue-400 border border-blue-200 dark:border-blue-800" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">串行初始化</span>
    <span class="bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-400 border border-gray-200 dark:border-gray-700" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">稳定性优先</span>
    <span class="bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-400 border border-gray-200 dark:border-gray-700" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">耗时较长</span>
  </div>

  <!-- 标签组 2 -->
  <div style="display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 20px;">
    <span class="bg-blue-50 dark:bg-blue-900/30 text-blue-600 dark:text-blue-400 border border-blue-200 dark:border-blue-800" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">并行初始化</span>
    <span class="bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-400 border border-gray-200 dark:border-gray-700" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">加速启动</span>
    <span class="bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-400 border border-gray-200 dark:border-gray-700" style="padding: 4px 12px; border-radius: 6px; font-size: 0.85rem;">可能引发竞态</span>
  </div>

  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 0;">
    初始化完成后挂载内存中的根文件系统 (rootfs)，最终执行第一个用户空间进程 <strong>init</strong>，标志着内核初始化阶段结束。
  </p>

</div>
:::

### 2.2 init进程与系统服务初始化

init进程是用户空间的第一个进程，负责读取init.rc脚本，启动系统服务。init进程的执行流程包括ueventd_main处理内核uevent事件、SubcontextMain在子上下文中执行命令、SetupSelinux设置SELinux安全策略，以及SecondStageMain执行第二阶段的初始化任务，包括启动关键服务和进程。init进程作为用户空间的总调度官，通过解析.rc脚本拉起servicemanager、vold等基础服务，为后续系统服务孵化奠定基础。

:::tip
<div style="border: 1px solid var(--line-divider); border-radius: 16px; padding: 24px; margin: 24px 0; background-color: var(--card-bg);">

  <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 1.25rem; color: var(--text-90);">init 进程：用户空间总调度</h3>
  
  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 20px;">
    init 是用户空间的第一个进程，负责读取 <strong>init.rc 脚本</strong>启动系统服务，其执行流程包含多个关键步骤。
  </p>

  <div style="display: flex; flex-direction: column; gap: 12px; margin-bottom: 20px;">

<!-- 子卡片 1 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="white" xmlns="http://www.w3.org/2000/svg"><path d="M13 2L3 14H12L11 22L21 10H12L13 2Z" /></svg>
      </div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">ueventd_main</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">处理内核 uevent 事件</div>
      </div>
    </div>

<!-- 子卡片 2 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="white" xmlns="http://www.w3.org/2000/svg"><path d="M13 2L3 14H12L11 22L21 10H12L13 2Z" /></svg>
      </div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">SubcontextMain</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">在子上下文中执行命令</div>
      </div>
    </div>

<!-- 子卡片 3 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="white" xmlns="http://www.w3.org/2000/svg"><path d="M13 2L3 14H12L11 22L21 10H12L13 2Z" /></svg>
      </div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">SetupSelinux</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">设置 SELinux 安全策略</div>
      </div>
    </div>

<!-- 子卡片 4 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="white" xmlns="http://www.w3.org/2000/svg"><path d="M13 2L3 14H12L11 22L21 10H12L13 2Z" /></svg>
      </div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">SecondStageMain</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">执行第二阶段初始化，启动关键服务和进程</div>
      </div>
    </div>

  </div>

  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 0;">
    init 通过解析 .rc 脚本拉起 <strong>servicemanager</strong>、<strong>vold</strong> 等基础服务，为后续系统服务孵化奠定基础。
  </p>

</div>
:::

### 2.3 Zygote进程与应用启动底座

Zygote是安卓系统的核心进程之一，作为提前启动的Java进程底座，通过fork+COW机制预加载Framework类、资源与库，实现公共环境共享。系统通过system_server向Zygote发送Unix Domain Socket请求，由其fork出App进程，快速创建应用并配置权限，大幅降低启动延迟。Zygote是安卓高效启动的核心设计，既保障了效率又维持了安全隔离，是安卓从Linux内核到应用桌面的中间枢纽。

### 2.4 system_server与系统服务启动

system_server进程是安卓系统服务的核心载体，负责启动AMS（Activity Manager Service）、WMS（Window Manager Service）、PMS（Package Management Service）等关键系统服务。这些服务的启动顺序严格按照依赖关系进行，AMS负责应用生命周期管理，WMS负责窗口管理和显示，PMS负责应用安装与卸载，确保系统服务能够有序启动并正常运行。system_server的启动标志着安卓系统服务的全面激活，为后续Launcher进程的启动创造了条件。

### 2.5 Launcher进程与桌面启动

Launcher是安卓系统的桌面应用，负责展示应用图标、处理用户交互并启动应用。当system_server完成系统服务启动后，会启动Launcher进程，进入用户可交互的安卓桌面界面。这一阶段标志着安卓系统启动流程的完成，用户可以开始使用设备的各项功能。

## 三、安全启动与完整性验证：构建可信启动链

### 3.1 Verified Boot信任链的逐级校验

安卓的安全启动机制采用Verified Boot（可信启动）框架，构建从硬件信任根到用户空间的完整信任链。信任链的起点是BootROM，通过内置公钥验证PBL签名；PBL验证通过后加载并执行SBL，SBL通过TrustZone验证ABL签名；ABL验证通过后加载内核，内核通过dm-verity验证system、vendor等分区的完整性；最后，init进程验证selinux策略和关键配置文件的完整性。这种逐级校验机制确保了从硬件到软件的每个环节都经过验证，任何环节的篡改都会导致启动失败。

:::tip
<div style="border: 1px solid var(--line-divider); border-radius: 16px; padding: 24px; margin: 24px 0; background-color: var(--card-bg);">

  <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 1.25rem; color: var(--text-90);">Verified Boot 信任链</h3>
  
  <p style="color: var(--text-75); font-size: 0.95rem; line-height: 1.6; margin-bottom: 20px;">
    安卓安全启动机制采用 <strong>Verified Boot</strong>（可信启动）框架，构建从硬件信任根到用户空间的完整信任链，任何环节篡改都会导致启动失败。
  </p>

  <!-- 列表容器 -->
  <div style="display: flex; flex-direction: column; gap: 12px;">

<!-- 子卡片 1 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">1</div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">BootROM → PBL</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">内置公钥验证 PBL 签名</div>
      </div>
    </div>

<!-- 子卡片 2 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">2</div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">SBL → ABL</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">SBL 通过 TrustZone 验证 ABL 签名</div>
      </div>
    </div>

<!-- 子卡片 3 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">3</div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">内核 → 分区</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">通过 dm-verity 验证 system、vendor 分区完整性</div>
      </div>
    </div>

<!-- 子卡片 4 -->
<div class="bg-gray-50 dark:bg-gray-800/50 border border-gray-100 dark:border-gray-700/50" style="display: flex; align-items: center; border-radius: 12px; padding: 16px; gap: 16px;">
      <div style="background-color: #60a5fa; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 1.1rem; flex-shrink: 0;">4</div>
      <div>
        <div style="color: var(--text-90); font-weight: bold; font-size: 1.05rem; margin-bottom: 4px;">init → 配置</div>
        <div style="color: var(--text-60); font-size: 0.85rem;">验证 selinux 策略与关键配置文件完整性</div>
      </div>
    </div>

  </div>
</div>
:::

## 3.2 AVB 2.0的分区验证与哈希树优化

Android Verified Boot 2.0（AVB 2.0）是Verified Boot的核心实现，采用分层验证机制，每个启动阶段只负责验证下一阶段所需的镜像文件。AVB 2.0的核心组件包括vbmeta分区、libavb库和dm-verity验证机制。vbmeta分区存储了各分区的哈希值和签名，libavb库负责在启动时验证分区完整性，dm-verity机制在内核层验证system、vendor等只读分区的完整性。AVB 2.0采用了稀疏哈希树算法，减少了I/O操作，提升了大分区的验证效率，同时支持动态分区和多密钥体系，增强了系统的灵活性和安全性。

### 3.3 回滚保护与防降级机制

AVB 2.0引入了回滚保护机制，通过硬件支持的密钥轮换和版本计数器，防止设备回滚到低版本固件。当设备启动时，Bootloader会检查vbmeta分区中的版本号，拒绝加载版本号更低的分区，确保设备始终运行最新的安全固件。此外，AVB 2.0还支持基于时间戳的签名验证，结合设备唯一ID生成nonce，确保时间戳不可预测，进一步强化了防降级能力。

### 3.4 DM-verity分区完整性验证

dm-verity是AVB的重要组成部分，用于在内核层验证system、vendor等只读分区的完整性。分区被划分为多个4KB块，每个块对应一个哈希值，存储在hashtree中。启动时内核验证哈希树，运行时按需检查访问的块是否被篡改，任何篡改都会导致验证失败，拒绝读取被篡改的分区。这种机制确保了系统分区的完整性，防止恶意软件植入。

## 四、可信执行环境构建：TEE与安全服务隔离

### 4.1 TrustZone硬件隔离与双世界切换

安卓的可信执行环境依托ARM TrustZone技术，将硬件资源划分为安全世界和普通世界，安全世界运行在TrustZone的Secure World中，普通世界运行在Normal World中。两个世界共享同一物理CPU核心，但在不同时间片内运行，通过NS位（Non-Secure bit）区分当前运行状态，NS=0表示安全世界，NS=1表示普通世界。普通世界与安全世界之间的切换通过SMC（Secure Monitor Call）指令实现，普通世界调用SMC指令触发硬件自动跳转到EL3的异常向量表，安全监控器检查SMC功能ID后切换到安全世界执行敏感操作。

### 4.2 Trusty TEE与安全服务承载

Trusty是谷歌为安卓设计的TEE操作系统，运行在ARM TrustZone的安全世界中，用于承载敏感操作，如密钥管理、生物识别认证、支付密钥保护等。Trusty采用微内核架构，仅提供必要的系统服务，支持多任务但严格限制权限，确保即使安卓系统被攻击，关键安全功能仍受保护。Trusty通过libtrusty和TEE驱动程序与安卓系统通信，应用通过Android Keystore或BiometricPrompt间接使用Trusty服务，实现了安全服务的隔离与便捷调用。

### 4.3 安全存储与敏感数据保护

Trusty的安全存储机制采用加密存储和绑定策略，敏感数据以加密形式落盘，解密密钥由TEE内部派生并与设备唯一标识绑定，确保数据无法跨设备迁移。安全存储还支持防重放攻击机制，通过版本号和消息认证码（MAC）校验，防止攻击者将存储回滚到旧版本。不同TA（Trusted Application）的安全存储相互隔离，TA之间不能互相读取对方的存储文件，只有经过授权的接口调用才能访问。

### 4.4 可信UI与安全交互通道

安卓的可信UI机制为敏感操作提供安全显示通道，防止钓鱼攻击。普通UI可能被恶意应用覆盖，而可信UI通过硬件隔离的显示通道展示认证信息，确保用户看到的真实认证界面不受恶意应用干扰。可信UI与普通UI分离，敏感操作的交互界面只能通过可信UI呈现，普通世界的驱动无法接触到可信UI的数据流，保证了交互过程的安全性。

## 五、启动性能优化与适配方案：从4KB到16KB内存页

### 5.1 16KB内存页的优势与适配要求

安卓系统支持16KB内存页，相比传统的4KB内存页，16KB页能减少TLB条目数量，降低页表遍历次数，提升内存密集型工作的CPU效率，同时加快应用冷启动速度，减少因频繁TLB缺失导致的电池消耗。Google Play规定，自2025年11月1日起，应用必须适配16KB内存页方可发布，包含原生库（.so文件）的应用需重新构建，以确保与16KB内存页对齐。

### 5.2 内存页对齐适配方法

适配16KB内存页的方法包括借助Google Play控制台的App Bundle资源管理器精准定位未对齐的SDK或.so文件，利用Android Studio的APK分析器检查.so文件的对齐情况，以及使用build.gradle的同步警告及时发现问题。修复方法包括使用正确的链接器标志重新构建，针对特定目标添加 `-Wl,-z,max-page-size=16384` 标志，或在Android.mk中添加 `LOCAL_LDFLAGS += -Wl,-z,max-page-size=16384` 标志。

**表2：安卓启动流程关键阶段对比**

| 启动阶段 | 核心功能 | 依赖条件 |
| :--- | :--- | :--- |
| BootROM阶段 | 硬件初始化、加载PBL、验证PBL签名 | 芯片固化代码生效 |
| PBL阶段 | 加载SBL、验证SBL签名 | 内置公钥可用 |
| SBL阶段 | DDR/时钟/TrustZone初始化、加载ABL | 设备树参数正确 |
| ABL阶段 | 加载内核与initramfs、传递启动参数 | 缓存/MMU已关闭 |
| 内核阶段 | 驱动加载、挂载rootfs、执行init | 设备树描述正确 |
| init阶段 | 解析.rc脚本、启动系统服务 | rootfs包含必备工具 |
| Zygote阶段 | 预加载Framework、创建App进程底座 | system_server启动完成 |
| system_server阶段 | 启动AMS/WMS/PMS等服务 | Zygote进程可用 |
| Launcher阶段 | 启动桌面界面、展示应用图标 | system_server完成服务启动 |

---

**参考文献**

[1]. [Android 安全性功能][ref1]  
[2]. [启动时验证][ref2]  
[3]. [Android 启动时验证][ref3]  
[4]. [自动启动][ref4]  
[5]. [Android开机时间全解析：从Kernel到Launcher的完整启动流程][ref5]  
[6]. [Bootrom -＞ bootloader -＞ kernel -＞ init ＞android][ref6]  
[7]. [Mark Wu 的博客][ref7]  
[8]. [Android系统流程 的相关内容][ref8]  
[9]. [系統啟動載入程式總覽][ref9]  
[10]. [標準啟動原因][ref10]  
[11]. [Linux内核][ref11]   
[12]. [Android Verified Boot 2.0 安全启动原理详解][ref12]  
[13]. [第十四板块：Android 硬件抽象与安全加固 第三十三篇：Verified Boot 与 硬件信任链（Trusty TEE）][ref13]  
[14]. [Operating System Settings][ref14]  
[15]. [Android系统开机启动流程详解][ref15]  
[16]. [Android 7系统休眠唤醒（二）开机全链路—BootROM到Launcher][ref16]  
[17]. [Pixel Update Bulletin—September 2026][ref17]  
[18]. [025.鸿蒙素材库 · 系统安全启动链][ref18]  
[19]. [Android Bootloader启动逻辑深度解析 - 指南][ref19]  
[20]. [从开机到fastbootd：Android启动路径完整指南][ref20]  
[21]. [Qualcomm's Chain of Trust - LineageOS][ref21]  
[22]. [Android 11 AVB 2.0 解锁原理：从 vbmeta.img 结构到 fastboot 命令的 3 种关闭校验方法][ref22]  
[23]. [Android系统的安全问题 - Android的启动时验证][ref23]  
[24]. [深入理解Android启动流程：从Bootloader到Zygote的6个关键节点全解析][ref24]  
[25]. [深入解析：Android AVB 2.0 核心机制与技术演进][ref25]  
[26]. [Android AVB学习笔记][ref26]  
[27]. [AVB(Android Verified Boot)中vbmeta结构浅析][ref27]  
[28]. [Android AVB 分析（九）Auxiliary Data 包含了哪些描述符和公钥？][ref28]  
[29]. [深入Android AVB 2.0：解锁安全启动新维度][ref29]  
[30]. [AVB(Android Verified Boot)中vbmeta结构浅析][ref30]  
[31]. [Android AVB 分析（二）AVB 2.0 自述文档(注释提问版)][ref31]  
[32]. [AVB 签名][ref32]  
[33]. [TEE 参考文档][ref33]  
[34]. [鸿蒙TEE可信执行环境高级原理：安全存储/加密运算隔离/可信UI/安全时钟/密钥管理底层机制][ref34]  
[35]. [ARM TrustZone 到底怎么隔离的？从硬件到OP-TEE完整拆解][ref35]  
[36]. [Android Security Bulletin—June 2026][ref36]  
[37]. [Android系统的安全问题 - Android的Trusty TEE][ref37]  
[38]. [Android 的 16KB 内存页：看似 “内存浪费”，实则提升应用性能达 30%][ref38] 
[39]. [CVE-2026-0083][ref39]  
[40]. [Android 15 大变更：支持16K 内存分页，所有native app 必须重编译 ...][ref40]  
[41]. [安卓手机的运行内存和机身存储对手机性能有哪些影响？ - 新浪][ref41]  
[42]. [Android 的 16KB 内存页：看似 “内存浪费”，实则提升应用性能达 30%][ref42]  

[ref1]: https://source.android.google.cn/docs/security/features?authuser=31&hl=zh-tw
[ref2]: https://source.android.google.cn/docs/security/features/verifiedboot?hl=zh-cn
[ref3]: https://source.android.google.cn/docs/security/features/verifiedboot/avb?hl=zh-cn
[ref4]: https://bkso.baidu.com/item/%E8%87%AA%E5%8A%A8%E5%90%AF%E5%8A%A8/56622676?fromModule=lemma_inlink
[ref5]: https://m.blog.csdn.net/hope8/article/details/151710147
[ref6]: https://m.blog.csdn.net/weixin_40557160/article/details/128388892
[ref7]: https://m.blog.csdn.net/wulong756273/
[ref8]: https://www.aliyun.com/sswb/1021192.html
[ref9]: https://source.android.google.cn/docs/core/architecture/bootloader?hl=zh-tw
[ref10]: https://source.android.google.cn/docs/core/architecture/bootloader/boot-reason?authuser=0&hl=zh-tw
[ref11]: https://baike.baidu.com/item/linux%E5%86%85%E6%A0%B8/10142820
[ref12]: https://m.blog.csdn.net/xxzhaoming/article/details/161106810
[ref13]: https://blog.csdn.net/seven_lauseon/article/details/161755586
[ref14]: https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/os-settings.html
[ref15]: https://www.jianshu.com/p/3fd7a553b2e6
[ref16]: https://blog.csdn.net/abc6368765/article/details/163142021
[ref17]: https://source.android.com/docs/security/bulletin/pixel/2026/2026-09-01
[ref18]: https://ost.51cto.com/posts/56090
[ref19]: https://www.cnblogs.com/jzssuanfa/p/19222697
[ref20]: https://m.blog.csdn.net/weixin_42376614/article/details/156163115
[ref21]: https://lineageos.org/engineering/Qualcomm-Firmware/
[ref22]: https://wenku.csdn.net/column/56ziijs9199
[ref23]: https://m.blog.csdn.net/xiaoma_pedro/article/details/146534861
[ref24]: https://wenku.csdn.net/column/6btzo9dou7
[ref25]: https://developer.baidu.com/article/detail.html?id=4064160
[ref26]: https://m.blog.csdn.net/lingyu_0312/article/details/158153867
[ref27]: https://m.blog.csdn.net/2509_93873983/article/details/154218423
[ref28]: https://m.blog.csdn.net/guyongqiangx/article/details/144753748
[ref29]: https://developer.baidu.com/article/detail.html?id=4064179
[ref30]: https://m.blog.csdn.net/anjiyufei/article/details/150499666
[ref31]: https://m.blog.csdn.net/guyongqiangx/article/details/146997926
[ref32]: https://cloud.tencent.com/developer/article/2648900
[ref33]: https://www.arm.com/zh-CN/technologies/trustzone-for-cortex-a/tee-reference-documentation
[ref34]: https://blog.csdn.net/2401_83963238/article/details/163464604
[ref35]: https://blog.csdn.net/qq_40786199/article/details/162768443
[ref36]: https://source.android.com/docs/security/bulletin/2026/2026-06-01
[ref37]: https://blog.csdn.net/xiaoma_pedro/article/details/146534272
[ref38]: https://news.qq.com/rain/a/LNK2025111904353600
[ref39]: https://cve.imfht.com/detail/CVE-2026-0083
[ref40]: https://blog.csdn.net/yan_chenglong/article/details/140050863
[ref41]: https://www.sina.cn/gc/article/nirtrzq5015428.html
[ref42]: https://www.toutiao.com/a7568395566984593954/
```  