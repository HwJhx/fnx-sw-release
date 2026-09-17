# fnx_sw AI CLI

> **TODO：一句话说明 fnx_sw 面向谁、解决什么问题。**
> 免源码、零依赖，支持快速部署。

> [!IMPORTANT]
> **当前状态**：`v0.0.1`，首个版本。内置技能集合仍在按 fnx_sw 的职责调整中。

> [!NOTE]
> **系统兼容性**：目前仅发布 **Linux x64** 版本。其他平台的安装包如有需要可另行构建。

---

## 快速安装与启动

在 Linux 终端中运行一键部署脚本：

```bash
# 1. 运行安装脚本（将提示输入商业授权激活码 License Key，完成一码一机绑定）
curl -fsSL https://raw.githubusercontent.com/HwJhx/fnx-sw-release/main/install.sh | bash

# 2. 刷新当前终端的环境变量（按你使用的 Shell 选择）
source ~/.bashrc       # Bash
source ~/.zshrc        # Zsh
source ~/.cshrc        # Csh
source ~/.tcshrc       # Tcsh
# 或直接重新打开一个终端

# 3. 启动
fnx_sw
```

安装后的目录：

```
~/.forenyx/                  机器级，与本机其它 Forenyx 智能体共用
├── forenyx.lic              授权文件
├── .env                     授权码 / client_id / 文档转换的视觉模型配置
└── fnx_sw/
    ├── bin/fnx_sw           命令入口，PATH 指向这里
    ├── libexec/             程序本体
    └── agent/               设置、会话历史、自定义技能
```

> 同一台机器可以并存多个 Forenyx 智能体（`fnx_dv`、`fnx_sw` …），各占一个子目录，
> 互不干扰，共用同一张授权。卸载其中一个不会影响其余。

### 安装指定版本

版本号须带 `v` 前缀，形如 `v0.0.1`：

```bash
# 方式一：管道直接指定
curl -fsSL https://raw.githubusercontent.com/HwJhx/fnx-sw-release/main/install.sh | bash -s -- --release v0.0.1

# 方式二：下载脚本后执行
curl -fsSLO https://raw.githubusercontent.com/HwJhx/fnx-sw-release/main/install.sh
bash install.sh --release v0.0.1
```

> [!TIP]
> * **免交互授权**：用 `--license` 一并传入授权码，例如
>   `bash install.sh --release v0.0.1 --license FNX-XXXX-XXXX-XXXX`
> * **覆盖 / 回退**：本机已激活过的话，再次执行会自动沿用已绑定的 License 并安全覆盖程序
> * **可用版本**：见 [Releases](https://github.com/HwJhx/fnx-sw-release/releases)
> * `--release` 仅适用于在线安装，不能与 `--offline` 同时使用

---

## 首次配置

### 1. 配置大模型

启动 `fnx_sw` 后，在命令行输入 `/login` 开启登录向导：

1. 选择 **`Use an API key`**
2. 选择服务提供商（如 **`SiliconFlow (CN)`**，国内低延迟）
3. 在模型列表中用方向键选择一个高代码能力的推理模型，回车锁定
4. 输入 API Key，并配置上下文与输出上限

> **上下文与输出上限按所选模型的实际规格填**，不要照抄别处的示例。填超过模型上限会直接请求报错。

### 2. 加快响应、减少 Token 消耗（可选）

输入 `/settings` 打开高级设置，把 **`Hide thinking`** 改为 `true` —— 隐藏模型的思考过程，
响应更快、日常交互的 Token 消耗更低。

### 3. 配置 `ic-docx2md` 的视觉模型（可选）

需要把 Word / PDF 文档转成 Markdown 时才用得上。编辑安装时自动生成的全局配置：

```bash
vi ~/.forenyx/.env
```

填入 `OPENAI_API_KEY`，其余按需调整：

```ini
OPENAI_API_KEY=
OPENAI_API_BASE="https://api.siliconflow.cn"
ARK_MODEL_NAME='Qwen/Qwen3.8-27B'
MAX_OUTPUT_TOKENS=32768
TEMPERATURE=0.1
```

> [!IMPORTANT]
> **这与第 1 步的 `/login` 是两套配置，别混。**
> * `/login` 配的是**对话模型**，写在 `~/.forenyx/fnx_sw/agent/auth.json`，每个智能体各自一份
> * 这里配的只管**文档转换里的图片识别**，写在 `~/.forenyx/.env`，本机所有智能体共用
>
> 同一个文件里的 `FORENYX_LICENSE_KEY` 与 `FORENYX_CLIENT_ID` 属于授权，**请勿手工修改**
> —— `CLIENT_ID` 在首次安装时与服务端完成绑定，改了会导致校验失败。

---

## 运维命令

### 升级

```bash
fnx_sw update
```

自动读取本机已绑定的 License 完成静默校验并覆盖升级，不必重跑安装脚本。

### 回退到指定版本

不用先卸载，直接带 `--release` 重跑安装命令即可安全覆盖：

```bash
curl -fsSL https://raw.githubusercontent.com/HwJhx/fnx-sw-release/main/install.sh | bash -s -- --release v0.0.1
```

会沿用本机已绑定的 License 及配置，仅切换程序版本。

### 卸载

```bash
fnx_sw uninstall
```

只移除 `fnx_sw` 自己的目录与 PATH 配置，可以选择是否保留自定义技能和历史会话。

> 本机还装有其它 Forenyx 智能体时，**共用的授权文件会自动保留**；只有在一个智能体都不剩
> 时，才会询问是否一并清除授权。

---

## 常见问题

**启动时提示「无法连接授权服务器，已改用本地授权文件校验」**

说明这次没连上授权服务器，改用本机 `~/.forenyx/forenyx.lic` 完成了校验 —— 程序能正常
启动就说明授权是有效的。若长期连不上，`.lic` 到期后会无法启动，建议排查网络：需要经代理
出网的环境，要在启动前导出 `HTTPS_PROXY` / `HTTP_PROXY` 环境变量（写在 CLI 设置里对授权
校验不生效）。

**安装时提示「检测到旧版目录布局」**

本机装过旧版（程序直接摊在 `~/.forenyx/` 下）。按提示先清理旧版再装；`forenyx.lic` 与
`.env` 不用动，新版仍然使用它们。

**`--version` 显示的名字是什么意思**

```
$ fnx_sw --version
ForeNyx CLI · fnx_sw v0.0.1
```

左边是产品名，右边是当前智能体 —— 同时开多个智能体时用它区分自己在哪一个里面。

---

## 技能（Skills）

> **TODO：待 fnx_sw 的技能集合确定后补充。**
> 当前版本携带的是上游智能体的技能集合，尚未按 fnx_sw 的职责调整。

在 CLI 内输入 `/` 可以查看当前可用的技能列表。
