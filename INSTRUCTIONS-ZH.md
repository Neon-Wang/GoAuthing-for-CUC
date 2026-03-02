---

# auth-cuc 使用说明书

auth-cuc 是中国传媒大学校园网（net.cuc.edu.cn）的命令行认证工具，用于在终端下一键登录/登出校园网。
声明：本 Release 为 Pre-release，功能可能未经完全测试。若发现使用问题，欢迎提交 Issue。

---

本说明涵盖了「认证」与「保活」相关用法；更多命令可通过 `./auth-cuc --help` 查看。

---

## 一、获取程序

从 [Releases](https://github.com/Neon-Wang/GoAuthing-for-CUC/releases) 页面下载与您系统对应的预编译包。

- **macOS (Intel)**：`auth-cuc.macos.x86_64`
- **macOS (Apple Silicon)**：`auth-cuc.macos.arm64`
- **Linux (x86_64)**：`auth-cuc.linux.x86_64`
- **Linux (arm64)**：`auth-cuc.linux.arm64`
- **Windows**：对应 Windows 版本可执行文件

在终端中进入程序所在目录后，可先为其添加执行权限（Linux/macOS）：

```bash
chmod +x auth-cuc.XXX
```

> **提示：** 下文所有示例中的 `./auth-cuc` 仅为引用示例，实际使用时请根据下载的文件名进行调整，例如将其重命名为 `./auth-cuc` 或直接使用 `./auth-cuc.macos.arm64` 等形式。

---

## 二、认证（登录）的几种方式

### 2.1 直接运行（交互输入账号密码）

在终端执行：

```bash
./auth-cuc
```

或显式使用 `auth` 子命令：

```bash
./auth-cuc auth
```

程序会依次提示输入 **用户名** 和 **密码**，输入完成后自动向校园网发起认证。认证成功后会输出 `Login Successfully!`。

### 2.2 使用配置文件（推荐）

将账号、密码等写在配置文件里，可避免每次手动输入，也便于配合开机自启或脚本使用。

程序会按以下顺序查找配置文件（找到即用）：

1. 环境变量 `$XDG_CONFIG_HOME/auth-cuc`
2. `~/.config/auth-cuc`
3. `~/.auth-cuc`

**配置文件为 JSON 格式。** 认证相关的最简示例：

```json
{
  "username": "你的学号或账号",
  "password": "你的密码"
}
```

保存为上述任一路径（例如 `~/.auth-cuc`）后，在终端执行：

```bash
./auth-cuc
```

或：

```bash
./auth-cuc auth
```

程序会从配置文件读取账号和密码并完成认证，无需再在终端输入。

### 2.3 命令行参数指定账号密码

不写配置文件时，也可通过全局参数传入账号和密码：

```bash
./auth-cuc auth -u 你的账号 -p 你的密码
```

**注意：** 在共享或多用户环境下，使用 `-p` 会在进程列表等处暴露密码，建议优先使用配置文件，并设置好文件权限（如 `chmod 600 ~/.auth-cuc`）。

### 2.4 指定配置文件路径

若配置文件不在默认路径，可用 `-c` / `--config-file` 指定：

```bash
./auth-cuc -c /path/to/your/config auth
```

---

## 三、认证（auth）相关选项说明

以下选项仅围绕「认证」场景，在 `auth-cuc auth` 下使用。

| 选项 | 简写 | 说明 |
|------|------|------|
| `--username` | `-u` | 校园网账号（学号等） |
| `--password` | `-p` | 校园网密码 |
| `--config-file` | `-c` | 配置文件路径，默认会查找 `~/.config/auth-cuc` 或 `~/.auth-cuc` |
| `--no-check` | `-n` | 不检查当前是否已在线，直接发起登录请求（适用于“已在线也要重新登录”的情况） |
| `--ipv6` | `-6` | 使用 IPv6 进行认证（net.cuc.edu.cn） |
| `--host` | — | 自定义认证服务器主机名，一般无需填写 |
| `--insecure` | — | 使用 HTTP 而非 HTTPS（仅在不支持 HTTPS 的环境下考虑使用） |
| `--keep-online` | `-k` | 登录成功后自动进入“保活”模式，定期访问网页以保持在线 |
| `--keep-online-retry` | `-r` | 保活请求连续失败多少次后退出，默认 2；仅在与 `-k` 同时使用时有效 |
| `--online-interval` | `-I` | 保活请求间隔（秒），默认 3；仅在与 `-k` 同时使用时有效 |
| `--ac-id` | — | 指定 ac_id，仅在特殊网络环境下需要 |
| `--debug` | — | 输出调试日志，排查问题时使用 |
| `--help` | `-h` | 显示帮助信息 |

**示例：登录并保持在线（保活间隔 5 秒）**

```bash
./auth-cuc auth -k -I 5
```

**示例：不检查是否已在线，强制重新登录**

```bash
./auth-cuc auth -n
```

---

## 四、配置文件中的认证相关字段

若使用配置文件，与「认证」相关的字段含义如下（与命令行选项对应）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `username` | 字符串 | 校园网账号 |
| `password` | 字符串 | 校园网密码 |
| `host` | 字符串 | 认证服务器主机名，一般留空即可 |
| `ip` | 字符串 | 为指定 IP 认证时填写；本机认证可留空 |
| `noCheck` | 布尔 | 是否跳过在线检查、直接发登录请求 |
| `useV6` | 布尔 | 是否使用 IPv6 认证 |
| `insecure` | 布尔 | 是否使用 HTTP |
| `keepOn` | 布尔 | 登录成功后是否保持在线（保活） |
| `onlineInterval` | 数字 | 保活间隔（秒） |
| `onlineRetry` | 数字 | 保活失败重试次数 |
| `acId` | 字符串 | 指定 ac_id，特殊环境使用 |
| `debug` | 布尔 | 是否输出调试日志 |

**示例：带保活与调试的配置文件**

```json
{
  "username": "你的账号",
  "password": "你的密码",
  "keepOn": true,
  "onlineInterval": 5,
  "onlineRetry": 2,
  "debug": false
}
```

---

## 五、认证成功后的行为

- **未加 `-k` / `--keep-online`**：程序在输出 `Login Successfully!` 后退出，仅完成本次登录。
- **加了 `-k` / `--keep-online`**：登录成功后会进入保活循环，按 `-I` / `onlineInterval` 间隔访问网络以保持在线，直到保活连续失败次数达到 `-r` / `onlineRetry` 后退出。

若当前已经在线且未使用 `-n`，程序会直接提示 “Currently online!”；若同时使用了 `-k`，会直接进入保活循环而不重复登录。

---

## 六、登出（与认证相对）

若需登出当前账号，可使用：

```bash
./auth-cuc deauth
```

或（兼容旧用法）：

```bash
./auth-cuc auth --logout
```

登出时也可通过 `-c` 指定配置文件；若配置了账号密码，可用于在已登录状态下执行登出。

---

## 七、常见问题

1. **提示找不到配置文件**  
   确认文件路径是否为 `~/.auth-cuc` 或 `~/.config/auth-cuc`，或使用 `-c` 显式指定路径。

2. **登录失败 / 报错**  
   先加上 `--debug` 查看详细日志，并确认账号密码、网络（是否在校园网内、是否需要 IPv6）无误。

3. **想长期保持在线**  
   使用 `-k`（或配置文件中的 `keepOn: true`），并视情况调整 `-I`、`-r`；也可配合系统自启（如 systemd、launchd）实现开机自动认证并保活。

4. **使用 `online` 命令时如何自动重认证？**  
   在配置文件中填写 `username` 和 `password`，当保活连续失败达到 `--retry` 次数后，程序会自动执行登出+登录流程。

---

## 八、独立保活（online）

除了 `auth -k` 登录后保活外，程序还提供了独立的 `online` 命令，用于在已登录状态下保持在线，或在后台持续保活。

### 8.1 基本用法

直接运行：

```bash
./auth-cuc online
```

程序会定期访问网站以维持在线状态，默认每 3 秒访问一次外网（百度），连续失败 2 次后退出。

### 8.2 常用选项

| 选项 | 简写 | 说明 |
|------|------|------|
| `--ipv6` | `-6` | 仅保持 IPv6 连接在线 |
| `--retry` | `-r` | 保活请求连续失败多少次后触发重认证，默认 2 |
| `--online-interval` | `-I` | 保活请求间隔（秒），默认 3 |

**示例：配置文件中设置保活参数**

```json
{
  "username": "你的账号",
  "password": "你的密码",
  "keepOnline": true,
  "onlineInterval": 5,
  "onlineRetry": 3
}
```

然后执行：

```bash
./auth-cuc online
```

### 8.3 与 `auth -k` 的区别

| 命令 | 说明 |
|------|------|
| `auth -k` | 先执行登录，登录成功后进入保活循环 |
| `online` | 直接进入保活循环，不主动登录；若配置了账号密码，在保活失败后可自动重新认证 |

`online` 命令适合用于：
- 已经登录，只想保持在线
- 配合 systemd/launchd 等后台服务，实现掉线自动重连
- 不想产生外网流量，仅保持校园网连接

---