# 深圳技术大学校园网自动登录（深澜）

> 本仓库提供我校深澜校园网的**自动登录配置模板**和使用教程。配合开源工具 [BitSrunLoginGo](https://github.com/Mmx233/BitSrunLoginGo) 使用，可以免去每次手动登录，支持掉线自动重连、开机自启。

> ⚠️ 本仓库**只提供配置模板和教程**，软件本身请从原作者 release 自行下载，见下文。

## 快速开始

1. 下载对应平台的软件（见 [下载软件](#1-下载软件)）
2. 复制 `Config.example.yaml` 为 `Config.yaml`，填入你的**账号**和**密码**（见 [准备配置](#2-准备配置)）
3. 运行（见 [运行](#3-运行)）

下面按步骤详细说明。

---

## 详细使用流程

### 1. 下载软件

打开原作者的 release 页面：

👉 **https://github.com/Mmx233/BitSrunLoginGo/releases**

选择对应平台的 `.tar.gz` 压缩包下载：

| 你的环境 | 下载文件 |
|---------|---------|
| Windows 64 位 | `bitsrun_windows_amd64.tar.gz` |
| Windows 32 位 | `bitsrun_windows_386.tar.gz` |
| macOS（Intel 芯片） | `bitsrun_darwin_amd64.tar.gz` |
| macOS（Apple 芯片 M 系列） | `bitsrun_darwin_arm64.tar.gz` |
| Linux 64 位 | `bitsrun_linux_amd64.tar.gz` |
| 路由器 / 树莓派（ARM 64 位） | `bitsrun_linux_arm64.tar.gz` |
| 路由器（ARM 32 位） | `bitsrun_linux_arm_v7.tar.gz` |
| 路由器（MIPS） | `bitsrun_linux_mips.tar.gz` |
| 路由器（MIPS 小端） | `bitsrun_linux_mipsle.tar.gz` |

**关于文件名后缀**：有些文件带 `_v2` / `_v3` / `_v4` / `_v9`（CPU 微架构级别）或 `hardfloat` / `softfloat`（浮点方式）后缀。如果你不确定选哪个，**选没有这些后缀的版本**，兼容性最好。

**路由器怎么查架构**：SSH 登录路由器后执行 `uname -m`，输出 `aarch64` 选 `linux_arm64`，`mips` 选 `linux_mips`，`armv7l` 选 `linux_arm_v7`，`x86_64` 选 `linux_amd64`。

下载后解压，得到一个可执行文件：
- Windows：`bitsrun.exe`
- macOS / Linux / 路由器：`bitsrun`

### 2. 准备配置

1. 下载本仓库的 [`Config.example.yaml`](./Config.example.yaml)，**复制一份并重命名为 `Config.yaml`**。
2. 用文本编辑器打开 `Config.yaml`，只需改动 `form` 下这几项：

```yaml
form:
  domain: 172.19.0.5   # 保持默认，全校通用
  username: ""         # 改成你的校园网账号（学号）
  user_type: ""        # 改成你的运营商，见下方说明
  password: ""         # 改成你的校园网密码
```

**`user_type` 怎么填**：登录校园网时，账号后面会自动加上运营商后缀（形如 `学号@cmcc`）。把 `@` 后面的部分填到这里：

| 运营商 | 填写值 |
|--------|--------|
| 移动 | `cmcc` |
| 联通 | `cucc` |
| 电信 | `ctcc` |
| 没有运营商后缀 | 留空 `""` |

> 其余字段（`acid`、`enc` 等）对全校同学是通用的，一般不用改。如果登录失败，参考 [登录参数获取](#登录参数获取)。

**配置文件必须和可执行文件放在同一目录下**（或运行时用 `--config` 指定路径）。

### 3. 运行

**Windows**：在 `bitsrun.exe` 所在目录打开命令行（在文件夹地址栏输入 `cmd` 回车），执行：

```bat
bitsrun.exe --config=Config.yaml
```

**macOS / Linux**：

```bash
chmod +x bitsrun
./bitsrun --config=./Config.yaml
```

**路由器（OpenWrt 等）**：把 `bitsrun` 和 `Config.yaml` 上传到设备同一目录，然后：

```sh
chmod +x bitsrun
./bitsrun --config=./Config.yaml
```

> 💡 **首次运行建议加 `--debug`**（如 `./bitsrun --config=./Config.yaml --debug`），会打印详细日志，方便排查问题。确认没问题后去掉即可。

如果想让程序在后台常驻、掉线自动重连，确认配置里 `settings.guardian.enable: true`（模板默认已开启）。

### 4. 验证是否成功

运行后看输出，出现登录成功的提示即表示已认证。更直接的验证方式：

- 电脑上打开浏览器，看能否正常上外网；
- 命令行 `ping www.baidu.com`，能 ping 通说明联网成功。

---

## 配置字段说明

### form（登录信息，需要你填写）

| 字段 | 含义 | 是否必填 |
|------|------|---------|
| `domain` | 校园网认证服务器地址 | 保持默认 `172.19.0.5` |
| `username` | 校园网账号（学号） | ✅ 必填 |
| `user_type` | 运营商（`cmcc`/`cucc`/`ctcc`/空） | ✅ 按实际情况填 |
| `password` | 校园网密码 | ✅ 必填 |

### meta（登录参数，全校通用，一般不用改）

| 字段 | 含义 | 说明 |
|------|------|------|
| `n` | 认证参数 | 保持默认 `200` |
| `type` | 认证类型 | 保持默认 `1` |
| `acid` | 认证服务器标识 | 本校为 `17`，一般不用改 |
| `enc` | 加密方式 | 默认 `srun_bx1`，可用 `--auto-enc` 自动嗅探 |
| `os` / `name` | 伪造的系统信息 | 保持默认即可 |
| `info_prefix` | info 字段前缀 | 默认 `SRBX1` |
| `double_stack` | 双栈（IPv6） | 默认 `false` |

### settings（进阶设置，可选）

| 字段 | 含义 | 说明 |
|------|------|------|
| `basic.timeout` | 请求超时（秒） | 默认 `5` |
| `basic.dns_server` | 指定 DNS 服务器 | 留空用系统默认；解析异常时可填 `114.114.114.114` |
| `guardian.enable` | 守护模式 | `true` 时后台常驻、掉线自动重连 |
| `guardian.duration` | 在线检查周期（秒） | 默认 `300` |
| `log.debug_level` | 调试日志 | 排查问题时设为 `true` |

> 其余字段（`backoff` 重试退避、`ddns`、`reality`、`custom_header`）保持模板默认即可，一般无需修改。

---

## 登录参数获取

正常情况下，模板里已经填好了全校通用的 `acid`，`enc` 用默认值或自动嗅探即可。只有遇到**登录失败**才需要自己获取参数，提供两种方案：

### 方案一：自动嗅探（推荐，最简单）

运行时加上两个参数：

```bash
./bitsrun --config=./Config.yaml --auto-acid --auto-enc
```

程序会自动嗅探 `acid` 和 `enc` 的真实值，无需手动抓包。

### 方案二：浏览器抓包（自动嗅探失败时用）

1. 电脑连上校园网，打开登录页，按 **F12** 打开开发者工具，切到 **Network（网络）** 标签，勾选 **Preserve log（保留日志）**。
2. 正常输入账号密码登录。
3. 在请求列表中找到名为 `/srun_portal` 的请求，点开查看 **Payload（请求体）**，里面能找到 `acid`、`enc` 等字段。
4. 把抓到的值填进 `Config.yaml` 的 `meta` 里。

---

## 常见问题（FAQ）

### 登录失败怎么办？

运行时加 `--debug` 看详细日志：

```bash
./bitsrun --config=./Config.yaml --debug
```

日志会提示具体哪一步出错。常见原因：账号密码错误、`user_type` 填错、`acid`/`enc` 不对（参考上文两个方案重新获取）。

### 程序提示「读取配置文件失败：not a directory」

说明 `Config.yaml` 的父目录不存在或路径写错。检查：

- `Config.yaml` 是否真的存在、拼写是否正确（注意大小写）；
- `--config` 指定的路径是否正确；
- 若放在子目录（如 `/etc/bitsrun/Config.yaml`），确认 `/etc/bitsrun` 目录已创建：`mkdir -p /etc/bitsrun`。

### 能 ping 通公网 IP，但域名解析不了（DNS 问题）

这是校园网 DHCP 分配的 DNS 不可用导致的。有两种改法：

1. **改配置文件**：在 `settings.basic.dns_server` 里填公共 DNS（如 `223.5.5.5`），然后重启程序。
2. **路由器上指定 DNS 上游**（OpenWrt）：
   ```sh
   uci add_list dhcp.@dnsmasq[0].server='223.5.5.5'
   uci add_list dhcp.@dnsmasq[0].server='114.114.114.114'
   uci commit dhcp
   /etc/init.d/dnsmasq restart
   ```

### 怎么判断程序在不在线？

- 守护模式开启时，程序会周期性检查并在日志里报告在线状态；
- 直接 `ping www.baidu.com` 最直观。

---

## 致谢

- 登录工具：[BitSrunLoginGo](https://github.com/Mmx233/BitSrunLoginGo)（原作者 [Mmx233](https://github.com/Mmx233)），本仓库仅提供配置模板与教程。

## 许可证

本仓库的配置模板与文档仅作学习交流使用。软件本身的许可证请以原作者项目为准。
