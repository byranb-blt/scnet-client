# 模块名称：SClaw / 智能助手

## 0. 本模块检查过的代码位置

| 代码位置 | 识别内容 | 证据类型 |
|---|---|---|
| `src/config/sclaw-whitelist.js` | SClaw 白名单标识与白名单响应解析 | 权限条件 |
| `src/stores/user.js` | SClaw 白名单加载、可见性、登录后自动初始化、退出登录关闭服务 | 状态与权限 |
| `src/router/permission.js` | 登录后拉取用户信息并检查 SClaw 白名单 | 初始化流程 |
| `src/Layout/index.vue` | 左侧导航 SClaw 入口、内嵌 SClaw 容器、首次登录引导、页面恢复规则、埋点事件接收 | 全局入口 |
| `src/Layout/components/SubMenu.vue` | 退出登录时关闭 SClaw 后端并重置白名单 | 退出流程 |
| `src/composables/useSClawSync.js` | 主题、语言、用户、活动禁用状态同步，SClaw toast 映射到客户端提示 | 宿主联动 |
| `main/preload.ts` | `sclaw` 与 `openclaw` 宿主能力暴露 | 宿主能力边界 |
| `main/sclaw/view-manager.ts` | SClaw WebView 注册、消息转发、右键菜单、埋点透传 | 内嵌窗口 |
| `main/sclaw/sclaw-preload.ts` | SClaw 内部桥接、服务偏好、后端生命周期、登录态跳转 | 内嵌能力 |
| `main/sclaw/state-bridge.ts` | 宿主与 SClaw 共享状态字段 | 状态同步 |
| `main/sclaw/openclaw-integration.ts` | SClaw 服务生命周期、Gateway 状态、内置技能安装、渠道消息转发、CLI 自动安装 | 后台服务 |
| `main/index.js` | SClaw IPC 注册、应用退出时关闭 SClaw 服务 | 客户端生命周期 |
| `submodules/SClaw/src/App.tsx` | SClaw 路由、Setup 向导、内嵌模式设置页 | SClaw 主框架 |
| `submodules/SClaw/src/components/layout/Sidebar.tsx` | SClaw 内部侧边栏、会话分组、新建对话、删除会话、Gateway 状态与启动按钮 | SClaw 内部导航 |
| `submodules/SClaw/src/pages/Chat/*` | 对话、消息、附件、技能选择、思考展示、工具状态、错误条 | 智能对话 |
| `submodules/SClaw/src/pages/Skills/index.tsx` | 技能列表、搜索、启停、配置、更新、社区入口 | 技能管理 |
| `submodules/SClaw/src/pages/Channels/index.tsx`、`submodules/SClaw/src/components/channels/ChannelConfigModal.tsx` | 消息频道列表、配置、扫码、凭证验证、删除 | 消息频道 |
| `submodules/SClaw/src/pages/Cron/index.tsx` | 定时任务列表、创建、编辑、启停、立即运行、删除 | 定时任务 |
| `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` | 内嵌 SClaw 设置、Gateway 状态、自动运行、模型提供商、工作空间、恢复默认配置 | 设置 |
| `submodules/SClaw/src/components/settings/ProvidersSettings.tsx` | 模型提供商账户、默认模型、API Key、OAuth、回退模型 | 模型配置 |
| `submodules/SClaw/src/pages/Models/index.tsx` | 模型管理和 Token 用量历史 | 隐藏路由 |
| `submodules/SClaw/src/pages/Agents/index.tsx` | Agent 列表、创建、删除、设置、频道绑定只读展示 | 隐藏路由 |
| `submodules/SClaw/src/pages/Setup/index.tsx` | 首次设置向导、运行时检查、模型配置、基础技能安装 | 首次配置 |
| `submodules/SClaw/src/i18n/locales/zh/*.json` | 中文文案、提示、空状态、确认文案 | 用户可见文案 |

## 1. 模块定位

SClaw 是客户端内嵌的智能助手工作台，承载自然语言对话、技能调用、消息频道、定时任务、模型配置和服务生命周期管理。它在主客户端中表现为左侧一级导航入口，只有通过白名单校验的用户可见。SClaw 的核心价值是把 SCNet 账号、作业、文件、技能和外部消息渠道接入对话式工作流，让用户通过提问、选择技能、上传文件或配置定时任务完成复杂操作。

该模块面向已登录且具备 SClaw 权限的用户。主要场景包括：在客户端内直接提问、通过技能处理科研/作业/文件/网页任务、管理模型服务、连接微信/飞书/钉钉等消息渠道、创建自动化任务、查看模型 Token 用量、维护 Gateway 服务状态。

## 2. 识别出的页面、入口、组件、弹窗

