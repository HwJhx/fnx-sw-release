# fnx_sw AI CLI

> **芯片嵌入式软件生成智能体：从芯片架构描述与各 IP 的寄存器描述、规格书出发，生成寄存器头文件、裸机驱动、HAL 与 BSP，交叉编译后在模拟器上验证。**
> 免源码、零依赖，支持快速部署。

> [!IMPORTANT]
> **当前版本**：`v0.0.12`。

> [!NOTE]
> **系统兼容性**：目前仅发布 **Linux x64** 版本。其他平台的安装包如有需要可另行构建。

---

## 安装

> [!IMPORTANT]
> **已不支持在线安装。** 原来的 `curl … install.sh | bash` 现在只会提示「已不支持在线安装，请向管理员索取离线安装包」。
> 请向管理员索取离线安装包，解开后按包内 `README.txt` 操作。

有两种装法，都在离线包目录里执行：

```bash
# A. 装到指定目录，所有用户共用（管理员先建好 /tools/forenyx，由 root 或某个账号执行一次）
./install-release.sh --prefix /tools/forenyx

# B. 只给自己用，装到 ~/.forenyx
./install-release.sh
```

装到指定目录后：

```
/tools/forenyx/.forenyx/        所有用户共用，只放程序
├── forenyx-hostid.sh           申请授权时运行它
└── fnx_sw/
    ├── bin/fnx_sw              命令入口
    ├── libexec/                程序本体、加密的内置技能
    └── tools/                  fd、rg

~/.forenyx/                     每个用户自己的，首次运行 fnx_sw 时自动建好
├── .env                        设备标识、视觉模型配置、VENV_PYTHON
├── forenyx.lic                 授权文件（license import 导入后才有）
└── fnx_sw/agent/               设置、会话历史、自定义技能
```

* root 安装时会写入 `/etc/profile.d/forenyx.sh` 与 `forenyx.csh`，用户重新登录后直接输入 `fnx_sw`；普通账号安装时只改自己的 shell 配置文件，并打印这两个文件的内容供 root 放入。
* 每个用户首次运行 `fnx_sw` 时按提示申请授权：运行 `/tools/forenyx/.forenyx/forenyx-hostid.sh`，把输出发给管理员换取授权文件，再执行 `fnx_sw license import <授权文件>`。
* 配置目录可以用环境变量 `FORENYX_CODING_AGENT_DIR` 另行指定（例如每个 tmux 窗口一份配置），授权仍用 `~/.forenyx` 里的。

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

---

## 运维命令

### 升级

不支持 `fnx_sw update`（执行时只会说明升级方法）。向管理员索取新版离线包，按原来的装法重新执行安装命令；装到指定目录的写同一个 `--prefix`，由安装账号执行，升级前请通知用户退出。各用户 `~/.forenyx` 里的授权与配置都会保留。

### 卸载

```bash
fnx_sw uninstall
```

* 装到指定目录时，只有安装账号（或 root）能卸载，其他用户执行会提示「你没有删除权限，请联系安装账号」。卸载不动任何用户自己的 `~/.forenyx`。
* 以前在线安装的机器，也可以用本仓库的卸载脚本：

```bash
curl -fsSL https://raw.githubusercontent.com/HwJhx/fnx-sw-release/main/uninstall.sh | bash
```

---

## 常见问题

**启动时提示「无法连接授权服务器，已改用本地授权文件校验」**

说明这次没连上授权服务器，改用安装目录下的 `forenyx.lic` 完成了校验 —— 程序能正常
启动就说明授权是有效的。若长期连不上，`.lic` 到期后会无法启动，建议排查网络：需要经代理
出网的环境，要在启动前导出 `HTTPS_PROXY` / `HTTP_PROXY` 环境变量（写在 CLI 设置里对授权
校验不生效）。

**安装时提示「检测到旧版目录布局」**

本机装过旧版（程序直接摊在 `~/.forenyx/` 下）。按提示先清理旧版再装；`forenyx.lic` 与
`.env` 不用动，新版仍然使用它们。

**`--version` 显示的名字是什么意思**

