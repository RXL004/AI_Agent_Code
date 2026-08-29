# VSCODE_CLOUD CODE —— 2026-08-29 环境搭建全流程记录

> 当日目标：① 本地 Git 连接 GitHub ② 下载 cc-switch ③ 安装 Cloud Code
> 实际落地：三项全部完成，并额外打通 Claude Code + SiliconFlow AI 编程链路，完成 AI_Agent_Code 仓库首次提交。

---

## 一、环境初始检查

| 项目 | 初始状态 |
|---|---|
| Git | ✅ 2.54 已安装 |
| Node | ✅ v22.22.2 |
| VS Code | ✅ 1.134（后自动更新到 1.135） |
| Git 全局用户名/邮箱 | ⚠️ 未配置 |
| SSH 密钥 | ❌ 不存在 |
| Chrome | ✅ 用户级安装正常 |

## 二、任务 ①：本地 Git 连接 GitHub（SSH 免密）

**选定方案：SSH 密钥认证**

1. 生成 Ed25519 密钥（无密码短语）：
   ```bash
   ssh-keygen -t ed25519 -C "btcrx-bi-workstation" -f ~/.ssh/id_ed25519 -N ""
   ```
2. 写入 `~/.ssh/config`：
   ```
   Host github.com
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519
       PreferredAuthentications publickey
   ```
3. 配置 Git 全局身份：
   ```bash
   git config --global user.name "RXL004"
   git config --global user.email "btcrxl@126.com"
   ```
4. 用户在 GitHub 网页添加公钥（Settings → SSH keys）
5. 复验通过：`Hi RXL004! You've successfully authenticated`

**公钥（已绑定 GitHub）：**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOUueND+RRDcvx7ESWcGxtW4DGgXObTIegRJMxtmjlcE btcrx-bi-workstation
```

clone/push 直接用 SSH 地址：
```bash
git clone git@github.com:RXL004/AI_Agent_Code.git
```

## 三、任务 ②：下载安装 cc-switch

- 官方仓库：github.com/farion1231/cc-switch，最新版 **v3.20.1**
- 下载 Windows MSI（12.9 MB）→ 双击安装完成
- 坑点：首次下载遇 SSL 错误（exit 35），加 `--retry 5 --retry-all-errors -C -` 断点续传后成功

## 四、任务 ③：安装 Cloud Code（VS Code 插件）

**关键认知：Google 的 "Cloud Code" 插件已更名为 Gemini Code Assist。**

已装插件（`code --list-extensions` 可查）：

| 插件 | 版本 |
|---|---|
| google.geminicodeassist | 2.98.0 |
| GoogleCloudTools.cloudcode | 2.41.0 |
| Google data cloud tools | 0.9.1 |

**使用方法：** 重启 VS Code → 侧边栏 Gemini ✦ 图标 → Sign in with Google → 状态栏显示 `Gemini: Signed In` 即可用（对话面板 / 行内补全 Tab / 右键选中代码操作）。

## 五、问题：无法登录 Google 账号

**排查过程（Chrome 打不开 Google 的诊断）：**

| 检查项 | 结果 |
|---|---|
| Chrome 安装/运行 | ✅ 正常（headless 实测启动成功） |
| baidu.com / github.com | ✅ HTTP 200 |
| **google.com** | ❌ **连接超时** |
| 系统代理 | ProxyEnable=0，未配置 |

**结论：** 浏览器没坏，是网络环境无法直连 Google（无国际出口）。
**本机无法"修复"**，需公司 IT 提供合法代理出口，或改用其他方案。

## 六、方案二：cc-switch + Claude Code + SiliconFlow（已跑通 ✅）

### 6.1 安装 Claude Code

- `claude.ai/install.ps1` 拿不到脚本 → 改走 npm
- 安装命令（独立目录，不污染系统）：
  ```bash
  npm install -g --prefix C:\Users\btcrx\npm-global @anthropic-ai/claude-code
  ```
- 结果：**Claude Code v2.1.251**，新版为原生二进制 `bin/claude.exe`（无需 Node 运行时）
- 已把 `C:\Users\btcrx\npm-global` 追加进用户 PATH（**新开终端**才能敲 `claude`）

### 6.2 供应商选择

- Anthropic 官方 API（api.anthropic.com）返回 403 —— 大陆 IP 被地理封锁，走不通
- 截图候选供应商中选定 **SiliconFlow（硅基流动）**：国内头部聚合平台、兼容 Anthropic 协议、按量计费

### 6.3 SiliconFlow 注册与配置

1. 注册入口：**https://cloud.siliconflow.cn/**（官网底部的 Subscribe 页面只是邮件订阅，别点错）
2. API Key：https://cloud.siliconflow.cn/account/ak → 新建密钥
3. cc-switch → Claude Code 页签 → 添加供应商：

| 字段 | 填写值 |
|---|---|
| 供应商名称 | SiliconFlow |
| **Base URL** | `https://api.siliconflow.cn`（**不带 /v1、不带尾斜杠**，cc-switch 自动补路径） |
| API Key | `sk-` 开头的密钥 |
| 模型 | Pro/MiniMaxAI/MiniMax-M2.5（可改 SiliconFlow 模型广场里 Claude 系列 ID） |

4. 切换后配置自动写入 `~/.claude/settings.json`（ANTHROPIC_BASE_URL / ANTHROPIC_AUTH_TOKEN / ANTHROPIC_MODEL）

### 6.4 坑点：402 余额不足

首次对话报 `API Error: 402 your account balance is insufficient` → SiliconFlow 充值后实测 `/v1/messages` 返回 **HTTP 200**，全链路打通。

## 七、AI_Agent_Code 仓库初始化

```bash
cd C:\Users\btcrx\AI_Agent_Code
git add .
git commit -m "init: 仓库初始化（README、.gitignore、DAY1 记录）"
git push -u origin main
```

- 首次 commit：`928fc71`，含 README.md、.gitignore、AI_Agent_DAY1.txt
- SSH 免密直推成功，远端：https://github.com/RXL004/AI_Agent_Code

## 八、Claude Code 工作模式说明

| 模式 | 作用 |
|---|---|
| Manual | 每次改动前询问确认（最安全，当前选用） |
| Edit automatically | 直接编辑文件不询问 |
| Plan | 先出操作计划，确认后执行 |
| Auto | 低风险自动执行，高风险才询问 |
| Effort (High) | 回答深度/搜索范围：High 最仔细 |

## 九、遗留事项

- [ ] Gemini Code Assist 待公司网络出口后启用（备选方案）
- [ ] SiliconFlow 默认模型为 MiniMax-M2.5，如需 Claude 系可改模型 ID
- [ ] `AI_Agent_DAY1.txt` 目前为空，可补充 Day1 学习记录

## 十、当日成果速览

| 任务 | 状态 |
|---|---|
| Git ↔ GitHub SSH 免密 | ✅ |
| cc-switch v3.20.1 | ✅ 已装并配好 SiliconFlow |
| Claude Code v2.1.251 | ✅ 可正常 AI 对话/编程 |
| VS Code Gemini Code Assist 插件 | ✅ 已装（待网络出口） |
| AI_Agent_Code 首次提交 | ✅ commit 928fc71 已推送 |
