# Agent Note：上游 DSH 0.1.1-rc.1 适配

状态：已实现

[English](2026-08-21-upstream-dsh-0.1.1-rc.1-adaptation.md) | 中文

## 问题

Studio 从官方 `dsh-v0.1.0-rc.8` 发布版分叉，而官方 Harness 已前进到 `dsh-v0.1.1-rc.1`。如果只摘取发布说明中的可见功能，包版本、凭据事件、Session Projection 合同、Web 启动注入、构建工具与安全行为仍会停留在不兼容的两代实现上。

## 决策

Studio 合并不可变的官方 `dsh-v0.1.1-rc.1` 标签，不追随持续变化的上游分支。合并保持 Studio 的 Desktop 版本和产品表面独立，同时接入官方核心版本、DeepSeek 原生视觉模型、凭据记录及重命名后的引用事件、Session Projection 变更、结构化 Web 启动注入、兼容 Windows 的 pnpm 调用、Bubblewrap PID 隔离和对话体验优化。Studio 自有的权限文案、插件／Preset／应用入口、主题、恢复与视觉路由继续由产品层持有。

Desktop 打包 Host 的部署根现已包含新增授权包。Studio 视觉桥接改为监听 `credentials/reference-updated`，浏览器夹具改走官方结构化 `bootInjections()`。包 `src/` 目录下过时的生成 JavaScript、声明和 source map 已移除，因为它们会在运行时遮蔽合并后的 TypeScript 源码。

## 验证

完整 Host、Client、Web 与 Desktop 构建已在合并后的核心版本上通过。针对 DeepSeek 视觉、本地视觉增强、Session／API Projection、静态资源、Markdown、多行提问、子 Agent 标题、引用编辑和沙箱策略的官方与 Studio 定向测试均通过。完整 GUI 通道通过 4,135 项测试，另有 1 项按设计跳过；更新发布指纹断言后，42 个 Desktop 测试文件全部通过；组装启动和视觉重启快照均在 replay 模式通过。

本机解析到的新 Playwright 版本尚未下载 Chromium Headless Shell，因此完整 Playwright Web 矩阵不作为本次通过证据。该次运行同时暴露并促成了陈旧生成源码的清理，之后才记录这一环境边界。本次没有下载浏览器运行时，也没有构建 Desktop 安装包。

## 考虑过的替代方案

**追随上游 `master`。** 未采用，因为其 head 会在适配过程中继续移动，不能定义可复现的 Desktop 发布基线。

**只摘取发布说明对应提交。** 未采用，因为可见功能依赖完整发布标签中的凭据、Projection、Web 注入、构建和包版本合同。

**用官方目录替换 Studio。** 未采用，因为这会移除 Desktop 控制面，以及插件中心、插件发现、Preset 广场、应用中心、主题、恢复和视觉提供方扩展。

## 后果

Studio 当前把 Harness 核心标识为 `0.1.1-rc.1`，Electron 应用在下一次桌面打包发布前仍保持 `0.1.0-rc.14`。DeepSeek 原生视觉会绕过兼容视觉提供方；纯文本模型仍可使用 Studio 已验证的云端或自托管视觉路由。后续发布安装包前，必须在 macOS 和 Windows 上重新暂存并验证 Desktop Host 闭包。
