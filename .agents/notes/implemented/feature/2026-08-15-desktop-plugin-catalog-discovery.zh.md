# Agent Note: Desktop 插件目录发现

Status: implemented

[English](2026-08-15-desktop-plugin-catalog-discovery.md) | 中文

## 问题

Desktop 设置界面已经提供插件配置与 Loader 运行清单，但没有产品归属的发现视图。一个类似市场的页面不能安全接受渲染器传来的包名、URL 或任意 Skill 文件夹，因为目录元数据属于不可信输入，而后续 Feature 还要使用同一确定版本身份完成兼容判断、安装、激活证据与卸载。产品也需要明确 Skills 边界：打包为 DSH Bundle 的 Skill Pack 可以发现，但不能因此创建第二套原始 Skill 安装权威。

## 决策

**目录发现是 Desktop 归属的边界。** `@deepseek-ai/dsh-plugin-center-contracts` 为目录摘要、详情、筛选、兼容性、新鲜度、制品与声明运行身份定义严格的精确字段解码器。Desktop 在固定 npm registry 中搜索 `dsh-plugin`；DeepSeek Harness 文档把它定义为发现关键词。主进程再读取确定版本元数据，并只保留声明 `dsh.bundle` 的包。HTTPS origin、响应与压缩包边界、不可变完整性校验、权威缓存原子替换、离线回退和 IPC 发送方检查都归主进程所有；渲染器不能提交来源 URL、文件系统路径、包命令或可执行文件。

**插件与 Skill Pack 共用一个目录身份。** 目录条目携带 `catalogKind: plugin | skill-pack`、一个确定包版本、Bundle 成员，以及激活检查必须观察的身份。Skill Pack 是普通已校验 DSH Bundle，其声明包含 `skill` 能力，并在发布时可包含 `expectedSkillIds`；独立原始 Skill 文件夹不是市场条目，也没有单独安装路径。

**发现是一级应用页面，现有插件设置保持独立。** `@deepseek-ai/dsh-client-ui-plugin-center` 通过 `sidebar.primary.action` 贡献侧栏动作，并通过 keyed `main.page` 贡献独立页面。`ui-layout` 拥有所选 `primaryPage` 并保持 Conversation 界面挂载，`ui-workspace` 在用户新建或打开会话时关闭该页面。页面按 Codex 参考层级提供「插件/技能」导航、居中搜索与内容、已安装图标条、公开/个人范围、响应式双列无卡片外框发现条目，以及进入同页确定版本详情的面包屑导航。未安装的公开条目直接显示安装动作；已安装目录条目以当前 Profile 投影为准，并通过紧凑三点菜单只显示后台支持的更新、启用、停用或卸载动作。展开的已安装区域继续承载来源、运行状态、兼容性、配置与运行清单详情，现有设置路径仍可独立进入。安装与管理动作只通过已交接桥接合同出现，并复用确定版本预检、确认与持久事务状态。

**实时来源是预构建 npm DSH Bundle 生态。** 搜索卡片来自带 `dsh-plugin` 标签的 npm 确定版本；该标签是社区发现信号，不代表 DeepSeek 背书。打开或安装结果时会下载 tarball，只有 registry 完整性、SHA-256、压缩包边界、包身份、Bundle patch、Loader 条目、可选客户端模块与可选 Skill 声明一致，才会生成安装权威。已校验确定版本会进入缓存，使已安装插件在离线时仍可管理。只有 GitHub 源码的仓库和不含 `dsh.bundle` 的包不会进入一键安装路径。

**浏览器开发使用完整 Desktop UI 组合与显式 fixture 接缝，而不是第二套生产权威。** `pnpm run dev:desktop:web`（兼容别名 `dev:plugin-center`）以 `DSH_DESKTOP=1` 启用与 Desktop 相同的客户端 UI 名册和皮肤，通过 Host 注入的开发标记与确定只读桥接启用同一插件页面，并把 `DSH_HOME` 隔离在项目工件目录。页面根节点提供可检查的开发数据标记和悬停说明，不额外插入破坏参考 UI 的横幅；URL 场景覆盖正常、空目录、陈旧缓存与读取失败。该入口不能启动真实 MCP、Electron IPC、文件系统、包管理器、Host 重启或安装动作，Desktop 始终优先使用 preload 桥接。

## 曾考虑的替代方案

**让 React 客户端直接加载公共 registry。** 不采用：这会把网络策略、缓存归属和来源选择移入不可信渲染器，并让后续变更依赖客户端提供的权威。

**把任意本地 Skill 文件夹视为市场条目。** 不采用：原始 Skill 不具备安全一键生命周期所需的确定版本包、Bundle、兼容性、完整性与运行证据合同。

**用市场替换现有插件设置页。** 不采用：配置卡片和 Loader 清单承载不同的已安装状态与诊断旅程，不属于发现功能。

**在后端合同交接前渲染安装控件。** 不采用：缺少 F002 兼容证据和 F003/F005 事务与恢复链路时，安装入口会宣传产品无法验证的生命周期。

## 验证

合同测试覆盖畸形目录输入、确定身份、npm 制品 origin、Skill Pack 要求、缓存归属、原子替换与陈旧回退。Desktop 测试覆盖固定桥接方法、发送方/origin 拒绝、npm 搜索过滤、确定 tarball 补全、离线权威复用、安装与管理事务以及恢复。实时 smoke 已解析并校验 `dsh-latex-tools@0.1.2`；隔离包管理器 smoke 已安装并移除该公开确定版本。客户端测试继续覆盖一级页面、搜索、详情、确认、进度、已安装管理和浏览器开发旅程。`pnpm run dev:desktop:web` 仍是 UI 开发入口，真实包变更必须使用 Desktop。

## 后果

Desktop 用户可以搜索公开 DSH 生态，并安装、运行、更新、停用、启用或卸载已校验的 npm 确定版本 Bundle，而渲染器不会因此获得来源选择或包管理器权威。兼容、变更、恢复与已安装投影消费同一套严格身份和证据词汇。代价是生态包仍是拥有广泛进程权限的社区代码；校验证明不可变身份与激活声明，不等于官方代码审计。