| 类型 | 名称 | 入口/触发 | 承载功能 | 用户场景 | 代码证据 |
|---|---|---|---|---|---|
| 主导航入口 | SClaw | 登录后左侧一级导航，白名单通过时显示 | 进入 SClaw 内嵌工作台 | 用户打开智能助手 | `src/Layout/index.vue`、`src/stores/user.js` |
| 内嵌页面 | SClaw WebView | 点击 SClaw 导航 | 承载 SClaw 全部页面 | 客户端内使用智能助手 | `src/Layout/index.vue`、`main/sclaw/view-manager.ts` |
| 内部页面 | Chat | SClaw 根路径 `/`、新建对话、切换会话 | 对话、技能选择、附件、流式回复 | 通过自然语言使用 AI | `submodules/SClaw/src/pages/Chat/index.tsx` |
| 内部页面 | Skills | SClaw 侧边栏“技能” | 技能列表、搜索、启停、配置、更新、社区入口 | 扩展或管理 AI 能力 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| 内部页面 | Channels | SClaw 侧边栏“消息频道” | 频道配置、凭证校验、扫码连接、删除 | 将助手接入微信/飞书/钉钉等渠道 | `submodules/SClaw/src/pages/Channels/index.tsx` |
| 内部页面 | Cron | SClaw 侧边栏“定时任务” | 自动化任务创建、启停、立即运行 | 定时触发 AI 任务 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| 内部页面 | Settings | SClaw 侧边栏“设置” | 服务状态、自动运行、模型提供商、工作空间、恢复配置 | 维护 SClaw 运行环境 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| 隐藏路由 | Models | 路由存在，侧边栏未展示 | 模型提供商与 Token 用量历史 | 高级模型用量管理 | `submodules/SClaw/src/App.tsx`、`submodules/SClaw/src/pages/Models/index.tsx` |
| 隐藏路由 | Agents | 路由存在，侧边栏未展示 | Agent 创建、删除、设置、频道归属查看 | 多 Agent 管理 | `submodules/SClaw/src/App.tsx`、`submodules/SClaw/src/pages/Agents/index.tsx` |
| 首次向导 | Setup | SClaw 设置未完成时自动进入 `/setup/*` | 欢迎、环境检查、模型提供商、基础技能安装、完成 | 首次使用 SClaw | `submodules/SClaw/src/App.tsx`、`submodules/SClaw/src/pages/Setup/index.tsx` |
| 弹窗 | 删除会话确认 | 会话列表悬浮删除按钮 | 删除对话会话 | 清理历史对话 | `submodules/SClaw/src/components/layout/Sidebar.tsx` |
| 弹窗 | 技能详情抽屉 | 点击技能卡片 | 技能信息、路径、API Key、环境变量、启停 | 配置单个技能 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| 弹窗 | 频道配置弹窗 | 点击频道卡片 | 选择频道、填写凭证、扫码、查看文档 | 配置消息渠道 | `submodules/SClaw/src/components/channels/ChannelConfigModal.tsx` |
| 弹窗 | 删除频道确认 | 已配置频道悬浮删除按钮 | 删除频道配置 | 移除渠道连接 | `submodules/SClaw/src/pages/Channels/index.tsx` |
| 弹窗 | 创建/编辑定时任务 | 新建任务或点击任务卡片 | 名称、提示词、调度计划、启用状态 | 创建或修改自动化任务 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| 弹窗 | 删除定时任务确认 | 任务卡片删除 | 删除自动化任务 | 清理任务 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| 弹窗 | 恢复默认网关配置确认 | 设置页“恢复默认配置” | 重建网关核心配置并重启 | 修复 SClaw 配置异常 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| 弹窗 | 模型提供商配置 | 设置页“添加/编辑模型提供商” | API Key、OAuth、Base URL、模型 ID、协议、回退 | 配置模型能力 | `submodules/SClaw/src/components/settings/ProvidersSettings.tsx` |
| 弹窗 | 图片预览灯箱 | 点击消息图片 | 查看大图、在文件夹中显示、关闭 | 查看上传或生成图片 | `submodules/SClaw/src/pages/Chat/ChatMessage.tsx` |
| 弹窗 | Agent 创建/设置/删除确认 | Agents 路由 | 创建 Agent、修改名称、查看频道绑定、删除 Agent | 多 Agent 管理 | `submodules/SClaw/src/pages/Agents/index.tsx` |
| 弹窗 | Token 用量内容明细 | Models 路由开发模式入口 | 查看单条用量记录内容 | 调试模型消耗 | `submodules/SClaw/src/pages/Models/index.tsx` |

## 3. 用户入口

| 入口名称 | 入口位置 | 触发方式 | 进入后页面/弹窗 | 备注 |
|---|---|---|---|---|
| SClaw 一级入口 | 主客户端左侧导航 | 点击“SClaw” | SClaw 内嵌工作台 | 受白名单控制 |
| 新建对话 | SClaw 内部侧边栏顶部 | 点击“新建对话” | Chat 页面空会话 | 旧会话有消息时创建新会话 |
| 历史会话 | SClaw 内部侧边栏中部 | 点击会话名称 | 对话历史 | 按今天、昨天、近一周、近两周、近一月、更早分组 |
| 删除会话 | 会话项悬浮删除按钮 | 点击删除图标 | 删除确认弹窗 | 删除后切换到其他会话或默认会话 |
| 技能入口 | SClaw 侧边栏底部 | 点击“技能” | Skills 页面 | 记录页面点击埋点 |
| 消息频道入口 | SClaw 侧边栏底部 | 点击“消息频道” | Channels 页面 | 记录页面点击埋点 |
| 定时任务入口 | SClaw 侧边栏底部 | 点击“定时任务” | Cron 页面 | 记录页面点击埋点 |
| 设置入口 | SClaw 侧边栏底部 | 点击“设置” | Settings 页面 | 内嵌模式进入 SacpSettings |
| 技能选择入口 | Chat 输入框工具栏 | 点击“技能” | 技能选择浮层 | 只展示启用且可用技能 |
| 附件入口 | Chat 输入框工具栏 | 点击附件、拖拽文件、粘贴文件 | 附件预览 | 支持图片与普通文件 |
| 推荐卡片 | Chat 空会话欢迎区 | 点击推荐卡片 | 自动发送预设技能提示词 | Gateway 停止时置灰 |
| SCNet AI Hub | Skills 页面、Chat 技能浮层 | 点击“前往社区安装” | 外部 SCNet AI Hub | 内嵌时带登录态跳转 |
| 模型提供商入口 | Settings 页面 | 点击添加/编辑模型提供商 | 模型提供商配置弹窗 | 支持 API Key 与 OAuth |
| Gateway 启动入口 | SClaw 侧边栏状态区、Settings 页面 | 点击“启动” | Gateway 启动状态 | 服务停止时可手动启动 |
| Gateway 重启入口 | Settings 页面 | 点击“重启” | Gateway 重启 | 运行中才可用 |
| Gateway 停止入口 | Settings 页面 | 点击“停止” | Gateway 停止 | 影响聊天、技能、频道、任务 |
| Setup 向导入口 | SClaw 内部初始化 | 设置未完成时自动跳转 | Setup 页面 | 入口在 SClaw 子项目内部 |
| Models 隐藏入口 | SClaw 路由 | 路由可访问，侧边栏未展示 | Models 页面 | 入口待产品确认 |
| Agents 隐藏入口 | SClaw 路由 | 路由可访问，侧边栏未展示 | Agents 页面 | 入口待产品确认 |

## 4. 功能层级关系

