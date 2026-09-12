# OMA Bot Desktop

[English](README_EN.md) · 中文

本地多 Agent 桌面端。多对话并发、Bot 工作台、项目轨、可换肤。

**本仓库不开放源代码。** 只通过 [Releases](../../releases) 提供官方 Windows EXE。

[![Release](https://img.shields.io/badge/Distribution-EXE%20only-c24d1d)](../../releases)
[![License](https://img.shields.io/badge/License-Proprietary-5a5348)](LICENSE)
[![Source](https://img.shields.io/badge/Source-Not%20published-8a8174)](#源码政策)

---

## 下载

1. 打开仓库的 **Releases**（右侧「Releases」，或上方导航）。
2. 选最新版本，下载 `OMA-Bot-Setup.exe` 或便携包 `OMA-Bot-Portable.zip`。
3. 对照该版本说明里的 SHA256，校验后再运行。

没有 Release 资产时，说明作者还没挂上安装包——请等下一版，不要在 Issues 里要源码。

首次运行若被 SmartScreen 拦截：这是未签名私有软件的常见提示。以你是否信任发布者为准，不要从第三方网盘下载「破解版」。

---

## 它做什么

把大模型接到本地工作目录上：写材料、改稿、整理纪要、跑重复办公流程。每个对话是独立 Agent，可同时跑；切走再切回来，后台结果还在。

| 能力 | 说明 |
| --- | --- |
| 多对话并发 | 侧栏切换；停止只停当前对话 |
| 项目图标轨 | 多个工作目录常驻，点击切换，右键关闭 |
| Bot 工作台 | 预置办公秘书、会议纪要、文书润色、翻译、计划拆解、调研整理等 |
| 执行模式 | 快速 / 默认 / 北极星；规划强化可选 |
| 皮肤 | 印刷纸、宣纸、青瓷、琥珀书房、雾蓝等，切换不重建窗口 |
| 附件 | 拖拽到输入框；图片会按图像任务处理 |
| 补全 | `/` 选 Skill 或 Agent，`@` 搜项目文件 |
| 导出 | 对话另存 Markdown，可选是否带思考过程 |

这不是云服务控制台。模型、Key、工作目录都在你这台机器上。

---

## 运行要求

- Windows 10 / 11 x64（当前公开包以 Windows 为主）
- 建议内存 ≥ 8 GB
- 能访问你在配置里填写的模型接口（本地 Ollama 或云端 API）
- 杀毒软件可能对未签名 EXE 误报，请加入白名单而不是关实时防护

同目录常见文件（运行后生成，不要发到网上）：

```
oma_config.yaml          模型与 Agent 配置
oma_gui.yaml             窗口偏好、皮肤、默认模式
oma_skins.yaml           可选；没有则用内置皮肤
modellist.yaml           模型下拉名单
.opencode/               项目内 Bot 清单与会话
.oma_attachments/        浏览器拖入的附件落地处
```

API Key 只写在本机配置里。

---

## 十分钟上手

1. 解压或安装后启动 EXE，窗口默认最大化。
2. 确认顶栏配置文件名（例如 `oma_config.yaml`），状态为「就绪」。
3. 左侧项目轨打开你的工作目录。
4. 命令条选模型（「默认」= 配置文件决定）和模式（先用「快速」）。
5. 输入任务，`Ctrl+Enter` 发送。
6. 重复性工作切到 **Bots**：选模板 → 写清任务 → 开始任务 → 在对话里看结果。

### 快捷键

| 按键 | 作用 |
| --- | --- |
| `Ctrl+Enter` | 发送 |
| `Ctrl+滚轮` | 字体大小（会记住） |
| `/` | Skill / Agent 补全 |
| `@` | 项目文件补全 |
| `Esc` | 关掉补全 |

更完整的界面说明见 [docs/USER_GUIDE.md](docs/USER_GUIDE.md)。浏览器打开 [docs/index.html](docs/index.html) 可看带目录的介绍页。

---

## 源码政策

| 问题 | 答案 |
| --- | --- |
| 仓库里有 `.py` 吗？ | 没有，也不会有 |
| 能 PR 功能吗？ | 不接受源码补丁。欢迎缺陷报告（版本号 + 复现步骤 + 日志摘录） |
| 能二次分发 EXE 吗？ | 不可以。请发本仓库 Releases 链接 |
| 为什么 GitHub 还要建仓库？ | 当发布页、校验和与说明的固定地址，不是开源托管 |

完整条款见 [LICENSE](LICENSE)。

---

## 安全与校验

每个 Release 说明应附：

```
OMA-Bot-Portable.zip
SHA256: <发布者填写>
```

Windows 校验：

```powershell
Get-FileHash .\OMA-Bot-Portable.zip -Algorithm SHA256
```

哈希对不上就删掉，不要运行。

---

## 常见问题

**启动闪退**  
先看同目录或 `%TEMP%` 下的日志。缺 VC 运行库、被杀毒隔离、路径含异常权限时最常见。

**连不上模型**  
检查 `oma_config.yaml` 的 `base_url` / `api_key`。本机 Ollama 必须用 `http://127.0.0.1:...`，不要写 `https`。

**SmartScreen / 杀毒报警**  
未签名软件会这样。只从本仓库 Releases 下载，并对照 SHA256。

**要源码 / 要 APK / 要 macOS**  
当前公开通道只有官方 Windows 包。其他平台不承诺。

**这是投资或办公 SaaS 吗？**  
不是。输出由你接入的模型产生，作者不对生成内容负责。
