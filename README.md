# Openpilot PC 工具箱 (op_pc_toolbox)

图形化的 **openpilot**（dragonpilot / opendbc / …）PC 端开发辅助工具，纯 C + GTK3 实现。
本仓库为**二进制发布仓库**，仅包含可运行产物，无需自行编译。

---

## 这是什么

`op_pc_toolbox` 把 openpilot 开发中常用的几套命令行操作整合成一个图形界面，让不熟悉终端的同学也能完成：克隆源码、搭建环境、编译固件、监控运行进程。所有命令在后台异步执行，UI 不会卡死，可随时取消。

> 本工具采用**设备授权**机制，仅供已授权设备使用。首次运行会显示你的设备码，联系作者加入白名单后即可永久使用。

---

## 仓库内容

| 文件 | 说明 |
|---|---|
| `op_pc_toolbox` | 主程序（GTK3 图形界面） |
| `op_pc_license_admin` | 授权白名单管理工具（命令行，作者侧使用） |
| `liblicense_data.so` | 授权白名单共享库（主程序运行时加载） |

三个文件需放在**同一目录**下运行（主程序会从自身所在目录加载 `liblicense_data.so`）。

---

## 运行环境

仅支持 **x86_64 Ubuntu 20.04 / 22.04**（其他发行版未做兼容测试）。

安装运行依赖：

```bash
sudo apt update
sudo apt install -y libgtk-3-0
```

> 注：这是运行时依赖；如果你还要在本机克隆/编译 openpilot，需另外安装 `git`、`build-essential`、`scons` 等，具体参考 openpilot 官方文档。

---

## 快速开始

1. 下载本仓库的三个文件到同一目录，例如 `~/op_pc_toolbox/`：

   ```bash
   mkdir -p ~/op_pc_toolbox && cd ~/op_pc_toolbox
   # 将 op_pc_toolbox / op_pc_license_admin / liblicense_data.so 放到此目录
   chmod +x op_pc_toolbox op_pc_license_admin
   ```

2. 运行主程序：

   ```bash
   ./op_pc_toolbox
   ```

3. 首次启动若弹出「未授权」对话框，按下方 [授权流程](#授权流程) 操作即可。

---

## 授权流程

本工具基于**设备码**授权。设备码由本机的机器 ID、BIOS UUID、主机名、MAC 地址等因子计算得出（16 位十六进制），每台机器唯一。

1. 运行 `./op_pc_toolbox`，若未授权会弹出对话框，其中高亮显示**你的设备码**。
2. 点击「📋 复制设备码」复制设备码。
3. 通过以下任一方式联系作者，附上设备码：

   | 渠道 | 号码 |
   |---|---|
   | 🐧 QQ | `2218808853` |
   | 💬 微信 | `17695579920` |

4. 作者将你的设备码加入白名单后，会发布新的 `liblicense_data.so`。**下载新的 `liblicense_data.so` 覆盖本地旧文件**，然后在对话框点击「🔄 重新验证」即可通过授权。

> 重新验证无需重启程序；授权通过后本机即可永久使用（除非硬件大幅变更导致设备码改变）。

### 自查设备码

不启动主程序也能查看本机设备码：

```bash
./op_pc_license_admin show
```

---

## 功能页面

主程序左侧侧边栏切换不同功能：

| 页面 | 用途 |
|---|---|
| **克隆 / 更新** | 克隆或拉取 openpilot、dragonpilot、opendbc、panda、cereal、msgq、rednose 等仓库，支持浅克隆与子模块递归 |
| **环境构建** | 一键执行 `tools/op.sh setup` 搭建 openpilot 开发环境，可分步运行 |
| **编译** | 调用 `scons -u -j$(nproc)` 编译 openpilot，可选目标（all / camerad / controlsd / plannerd / modeld / clean）、并发数、是否先 clean |
| **运行时监控** | 周期刷新系统进程列表（按命令行过滤），显示 PID / USER / CPU / RSS，方便观察 camerad、controlsd 等进程 |
| **疑难解答** | 常见问题与解决办法 |
| **硬件购买** | 推荐的硬件购买信息 |

---

## 配置文件

- `~/.config/op_pc_toolbox/settings.ini` —— 自动保存上次选择的目录、分支、并发数等设置。

---

## `op_pc_license_admin` 命令参考

该工具主要供**作者侧**管理白名单，普通用户一般只需 `show` 查看设备码。

```bash
./op_pc_license_admin show            # 显示本机设备码
./op_pc_license_admin check           # 检查当前设备是否已授权
./op_pc_license_admin list            # 列出白名单中所有设备码
./op_pc_license_admin add <code>      # 添加设备码到白名单
./op_pc_license_admin remove <code>   # 从白名单移除设备码
./op_pc_license_admin clear           # 清空白名单
./op_pc_license_admin import <file>   # 从文件批量导入设备码
./op_pc_license_admin export <file>   # 导出白名单到文件
./op_pc_license_admin path            # 显示白名单源文件与 .so 库路径
./op_pc_license_admin build           # 显示编译 liblicense_data.so 的命令
```

> 白名单源文件为 `license_data.cc`，编译后生成 `liblicense_data.so` 分发给用户。修改白名单后需重新编译 `.so` 并随版本发布。

---

## 高级：自定义授权库路径

主程序默认从自身所在目录加载 `liblicense_data.so`。如需指定其他路径，可设置环境变量：

```bash
OP_PC_LICENSE=/path/to/liblicense_data.so ./op_pc_toolbox
```

---

## 已知限制

- 仅支持 x86_64 Ubuntu 20.04 / 22.04，其他发行版未做兼容测试。
- 不支持 macOS / Windows。
- 运行时监控的 CPU% 为累计值（开机以来），非瞬时值。