- L1：SClaw / 智能助手
  - L2：主客户端入口与权限
    - L3：白名单校验
    - L3：左侧导航可见性
    - L3：首次登录引导
    - L3：页面层级恢复与隐藏
  - L2：SClaw 服务生命周期
    - L3：自动运行偏好
    - L3：启动/停止/重启 Gateway
    - L3：生命周期状态展示
    - L3：退出登录关闭服务
    - L3：恢复默认网关配置
  - L2：智能对话
    - L3：新建/切换/删除会话
    - L3：消息发送与停止
    - L3：流式回复与等待状态
    - L3：思考过程显示/隐藏
    - L3：工具调用状态展示
    - L3：附件上传与图片预览
    - L3：技能选择与技能标签
    - L3：推荐场景卡片
    - L3：错误条与配额跳转
  - L2：技能管理
    - L3：技能列表与搜索
    - L3：技能可用状态
    - L3：技能启停
    - L3：技能详情配置
    - L3：内置技能更新
    - L3：社区安装入口
  - L2：消息频道
    - L3：可用频道与支持频道
    - L3：频道配置与更新
    - L3：凭证校验
    - L3：二维码连接
    - L3：频道删除
    - L3：频道运行状态
  - L2：定时任务
    - L3：任务列表与统计
    - L3：创建/编辑任务
    - L3：预设 Cron 与自定义 Cron
    - L3：启用/暂停
    - L3：立即运行
    - L3：删除任务
    - L3：上次运行与下次运行展示
  - L2：模型与设置
    - L3：模型提供商账户
    - L3：默认模型
    - L3：API Key / OAuth
    - L3：回退模型与回退 Provider
    - L3：工作空间目录
    - L3：Token 用量历史
  - L2：Agent 管理
    - L3：Agent 列表
    - L3：创建/删除 Agent
    - L3：Agent 设置
    - L3：频道绑定只读展示
  - L2：首次设置向导
    - L3：欢迎与语言选择
    - L3：运行时检查
    - L3：模型提供商配置
    - L3：基础技能安装
    - L3：完成进入 Chat

## 5. 功能点明细表