```
$ fnx_sw --version
ForeNyx CLI · fnx_sw v0.0.12
```

左边是产品名，右边是当前智能体 —— 同时开多个智能体时用它区分自己在哪一个里面。

---

## 技能（Skills）

fnx_sw 带一个生成流程命令 `/sw-graph`（内置命令，直接输入，不要写成 `/skill:sw-graph`）和 6 个技能：5 个由流程在需要时自动调用，也可以单独调用重做某一步；`sw-hal-doc` 单独使用。工作目录与输入由平台变量给出（见下一节），命令里不写路径。

| 命令 / 技能 | 用途 | 怎么用 |
| --- | --- | --- |
| **`/sw-graph`**（内置命令） | 整条生成流水线：清点输入、生成、编译、仿真、汇总 | `/sw-graph [--only …] [--restart] [--status]` |
| **`sw-driver-gen`** | 为一个驱动库写 LL 驱动 | 流程自动调用；单独重做：`/skill:sw-driver-gen <驱动库>` |
| **`sw-hal-gen`** | 为一个有 spec 的驱动库写 HAL | 流程自动调用；单独重做：`/skill:sw-hal-gen <驱动库>` |
| **`sw-bsp-gen`** | 为脚本模板没覆盖的 CPU 核写 BSP | 流程自动调用；`/skill:sw-bsp-gen` |
| **`sw-fix-compile`** | 修交叉编译错误 | 流程自动调用；`/skill:sw-fix-compile` |
| **`sw-diagnose-sim`** | 诊断仿真失败 | 流程自动调用；`/skill:sw-diagnose-sim` |
| **`sw-hal-doc`** | 根据 IP 规格书与寄存器描述生成 HAL 说明文档 | `/skill:sw-hal-doc <驱动库>` |

`<驱动库>` 是驱动库 id，与生成的 `lib/<id>/`、`hal/<id>/` 同名，如 `uart`。`/sw-graph` 的参数：`--only a,b` 只处理列出的驱动库；`--restart` 把上次的状态整体备份后从头开始；`--status` 只看进度。再次执行 `/sw-graph` 只处理变了的部分，人改过的代码不覆盖。

---

## 平台环境变量

* **一颗芯片一个工作目录**：平台先建好 `FNX_SW_DIR`，再在它下面启动 fnx_sw。生成的代码与运行状态都在这里。
* 值都是绝对路径；每个命令 / 技能开始时检查用到的变量，缺失或不合法就列出全部问题后停止。
* 只在启动时读取；工具变量或 `toolchain.json` 改了，重新启动 fnx_sw 并用 `/sw-graph --restart`。同一个工作目录同时只能跑一个 `/sw-graph`。
* 工具只认这些变量，不在 `PATH` 里找；模拟器都不设时只生成、编译，报告写明未仿真。

| 变量 | 必需/选用 | 作用 | 格式要求 |
|---|---|---|---|
| `FNX_SW_DIR` | 必需 | 芯片级固件工程目录；fnx_sw 在这里启动，生成的代码与运行状态都在这里 | 必须已存在且可写；等于 fnx_sw 的启动目录 |
| `FNX_SOC_ARCH_FILE` | 必需 | 芯片架构描述 `soc_arch.json`（平台 SpecAgent 产出）：CPU、内存区、设备基址、中断、各 IP 的输入位置 | 可读文件；`schema` 为 `forenyx.soc_arch/*`；设备的 `ipxact`、`component_json`、`spec` 须为绝对路径，同一驱动库的设备这三个字段须相同 |
| `FNX_CC` | 必需 | 交叉编译器 gcc 的绝对路径，如 `/opt/riscv/bin/riscv64-unknown-elf-gcc` | 可执行文件，文件名以 `gcc` 结尾（交叉前缀从文件名取） |
| `FNX_OBJCOPY` | 选用 | objcopy 的绝对路径 | 不设时取 `FNX_CC` 同目录、同前缀的 `objcopy`，它须存在且可执行 |
| `FNX_SIZE` | 选用 | size 的绝对路径 | 同上，`size` |
| `FNX_SPIKE` | 选用 | 模拟器 spike 的绝对路径（RISC-V；寄存器写检查只用 spike） | 设置了须为可执行文件 |
| `FNX_QEMU` | 选用 | 模拟器 qemu 的绝对路径（`qemu-system-riscv32` / `-riscv64` / `-arm`，与芯片对应的那一个） | 设置了须为可执行文件 |

