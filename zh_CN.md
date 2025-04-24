# OpenWrt 配置 ipip6 教程（适用于 NTT + v6plus + 固定 IPv4）

[🇺🇸 English](./README.md) | [🇨🇳 中文](./zh_CN.md) | [🇯🇵 日本語](./ja_JP.md)

---

本仓库提供一份面向在日本使用 NTT 系运营商（`v6plus` + **固定 IPv4**）的用户，在 OpenWrt 路由器上手动配置 `ipip6` 隧道的详细教程。

このrepoは、NTT系のISP（`v6plus` + **固定IPv4**）を利用する日本国内のユーザー向けに、OpenWrt上で `ipip6` トンネルを構成するための手順を提供します。

This repository provides a detailed guide for configuring an `ipip6` tunnel on OpenWrt with NTT-based ISPs in Japan, specifically for users using `v6plus` and a **static IPv4 address**.

本教程基于以下前提：

- 使用 DHCPv6 Prefix Delegation（PD）
- ISP 提供 `/56` 的 IPv6 前缀（若为 `/64`，则行为未知）
- 使用 ISP 提供的 BR 地址、固定 IPv4 地址和 interface ID 手动配置 `ipip6` 隧道，实现 4in6 通信

📌 旨在帮助用户成功建立 IPv4 over IPv6 的 OpenWrt 隧道连接。

---

## 配置步骤

### 1. 安装 ipip6 包

使用以下仓库安装：

> https://github.com/HiraokaHyperTools/openwrt-ipip6

安装完成后，请**重启路由器**。

---

### 2. 配置 `wan6` 接口（DHCPv6 Client）

- 创建新接口  
- 协议：`DHCPv6 Client`  
- 分配物理接口并设置防火墙 zone  
- 确认成功获取 `/56` 的 IPv6 前缀，例如：`240b:114:514:8100::/56`

---

### 3. 添加 `ipip6` 接口

- 协议：`IPv4 over IPv6 (ipip6)`  
- 从 OpenWrt 23.05 起可通过 `opkg` 安装使用

#### 3.1 基本设置

- Remote IPv6 address：例如 `2404:9200:225:100::65`  
- Local IPv4 address：ISP 分配的静态 IPv4 地址  
- Local IPv6 address：将获取的前缀与 interface ID 组合  
  示例：`240b:114:514:8100:1919:0810:0893:0000`

#### 3.2 高级设置

- Tunnel link：选择 `wan6`  
- Encapsulation limit：设为 `ignore`  
- IPv6 assignment length：`64`  
- IPv6 suffix：`::1919:0810:0893:0000`  
- IPv6 preference：`0`  
- MTU：默认 `1280`，可设置最大 `1460`

#### 3.3 防火墙设置

- 指定防火墙 zone  
- 关闭 DHCPv6 与 RA（Router Advertisement）

---

### 4. 配置 LAN 接口

- IPv6 assignment length：`/64`  
- IPv6 assignment hint：例如 `1` 或 `10`  
- 设置为 IPv6 server 模式

---

### 5. 状态确认

#### 5.1 检查 ipip6 接口地址

接口地址应类似：`240b:114:514:8100::1/64`  
可通过 SSH 验证：

```
link/tunnel6 240b:114:514:8100:1919:0810:0893:0000 peer 2404:9200:225:100::65
```

#### 5.2 检查 LAN 地址

assignment hint 为 `1`：`240b:114:514:8101::1/64`  
assignment hint 为 `10`：`240b:114:514:8110::1/64`

---

### 6. 验证隧道连通性

- 若 `ipip6` 接口有 TX/RX 流量，则表示隧道建立成功  
- 可通过测试网站验证 IPv4/IPv6 连通性

---

## 总结

**`ipip6` 接口必须分配一个与 DHCPv6 获取的前缀一致的 IPv6 地址。**

如果未设置 assignment hint 和 IPv6 preference，可能出现：

- `lan` 分配 `8100::/64`
- `wan` 分配 `8101::/64`

这会导致生成的 `ipip6` 地址不符合 ISP 要求，从而验证失败。

**解决方法：**

- 给 `lan` 接口设置 assignment hint  
- 给 `ipip6` 接口设置 IPv6 preference

确保正确地址组合，才能成功建立隧道。

> Keywords: OpenWrt, ipip6, IPv4 over IPv6, 4in6, v6plus, static IP, NTT, Japan, OpenWrt tunneling, 固定IPv4, IPv6トンネル