| 功能编号 | 功能名称 | 层级 | 功能说明 | 用户操作 | 前置条件 | 结果反馈 | 关联模块 | 复杂度 | 是否核心功能 | 代码证据 |
|---|---|---|---|---|---|---|---|---|---|---|
| SCLAW-001 | SClaw 白名单控制 | L3 | 客户端登录后校验用户是否可使用 SClaw，控制导航入口和服务启停 | 登录客户端 | 用户已登录 | 入口显示或隐藏；无权限时关闭后端服务 | 登录、权限 | L4 | 核心 | `src/config/sclaw-whitelist.js`、`src/stores/user.js` |
| SCLAW-002 | SClaw 一级导航入口 | L3 | 在主客户端左侧导航展示 SClaw 入口 | 点击 SClaw | 白名单通过 | 打开 SClaw 内嵌工作台 | 壳层导航 | L3 | 核心 | `src/Layout/index.vue` |
| SCLAW-003 | 内嵌工作台加载 | L3 | 通过 WebView 加载 SClaw 子应用并展示加载动画 | 点击 SClaw | SClaw 配置可用 | 显示 SClaw 工作台或加载态 | 壳层 | L4 | 核心 | `src/Layout/index.vue`、`main/sclaw/view-manager.ts` |
| SCLAW-004 | 页面层级恢复 | L3 | 从用户中心等页面返回时恢复 SClaw 浮层 | 切换主导航、打开用户中心后返回 | SClaw 曾处于激活态 | SClaw 页面恢复显示 | 壳层、个人中心 | L3 | 重要 | `src/Layout/index.vue` |
| SCLAW-005 | 首次登录 SClaw 引导 | L3 | 首次登录时根据白名单决定是否展示含 SClaw 的引导 | 首次登录 | 白名单通过 | 显示新手引导，窗口尺寸受控 | 登录、壳层 | L3 | 重要 | `src/Layout/index.vue` |
| SCLAW-006 | 宿主状态同步 | L3 | 将主题、语言、用户信息和活动禁用状态同步给 SClaw | 打开 SClaw 或状态变化 | SClaw 已加载 | SClaw 内部状态保持一致 | 壳层、账号 | L4 | 重要 | `src/composables/useSClawSync.js`、`main/sclaw/state-bridge.ts` |
| SCLAW-007 | Toast 提示同步 | L2 | SClaw 内部提示映射为客户端消息提示 | SClaw 产生提示 | SClaw 已加载 | 客户端展示成功/错误/警告/信息提示 | 壳层 | L2 | 重要 | `src/composables/useSClawSync.js` |
| SCLAW-008 | SClaw 埋点透传 | L2 | 对话、页面点击、技能安装信息上报到宿主 | 点击页面、发送消息、加载技能 | SClaw 已加载 | 行为数据上报 | 数据埋点 | L3 | 辅助 | `src/Layout/index.vue`、`main/sclaw/view-manager.ts` |
| SCLAW-009 | 自动运行 SClaw | L3 | 用户可控制客户端是否自动拉起 SClaw 服务 | 设置页切换“自动运行SClaw” | SClaw 权限通过 | 设置保存成功或失败提示 | 设置、更新机制 | L4 | 重要 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| SCLAW-010 | Gateway 状态展示 | L3 | 展示 Gateway 运行、启动中、错误、停止等状态 | 查看 SClaw 侧边栏或设置页 | SClaw 已打开 | 状态徽标、端口、按钮可用性变化 | 服务生命周期 | L3 | 核心 | `submodules/SClaw/src/components/layout/Sidebar.tsx`、`submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| SCLAW-011 | Gateway 手动启动 | L3 | 服务停止时用户可手动启动本次会话服务 | 点击“启动” | 服务初始化完成 | 状态进入启动中，成功后运行 | 聊天、技能、频道、任务 | L4 | 核心 | `submodules/SClaw/src/components/layout/Sidebar.tsx`、`submodules/SClaw/src/stores/sacp-service.ts` |
| SCLAW-012 | Gateway 停止与重启 | L3 | 设置页支持停止、重启 SClaw 服务 | 点击停止或重启 | 服务运行或活跃 | 状态更新，相关功能可用性变化 | 聊天、技能、频道、任务 | L4 | 重要 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| SCLAW-013 | 恢复默认网关配置 | L3 | 重建核心配置并重启 Gateway，当前配置自动备份 | 点击恢复默认配置并确认 | SClaw 设置页可访问 | 成功/失败提示，成功后服务重启 | 设置、服务生命周期 | L4 | 辅助 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| SCLAW-014 | 退出登录关闭 SClaw | L3 | 用户退出客户端时关闭 SClaw 后端并重置白名单 | 点击退出登录确认 | 已登录 | 回到登录页，SClaw 服务关闭 | 登录 | L4 | 重要 | `src/Layout/components/SubMenu.vue` |
| SCLAW-015 | 新建对话 | L3 | 在当前已有消息时创建新会话并进入空对话 | 点击“新建对话” | SClaw 已打开 | Chat 进入空会话 | 智能对话 | L3 | 核心 | `submodules/SClaw/src/components/layout/Sidebar.tsx` |
| SCLAW-016 | 会话列表与分组 | L3 | 历史会话按时间分桶展示，可切换会话 | 点击历史会话 | Gateway 运行并加载会话 | 展示对应对话历史 | 智能对话 | L3 | 核心 | `submodules/SClaw/src/components/layout/Sidebar.tsx` |
| SCLAW-017 | 删除会话 | L3 | 删除指定对话会话并更新当前会话 | 点击会话删除并确认 | 会话存在 | 会话从列表移除 | 智能对话 | L3 | 重要 | `submodules/SClaw/src/components/layout/Sidebar.tsx`、`submodules/SClaw/src/stores/chat/session-actions.ts` |
| SCLAW-018 | 发送文本问题 | L3 | 用户输入问题并发送给助手 | 输入文本，回车或点击发送 | Gateway 运行；输入有内容或附件就绪 | 用户消息进入对话，助手开始响应 | 模型、技能 | L4 | 核心 | `submodules/SClaw/src/pages/Chat/ChatInput.tsx` |
| SCLAW-019 | 停止生成 | L3 | 助手生成中可停止当前响应 | 点击停止按钮 | 正在发送或生成 | 生成停止，状态恢复 | 智能对话 | L3 | 重要 | `submodules/SClaw/src/pages/Chat/ChatInput.tsx`、`submodules/SClaw/src/stores/chat/runtime-send-actions.ts` |
| SCLAW-020 | 刷新对话 | L2 | 重新加载当前对话历史 | 点击刷新 | Chat 页面 | 按钮旋转，历史更新 | 智能对话 | L2 | 辅助 | `submodules/SClaw/src/pages/Chat/ChatToolbar.tsx` |
| SCLAW-021 | 思考过程展示 | L2 | 控制助手思考内容显示或隐藏 | 点击脑图标；展开 Thinking 块 | 消息含思考内容 | 展示或隐藏思考内容 | 智能对话 | L2 | 辅助 | `submodules/SClaw/src/pages/Chat/ChatToolbar.tsx`、`submodules/SClaw/src/pages/Chat/ChatMessage.tsx` |
| SCLAW-022 | 工具调用状态 | L3 | 展示工具运行、完成、错误、耗时和摘要 | 助手调用工具 | 对话生成中或历史含工具 | 显示工具状态条和工具卡片 | 技能、Agent | L4 | 核心 | `submodules/SClaw/src/pages/Chat/ChatMessage.tsx` |
| SCLAW-023 | 文件附件发送 | L3 | 支持选择、粘贴、拖拽文件并随消息发送 | 点击附件、粘贴、拖拽 | Gateway 运行；附件就绪 | 附件预览、上传中遮罩、错误遮罩、发送后清空 | 文件、技能 | L4 | 核心 | `submodules/SClaw/src/pages/Chat/ChatInput.tsx` |
| SCLAW-024 | 图片预览 | L3 | 对话中的图片可放大查看，并可在文件夹中显示 | 点击图片 | 消息含图片 | 打开灯箱，可关闭或打开所在目录 | 文件 | L3 | 重要 | `submodules/SClaw/src/pages/Chat/ChatMessage.tsx` |
| SCLAW-025 | 技能选择发送 | L4 | 输入框内选择启用且可用技能，发送时带技能标签和前缀 | 点击技能按钮、搜索、选择技能 | Gateway 运行；技能已启用且可用 | 已选技能显示为标签，发送后清空 | 技能管理 | L4 | 核心 | `submodules/SClaw/src/pages/Chat/ChatInput.tsx` |
| SCLAW-026 | 推荐场景卡片 | L2 | 空会话展示前沿文献、SCNet 管家、安全审查、网页操控等推荐 | 点击推荐卡片 | Gateway 运行 | 自动发送预设技能提示词 | 技能、SCNet | L3 | 辅助 | `submodules/SClaw/src/pages/Chat/index.tsx` |
| SCLAW-027 | 对话错误条 | L3 | 输入框上方展示 Gateway 停止、模型错误、SCNet 配额等错误 | 发送失败或服务停止 | 存在错误状态 | 错误条展示，可关闭；配额错误可跳转用量页 | 模型、费用 | L4 | 重要 | `submodules/SClaw/src/pages/Chat/index.tsx`、`submodules/SClaw/src/pages/Chat/ChatInput.tsx` |
| SCLAW-028 | 技能列表与搜索 | L3 | 展示可用技能，支持搜索名称、描述、ID、作者 | 打开 Skills，输入关键词 | Gateway 运行 | 列表过滤，空状态提示 | 技能 | L3 | 核心 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| SCLAW-029 | 技能可用状态 | L3 | 展示可用、受阻、未知、可更新、缺少依赖、白名单阻止等状态 | 查看技能列表或详情 | 技能数据加载 | 状态徽标和缺失依赖提示 | 权限、环境 | L4 | 核心 | `submodules/SClaw/src/pages/Skills/index.tsx`、`submodules/SClaw/src/utils/skill-status.ts` |
| SCLAW-030 | 技能启停 | L3 | 用户可启用或禁用非核心技能 | 点击技能开关或详情按钮 | 技能非核心 | 成功/失败提示，列表刷新 | Chat 技能选择 | L3 | 核心 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| SCLAW-031 | 技能配置 | L3 | 非核心技能可配置 API Key 和环境变量 | 打开技能详情，填写并保存 | 技能非核心 | 保存成功/失败提示 | Chat、技能 | L4 | 重要 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| SCLAW-032 | 内置技能更新 | L3 | 内置技能有更新时展示更新按钮 | 点击更新 | 技能标记可更新 | 更新中状态与成功/失败提示 | 技能 | L3 | 辅助 | `submodules/SClaw/src/pages/Skills/index.tsx` |
| SCLAW-033 | 技能社区入口 | L2 | 跳转 SCNet AI Hub 安装社区技能 | 点击“前往社区安装” | 已登录客户端 | 外部页面打开，内嵌模式携带登录态 | 外部服务 | L3 | 重要 | `submodules/SClaw/src/lib/open-scnet-aihub-skills.ts` |
| SCLAW-034 | 消息频道列表 | L3 | 展示已配置频道和支持频道 | 打开 Channels | SClaw 已打开 | 可用频道、支持频道、已配置徽标 | 频道、Agent | L3 | 重要 | `submodules/SClaw/src/pages/Channels/index.tsx` |
| SCLAW-035 | 消息频道配置 | L4 | 配置微信、钉钉、飞书、企业微信、QQ Bot 等主要频道 | 点击频道卡片，填写凭证或生成二维码 | Gateway 运行；部分字段必填 | 配置成功、连接中、验证失败、二维码展示 | 频道、Agent | L5 | 核心 | `submodules/SClaw/src/components/channels/ChannelConfigModal.tsx`、`submodules/SClaw/src/types/channel.ts` |
| SCLAW-036 | 二维码连接频道 | L4 | 微信等扫码渠道可生成二维码并监听连接结果 | 点击生成二维码，扫码确认 | Gateway 运行 | 展示二维码；成功/失败提示 | 频道 | L4 | 重要 | `submodules/SClaw/src/components/channels/ChannelConfigModal.tsx` |
| SCLAW-037 | 删除频道 | L3 | 删除已配置消息频道 | 点击删除并确认 | 频道已配置 | 频道从列表移除 | 频道、Agent | L3 | 重要 | `submodules/SClaw/src/pages/Channels/index.tsx` |
| SCLAW-038 | 定时任务统计 | L2 | 展示任务总数、运行中、已暂停、失败数量 | 打开 Cron | 任务数据加载 | 统计卡片展示 | 定时任务 | L2 | 辅助 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-039 | 创建定时任务 | L4 | 创建任务名称、提示词、调度计划和启用状态 | 点击新建任务，填写并保存 | Gateway 运行；名称/消息/调度有效 | 创建成功提示，任务进入列表 | Chat、频道 | L4 | 核心 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-040 | 编辑定时任务 | L3 | 修改任务名称、提示词、调度计划、启用状态 | 点击任务卡片 | 任务存在 | 更新成功提示 | 定时任务 | L3 | 重要 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-041 | 定时任务启停 | L3 | 切换任务启用或暂停 | 点击任务开关 | 任务存在 | 已启用或已暂停提示 | 定时任务 | L3 | 重要 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-042 | 立即运行任务 | L3 | 手动触发定时任务执行 | 点击立即运行 | Gateway 运行；任务存在 | 触发成功或失败提示，刷新运行记录 | Chat、定时任务 | L4 | 重要 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-043 | 删除定时任务 | L3 | 删除指定定时任务 | 点击删除并确认 | 任务存在 | 删除成功提示 | 定时任务 | L3 | 重要 | `submodules/SClaw/src/pages/Cron/index.tsx` |
| SCLAW-044 | 模型提供商管理 | L4 | 添加、编辑、删除、设默认模型提供商账户 | 设置页操作 | Settings 可访问 | 成功/失败提示，默认标记变化 | Chat、技能、Agent | L5 | 核心 | `submodules/SClaw/src/components/settings/ProvidersSettings.tsx` |
| SCLAW-045 | Provider API Key / OAuth | L4 | 支持 API Key、本地模型和 OAuth 登录配置 | 添加或编辑 Provider | Provider 类型支持对应认证 | Key 校验、OAuth 授权码、等待授权、成功/失败提示 | 模型 | L5 | 核心 | `submodules/SClaw/src/components/settings/ProvidersSettings.tsx` |
| SCLAW-046 | 模型回退配置 | L4 | 支持同 Provider 回退模型和跨 Provider 回退 | 编辑 Provider | 已存在多个 Provider 或模型字段 | 保存后用于模型调用兜底 | 模型、Chat | L4 | 重要 | `submodules/SClaw/src/components/settings/ProvidersSettings.tsx` |
| SCLAW-047 | 工作空间目录 | L3 | 设置 SClaw 可读写工作空间目录 | 手动输入或选择目录 | Settings 可访问 | 保存成功或失败提示 | 文件、安全 | L4 | 重要 | `submodules/SClaw/src/pages/Settings/SacpSettings.tsx` |
| SCLAW-048 | Token 用量历史 | L3 | 查看最近 Token 消耗，可按模型或时间分组，筛选 7/30 天/全部 | 访问 Models 路由 | Gateway 运行 | 用量图表、分页、空状态 | 模型、费用 | L4 | 辅助 | `submodules/SClaw/src/pages/Models/index.tsx` |
| SCLAW-049 | Agent 列表 | L3 | 展示 Agent、默认标记、模型、频道归属 | 访问 Agents 路由 | SClaw 可访问 | Agent 列表显示 | 频道、Chat | L4 | 待确认 | `submodules/SClaw/src/pages/Agents/index.tsx` |
| SCLAW-050 | 创建 Agent | L3 | 输入名称创建新 Agent | 点击添加 Agent | Gateway 运行 | 创建成功或失败提示 | Agent、频道 | L4 | 待确认 | `submodules/SClaw/src/pages/Agents/index.tsx` |
| SCLAW-051 | 删除 Agent | L3 | 删除非默认 Agent | 点击删除并确认 | 非默认 Agent | 删除成功或失败提示 | Agent、频道 | L4 | 待确认 | `submodules/SClaw/src/pages/Agents/index.tsx` |
| SCLAW-052 | Agent 设置 | L3 | 修改 Agent 名称，查看模型和频道绑定 | 点击 Agent 设置 | Agent 存在 | 设置弹窗显示，名称可保存 | Agent、频道 | L3 | 待确认 | `submodules/SClaw/src/pages/Agents/index.tsx` |
| SCLAW-053 | Setup 首次设置向导 | L4 | 设置未完成时引导用户完成欢迎、环境检查、模型、安装和完成流程 | 首次进入 SClaw | `setupComplete` 未完成 | 自动进入向导，完成后进入 Chat | 模型、服务、技能 | L5 | 重要 | `submodules/SClaw/src/pages/Setup/index.tsx` |
| SCLAW-054 | Setup 运行时检查 | L3 | 检查 Node.js、OpenClaw 包、Gateway 服务，支持日志查看和重检 | 向导运行时步骤 | 处于 Setup | 成功/失败/检查中状态；日志面板 | 服务、日志 | L4 | 重要 | `submodules/SClaw/src/pages/Setup/index.tsx` |
| SCLAW-055 | Setup 基础技能安装 | L3 | 自动安装 OpenCode、Python 环境、代码辅助、文件工具、终端等基础组件 | 向导安装步骤 | 通过运行时检查 | 安装进度、失败状态、可跳过 | 技能 | L4 | 重要 | `submodules/SClaw/src/pages/Setup/index.tsx` |

## 6. 状态与规则

| 状态对象 | 状态值 | 用户可见表现 | 可执行操作 | 不可执行操作 | 备注 |
|---|---|---|---|---|---|
| SClaw 权限 | 白名单未加载 | SClaw 入口暂未稳定显示 | 等待初始化 | 使用 SClaw | 登录后异步加载 |
| SClaw 权限 | 白名单通过 | 左侧导航显示 SClaw | 进入 SClaw，自动或手动启动服务 | 无 | 代码标识 `OCCLI` |
| SClaw 权限 | 白名单未通过 | SClaw 入口隐藏 | 使用其他客户端功能 | 使用 SClaw | 后端服务被关闭 |
| WebView 加载 | 加载中 | SClaw 吉祥物和加载点 | 等待 | 对话操作 | 主客户端加载层 |
| WebView 加载 | 已加载 | SClaw 页面可操作 | 使用内部功能 | 无 | WebContents 注册后隐藏加载 |
| Gateway | running | 绿色运行状态 | 聊天、技能、频道、任务、刷新 | 启动按钮隐藏或弱化 | 核心可用状态 |
| Gateway | starting/reconnecting | 启动中状态，旋转图标 | 等待、部分设置操作 | 多数业务操作 | 设置页和侧边栏显示 |
| Gateway | stopped | 停止状态，显示启动按钮 | 手动启动服务 | 聊天发送、技能刷新、频道管理、任务管理 | 输入框提示网关断开 |
| Gateway | error | 错误状态 | 重启/恢复配置/查看提示 | 稳定使用业务能力 | 频道、任务、技能均可能显示警告 |
| 对话 | 空会话 | 欢迎区和推荐卡片 | 输入问题、选择推荐卡片 | 查看历史消息 | Gateway 停止时推荐卡片置灰 |
| 对话 | 发送中 | 停止按钮、打字点、工具状态 | 停止生成 | 再次发送、附件选择、技能切换 | 输入区禁用部分操作 |
| 对话 | 等待工具结果 | 活动点状提示 | 等待或停止 | 查看最终回复 | 多轮工具调用场景 |
| 对话 | 错误 | 输入框上方错误条 | 关闭错误、修正配置后重试 | 正常发送受影响 | SCNet 配额错误带跳转 |
| 附件 | staging | 附件缩略图遮罩和旋转 | 等待或移除 | 发送 | 附件就绪前不可发送 |
| 附件 | ready | 附件正常展示 | 发送、移除 | 无 | 图片展示缩略图 |
| 附件 | error | 附件错误遮罩 | 移除后重新选择 | 发送 | 单个附件失败影响发送 |
| 技能 | eligible | “可用”徽标 | 启用、选择、配置 | 无 | 可进入 Chat 技能选择 |
| 技能 | blocked | “受阻”徽标，显示缺少依赖 | 查看详情、补配置 | 正常使用受限 | 缺少 bin/env/config/os 或白名单阻止 |
| 技能 | unknown | “状态未知”徽标 | 查看详情 | 稳定使用待确认 | Gateway 数据不完整 |
| 技能 | core | 锁图标 | 查看详情 | 禁用 | 核心系统技能 |
| 技能 | updateAvailable | “可更新”徽标 | 更新 | 无 | 内置技能更新 |
| 频道 | configured | 已配置徽标 | 编辑、删除 | 作为未配置重新添加 | 可用频道列表显示 |
| 频道 | connected | 已连接 | 使用消息渠道 | 无 | 状态来自运行时快照 |
| 频道 | connecting | 连接中 | 等待、刷新 | 稳定使用 | 频道进程运行但未确认连接 |
| 频道 | disconnected | 未连接 | 重新配置、刷新、删除 | 接收消息 | 配置存在但运行连接失败或停止 |
| 频道 | error | 异常 | 查看错误、重新配置、删除 | 正常消息收发 | 显示 lastError |
| 定时任务 | enabled | 运行中统计增加，显示下次运行 | 暂停、立即运行、编辑、删除 | 无 | 按计划执行 |
| 定时任务 | disabled | 已暂停统计增加 | 启用、编辑、删除、立即运行 | 按计划自动执行 | 开关关闭 |
| 定时任务 | lastRun success | 上次运行展示无错误 | 继续使用 | 无 | 卡片展示上次运行时间 |
| 定时任务 | lastRun failed | 失败统计增加，错误块展示 | 编辑、立即运行、删除 | 无 | 卡片展示失败原因 |
| Provider | 未配置 | 空状态和添加按钮 | 添加 Provider | 稳定聊天 | Chat 依赖默认 Provider |
| Provider | 已配置 | Provider 卡片显示 configured/default | 设默认、编辑、删除 | 无 | API Key 可隐藏存储 |
| OAuth | 请求授权码 | 显示授权码或登录页面 | 打开登录页、复制代码、取消 | 保存完成 | 依赖 Provider 支持 |
| Setup | 未完成 | 自动进入向导 | 按步骤配置或跳过 | 直接进入主 Chat 受控 | `setupComplete` 控制 |
| Setup | 安装中 | 安装步骤进度 | 等待或跳过 | 手动下一步 | 安装完成自动进入下一步 |

## 7. 权限与可见性

| 权限/条件 | 影响的功能 | 用户表现 | 提示文案 | 备注 |
|---|---|---|---|---|
| 登录态有效 | SClaw 白名单拉取、WebView 用户态同步 | 登录后才显示入口 | 无固定文案 | 登录态来自主客户端 |
| SClaw 白名单 | SClaw 一级导航、后端服务、首次引导 | 通过时显示入口；未通过时隐藏 | 无固定文案 | 标识 `OCCLI` |
| SClaw 服务偏好关闭 | 自动启动后端 | 设置页显示自动运行关闭；仍可手动启动 | “关闭后客户端不会自动拉起 SClaw 服务...” | 影响自动初始化 |
| Gateway 停止 | Chat、Skills、Channels、Cron | 输入框禁用；技能/频道/任务显示警告或按钮禁用 | “网关未运行...” | 可在侧边栏或设置页启动 |
| 技能核心属性 | 技能禁用 | 核心技能显示锁图标，禁用开关不可用 | 无单独文案 | `isCore` |
| 技能依赖缺失 | 技能选择和使用 | 技能显示受阻，缺少依赖文本 | “缺少依赖：...” | 包括 bin/env/config/os |
| 技能白名单阻止 | 技能使用 | 技能详情显示原因 | “原因：被工具白名单策略阻止” | 来自 `blockedByAllowlist` |
| 频道必填字段 | 频道保存和连接 | 保存按钮禁用或验证失败 | “验证失败”“配置失败: ...” | 不同频道字段不同 |
| 微信等扫码频道 | 账号 ID 编辑 | 插件管理扫码账号时隐藏账号 ID 编辑 | 无固定文案 | 微信映射为 OpenClaw 微信频道 |
| Agent 默认对象 | Agent 删除与名称编辑 | 默认 Agent 显示默认徽标，删除隐藏，名称只读 | 无固定文案 | 非默认 Agent 可删除 |
| Provider 类型 | API Key / OAuth / 本地模型 | 展示对应认证方式和字段 | “API 密钥”“OAuth 登录”“非必填” | 受 Provider 定义影响 |
| 开发者模式 | Models 用量明细内容、部分模型字段 | 开发模式下显示额外查看内容入口 | 无固定文案 | `devModeUnlocked` |
| Setup 运行时检查 | 向导下一步 | 检查失败时下一步禁用 | “检测到环境问题” | 运行时通过后继续 |

## 8. 弹窗、提示与异常

| 类型 | 触发场景 | 用户看到什么 | 用户可选操作 | 影响 |
|---|---|---|---|---|
| 加载提示 | 打开 SClaw WebView | SClaw 吉祥物和加载点 | 等待 | SClaw 加载完成后进入页面 |
| 状态提示 | Gateway 运行状态变化 | 运行/启动中/错误/停止徽标 | 启动、停止、重启 | 决定功能可用性 |
| 二次确认 | 删除会话 | 确认删除弹窗 | 删除、取消 | 移除会话 |
| 错误提示 | Chat 发送失败 | 输入框上方错误条 | 关闭、修正后重试 | 阻断或影响对话 |
| 引导提示 | SCNet 配额耗尽 | 错误条含跳转链接 | 打开用量说明页面 | 引导用户处理费用/额度 |
| 空状态 | Chat 无消息 | 欢迎标题、推荐卡片 | 输入或点击推荐卡片 | 引导开始对话 |
| 空状态 | 技能为空或搜索无结果 | “暂无可用技能”“未找到技能” | 搜索其他词、前往社区 | 引导安装技能 |
| 权限限制提示 | 技能受阻 | 可用状态徽标、缺少依赖、白名单原因 | 配置依赖或联系产品确认 | 技能不可稳定使用 |
| 维护/服务提示 | Skills/Channels/Cron Gateway 停止 | 黄色警告条 | 去设置重启或启动服务 | 禁用刷新/新建/管理 |
| 二次确认 | 删除频道 | “确定要删除此频道吗？” | 删除、取消 | 移除频道配置 |
| 配置提示 | 频道已配置 | 配置弹窗显示已配置提示 | 更新并重新连接 | 避免重复配置 |
| 二维码提示 | 微信扫码连接 | 二维码、扫码说明、刷新二维码 | 扫码、刷新 | 连接成功后保存频道 |
| 错误提示 | 频道凭证验证失败 | 验证失败或配置失败提示 | 修改凭证重试 | 频道不可用 |
| 二次确认 | 删除定时任务 | “确定要删除此任务吗？” | 删除、取消 | 移除任务 |
| 表单校验 | 创建/编辑定时任务 | 名称、消息、调度缺失提示 | 补充字段 | 阻止保存 |
| 运行失败提示 | 定时任务上次运行失败 | 卡片错误块展示错误原因 | 编辑、立即运行、删除 | 任务保持可管理 |
| 二次确认 | 恢复默认网关配置 | 恢复确认文案，提示备份和重启 | 确认恢复、取消 | 重建核心配置 |
| OAuth 提示 | 模型 Provider OAuth | 授权码、登录页、等待授权、认证失败 | 复制代码、打开登录页、取消、重试 | Provider 配置成功或失败 |
| 空状态 | Token 用量为空 | “还没有 token 消耗历史” | 等待数据产生 | 用量页面无数据 |
| 弹窗 | 图片预览 | 黑色遮罩大图 | 关闭、在文件夹中显示 | 查看图片细节 |
| 日志面板 | Setup 环境检查 | 日志内容、打开日志文件夹 | 关闭、打开文件夹 | 排查运行时问题 |

## 9. 模块联动关系

- SClaw 入口依赖登录模块提供用户信息，并依赖白名单接口决定可见性。
- SClaw 与客户端壳层联动：左侧导航进入，用户中心切换时隐藏或恢复 SClaw 浮层，退出登录关闭 SClaw 服务。
- SClaw 与设置模块联动：设置页管理 Gateway 自动运行、启动、停止、重启、工作空间、恢复默认配置。
- Chat 与技能模块联动：输入框技能选择只展示已启用且可用技能，发送时将技能附着到用户问题。
- Chat 与文件模块联动：支持文件附件、图片预览、打开所在目录；工作空间目录影响 SClaw 可读写范围。
- Chat 与模型模块联动：默认 Provider、API Key、OAuth、模型 ID 和回退策略直接影响回复能力。
- Chat 与费用/资源提示联动：SCNet 配额耗尽时展示用量跳转。
- Skills 与外部 SCNet AI Hub 联动：可跳转社区安装技能，内嵌模式携带客户端登录态。
- Channels 与 Agent 联动：频道账号可归属 Agent，Agents 页面展示绑定关系只读信息。
- Cron 与 Chat/Channels 联动：UI 创建任务默认推送到 SClaw Chat；外部渠道创建的任务由 Gateway 处理。
- Models 与数据埋点联动：Token 用量页记录页面访问、拉取尝试、成功、失败等事件。
- SClaw 与数据埋点模块联动：Chat 发送、页面点击、技能安装列表都会透传到宿主埋点。

## 10. 产品复杂度评估

| 维度 | 评分 1-5 | 说明 |
|---|---:|---|
| 功能数量 | 5 | 覆盖对话、技能、频道、任务、模型、Agent、Setup、服务生命周期 |
| 状态复杂度 | 5 | 白名单、Gateway、WebView、会话、附件、技能、频道、任务、OAuth 等多状态叠加 |
| 权限复杂度 | 5 | 登录态、SClaw 白名单、服务偏好、技能依赖、工具白名单、Provider 类型、开发模式共同影响可见性 |
| 跨模块联动 | 5 | 与登录、壳层、文件、费用、设置、埋点、外部 AI Hub 和消息渠道强联动 |
| 用户理解成本 | 4 | 功能集中在智能助手内，但内部概念较多：Gateway、Provider、Skill、Channel、Agent、Cron |
| 重构风险 | 5 | 入口、服务生命周期、权限、WebView、消息桥接和多页面状态均为主链路依赖 |

## 11. 当前功能完整性判断

| 功能 | 判断 | 说明 |
|---|---|---|
| 原始清单“SClaw 模块” | 已确认 | 已找到一级入口、白名单、Chat、技能、频道、定时任务、模型设置、服务生命周期证据 |
| SClaw 白名单 | 新增发现 | 原始清单仅提到白名单关注点，代码中已确认标识、加载状态和无权限关闭后端 |
| 首次登录 SClaw 引导 | 新增发现 | 主布局会根据 SClaw 白名单决定首次登录引导内容 |
| 内嵌 WebView 与主页面恢复 | 新增发现 | SClaw 以独立层覆盖主内容，并在用户中心、缓存提交窗口等场景有特殊恢复规则 |
| Chat 推荐场景卡片 | 新增发现 | 空会话提供四类推荐任务：文献追踪、SCNet 管家、安全检查、网页操控 |
| Chat 文件附件和图片灯箱 | 新增发现 | 对话输入支持文件，消息支持图片大图预览和打开所在目录 |
| 技能社区入口 | 新增发现 | Skills 页面和输入框技能浮层均可跳转 SCNet AI Hub |
| 技能可用性状态 | 新增发现 | 代码中存在可用、受阻、状态未知、缺依赖、白名单阻止、核心技能、可更新等细分状态 |
| 消息频道 | 已确认 | 支持微信、钉钉、飞书、企业微信、QQ Bot 主渠道；代码中还定义 Signal、iMessage、Matrix、LINE、Teams、Google Chat、Mattermost 等扩展类型 |
| 定时任务 | 已确认 | 支持创建、编辑、启停、立即运行、删除、失败展示、预设和自定义 Cron |
| 模型 Provider | 新增发现 | 设置页承担模型账户、默认模型、API Key、OAuth、回退模型/Provider 管理 |
| Token 用量历史 | 新增发现 | Models 路由存在用量历史和图表，但侧边栏无入口 |
| Agents 管理 | 新增发现 | Agents 路由存在 Agent 创建、删除、设置和频道绑定只读展示，但侧边栏无入口 |
| Setup 向导 | 新增发现 | SClaw 子项目首次进入时可自动进入运行时、模型、基础技能安装向导 |
| 批量启用/禁用技能 | 疑似废弃 | Skills 页面存在被注释的批量按钮代码，当前页面无可见入口 |
| 频道账号绑定编辑 | 待确认 | 文案包含绑定 Agent、默认账号、账号删除等能力，当前 Channels 主页面只确认频道配置与删除，账号绑定入口需继续检查后端或隐藏入口 |
| Models / Agents 入口 | 待确认 | 路由存在，SClaw 侧边栏未展示，产品需要确认是否为隐藏功能、开发入口或待上线功能 |

## 12. 面向重构的产品建议

| 建议 | 说明 |
|---|---|
| 保留白名单作为一级入口门槛 | SClaw 会启动本地服务并访问用户上下文，重构时需要继续把权限判定前置到导航和后端生命周期 |
| 将服务状态设计为全局能力 | Chat、Skills、Channels、Cron 均依赖 Gateway，状态提示、启动入口、错误恢复应保持一致 |
| 明确 SClaw 与主客户端页面层级关系 | 当前 SClaw 是覆盖式 WebView，重构时需要定义与用户中心、提交缓存窗口、独立子窗口的恢复规则 |
| 将 Chat、Skills、Channels、Cron 保持为内部一级子模块 | 这些能力各自复杂度高，合并会提高用户理解成本和重构风险 |
| 将 Provider 设置前置到首次使用路径 | Chat 的首要依赖是可用模型，Setup 和 Settings 的 Provider 配置需要保持可发现 |
| 为 Models 与 Agents 明确产品入口策略 | 当前存在路由和完整页面，但侧边栏无入口；建议产品确认公开、隐藏、管理员可见或移入设置 |
| 统一错误与空状态文案 | Gateway 停止、Provider 缺失、技能受阻、频道失败、定时任务失败会同时出现，建议建立统一异常字典 |
| SClaw 适合接管跨模块操作 | SCNet 管家技能已经指向账户、作业、文件管理，对作业失败分析、文件定位、资源查询等场景可由 AI 引导 |
| 保留本地工作空间边界 | 工作空间目录是安全边界，重构时需要保持用户可理解和可配置 |
| 梳理频道与 Agent 的关系 | Channels、Agents、Cron 均涉及账号和消息路由，建议抽象为“消息接入与分发”能力群 |

## 本模块产品还原结论

- 已确认功能：SClaw 白名单入口、内嵌工作台、服务生命周期、智能对话、会话管理、附件与图片预览、技能管理、消息频道、定时任务、模型提供商、工作空间、恢复默认配置、Setup 向导、埋点透传。
- 新增发现功能：首次登录 SClaw 引导、页面恢复规则、推荐场景卡片、SCNet 配额跳转、技能社区入口、技能依赖/白名单阻止状态、Token 用量历史、Agents 管理隐藏路由。
- 待确认功能：Models 与 Agents 是否公开为正式入口；频道账号绑定、默认账号和 Agent 绑定是否已有可见入口；批量技能启停是否计划恢复。
- 疑似废弃功能：Skills 页面批量启用/禁用按钮被注释，当前无可见操作入口。
- 复杂度最高的功能：SClaw 服务生命周期、Chat 对话与技能/附件/工具状态、消息频道配置、模型 Provider 配置。
- 最适合 AI 接管或辅助的功能：作业失败日志分析、SCNet 账户与资源查询、文件定位与批处理、技能推荐、频道/定时任务配置引导。
- 重构时最需要保留的能力：白名单控制、Gateway 状态一致性、Chat 主链路、Provider 配置、技能可用性判断、频道和定时任务的错误恢复。
- 重构时可以弱化或合并的能力：隐藏 Models/Agents 入口可并入设置或高级入口；Setup 中与设置页重复的 Provider 配置可复用同一产品流程；批量技能启停可在产品确认后移除或恢复。