模拟器都不设时：只生成代码、编译，不仿真、不做寄存器写检查，报告结论最高为 PARTIAL 并写明「未仿真」。

工具只认这些变量，不在 `PATH` 或常见安装目录里找。

设置脚本（模板）：

```csh
#!/bin/csh -f
# fnx_sw 平台环境变量（一颗芯片一份）。用法：cd $FNX_SW_DIR && source fnx_sw_env_chipA.csh && fnx_sw
# 路径为示例。选用的变量没有就整行删掉。

# 芯片级工作目录（必需）：平台先建好，再在这里启动 fnx_sw
setenv FNX_SW_DIR        /proj/chipA/software
# 芯片架构描述（必需）：各设备的 ipxact 写 ip.xml 的绝对路径
setenv FNX_SOC_ARCH_FILE /proj/chipA/spec/soc_arch.json
# 交叉编译器（必需）：文件名以 gcc 结尾，objcopy/size 默认取同目录同前缀的
setenv FNX_CC            /opt/riscv/bin/riscv64-unknown-elf-gcc
# 选用：objcopy / size 不在 FNX_CC 旁边时才设
#setenv FNX_OBJCOPY      /opt/riscv/bin/riscv64-unknown-elf-objcopy
#setenv FNX_SIZE         /opt/riscv/bin/riscv64-unknown-elf-size
# 选用：模拟器；都不设就只编译不仿真
setenv FNX_SPIKE         /opt/spike/bin/spike
#setenv FNX_QEMU         /opt/qemu/bin/qemu-system-riscv32
```

### 各 IP 的输入放在哪（不设变量）

soc_arch 里每个设备用下面三个字段给出该 IP 的输入，都写绝对路径：`ipxact`（ip.xml）、`component_json`、`spec`。`component_json`、`spec` 没写时，到 ip.xml 所在目录找固定文件名（`component.json`、`spec.md`）。平台有 ip.xml 时应写 `ipxact`；没有 ip.xml 的 IP 也可以只写 `component_json`（寄存器取自它）和 `spec`，但拿不到行为模型（它按 ip.xml 的位置找）：

| 输入 | 位置（以 `ipxact` = `/proj/ips/uart/docs/ip.xml` 为例） | 缺了会怎样 |
|---|---|---|
| ip.xml | `ipxact` 指向的文件 | 退回 `component.json` 的寄存器表；两者都没有就在「某个 IP 缺输入」确认点让人决定（补文件 / 占位 / 跳过） |
| component.json | 设备的 `component_json` 字段；没写就是同目录的 `/proj/ips/uart/docs/component.json`（实例基址、IRQ、时钟门控位号，另一份寄存器表） | 少一份交叉核对来源；时钟门控位号没有来源 |
| spec | 设备的 `spec` 字段；没写就是同目录的 `/proj/ips/uart/docs/spec.md` | 不生成 HAL；`sw-hal-doc` 无法生成 |
| 行为模型 | ip.xml 上两级目录的 `model/*.py`：`/proj/ips/uart/model/` | 不阻塞 |

三个字段都没写就当作该 IP 没有输入。同一驱动库的多个设备（如 tim0、tim1）这三个字段须一致，包括写不写（一个写了、一个没写也算不一致）。只改这些字段指向的路径，也和文件内容变了一样，算该 IP 的输入变了。还没完成验证的 IP 可以暂时没有 ip.xml。平台应让同一 IP 的 `ipxact` 与 fnx_dv 的 `FNX_REG_XML` 指向同一份文件。

时钟门控：寄存器偏移取自 soc_arch 中 rcc 设备的 ip.xml，各外设的位号取自各 IP 的 `component.json`；不要求在 soc_arch 里写位号。
