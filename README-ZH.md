# GoAuthing

[![Build Status](https://github.com/Neon-Wang/GoAuthing-for-CUC/actions/workflows/go.yml/badge.svg)](https://github.com/Neon-Wang/GoAuthing-for-CUC/actions)
![GPLv3](https://img.shields.io/badge/license-GPLv3-blue.svg)

中国传媒大学校园网（net.cuc.edu.cn）命令行认证工具。

请知悉：当前仍为早期测试版本，部分功能尚未充分验证。若遇到问题，欢迎提交 Issue。

## 下载

从 [Releases](https://github.com/Neon-Wang/GoAuthing-for-CUC/releases) 下载预编译好的可执行文件。

## 使用说明

直接运行 `./auth-cuc`，按提示输入用户名和密码即可。

```help
NAME:
   auth-cuc - 中国传媒大学校园网认证工具

USAGE:
   auth-cuc [options]
   auth-cuc [options] auth [auth_options]
   auth-cuc [options] deauth [auth_options]
   auth-cuc [options] online [online_options]

VERSION:
   2.3.5

AUTHORS:
   Yuxiang Zhang <yuxiang.zhang@tuna.tsinghua.edu.cn>
   Nogeek <ritou11@gmail.com>
   ZenithalHourlyRate <i@zenithal.me>
   Jiajie Chen <c@jia.je>
   KomeijiOcean <oceans2000@126.com>
   Sharzy L <me@sharzy.in>
   Modified by Neon Wang <i@jiashengfan.space>

COMMANDS:
     auth     （默认）通过 net.cuc.edu.cn 认证
       OPTIONS:
         --ip value         为指定 IP 进行认证
         --no-check, -n     跳过在线检测，始终发送登录请求
         --logout, -o       登出当前已登录账号（与 deauth 命令相同，保留以兼容旧版）
         --ipv6, -6         IPv6 认证
         --host value       自定义 srun4000 认证服务器主机名
         --insecure         使用 http 而非 https
         --keep-online, -k  登录后保持在线（保活）
         --ac-id value      指定 ac_id
     deauth   通过 net.cuc.edu.cn 登出
       OPTIONS:
         --ip value      为指定 IP 登出
         --no-check, -n  跳过在线检测，始终发送登出请求
         --ipv6, -6      IPv6
         --host value    自定义 srun4000 主机名
         --insecure      使用 http
         --ac-id value   指定 ac_id
     online   保持本机在线（保活）
       OPTIONS:
         --ipv6, -6  仅保持 IPv6 连接在线

GLOBAL OPTIONS:
   --username name, -u name           CUC 账号
   --password password, -p password  CUC 密码
   --config-file path, -c path       配置文件路径，默认 ~/.auth-cuc
   --hook-success value              登录/登出成功后要执行的 shell 命令
   --daemonize, -D                   不从标准输入读取账号密码；减少日志输出
   --debug                           输出调试信息
   --help, -h                        显示帮助
   --version, -v                     显示版本
```

程序会按以下顺序查找配置文件：`$XDG_CONFIG_HOME/auth-cuc`、`~/.config/auth-cuc`、`~/.auth-cuc`。  
可用 JSON 格式的配置文件保存用户名、密码及其他选项，例如：

```json
{
  "username": "your-username",
  "password": "your-password",
  "host": "",
  "ip": "166.xxx.xx.xx",
  "debug": false,
  "useV6": false,
  "noCheck": false,
  "insecure": false,
  "daemonize": false,
  "acId": ""
}
```

若无特殊需求，配置文件中只需填写 `username` 和 `password`。`host` 一般无需填写，代码中的默认值即可。`useV6` 为 true 时会自动选用对应主机。`ip` 仅在为其他设备（非运行 auth-cuc 的本机）认证时填写，否则留空。若某设备无法自动获取正确的 acid，可通过 `acId` 手动指定。其余选项含义见名知意。

## 开机自启

建议先手动配置并开启 `debug` 运行一次，确认配置无误后再配置为系统服务。`daemonize` 会使程序只输出错误日志，因此调试应在未使用 daemon 时手动完成。以系统服务方式运行时通常会自动开启 daemon 模式（参见相关 systemd 单元）。

### Systemd

在基于 systemd 的 Linux 发行版上配置开机自动认证，可参考 `docs/systemd` 目录下的文件。按需修改其中的路径后，将单元文件复制到 `/etc/systemd` 目录。

注意确保程序有权限读取配置文件。  
使用 `system/goauthing.service` 时，服务以 `nobody` 运行，无法读取 `/etc/goauthing.json`，可通过以下命令授权：

```shell
setfacl -m u:nobody:r /etc/goauthing.json
```

若希望更安全，可选用 `system/goauthing@.service` 或 `user/goauthing.service`，并将配置文件放在用户主目录下。

### OpenWRT

OpenWRT 用户有两种方式：使用配置文件的 `goauthing`，或通过 UCI 配置的 `goauthing@`。初始化脚本需放在 `/etc/init.d/`。使用后者时，可按以下步骤配置：

```shell
touch /etc/config/goauthing
uci set goauthing.config=goauthing
uci set goauthing.config.username='<你的CUC账号>'
uci set goauthing.config.password='<你的CUC密码>'
uci commit goauthing
/etc/init.d/goauthing enable
/etc/init.d/goauthing start
```

## 自行编译

需要 Go 1.11 或更高版本。

```shell
export GO111MODULE=on
go build -o auth-cuc github.com/Neon-Wang/GoAuthing-for-CUC/cli
```

## 致谢

本项目受以下项目启发：

- <https://github.com/jiegec/auth-tsinghua>
- <https://github.com/Berrysoft/TsinghuaNet>
