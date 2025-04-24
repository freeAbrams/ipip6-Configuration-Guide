# ipip6 Configuration Guide for OpenWrt (NTT + v6plus + Static IPv4)

[🇺🇸 English](./README.md) | [🇨🇳 中文](./zh_CN.md) | [🇯🇵 日本語](./ja_JP.md)

---

This repository provides a detailed guide for configuring an `ipip6` tunnel on OpenWrt with NTT-based ISPs in Japan, specifically for users using `v6plus` and a **static IPv4 address**.

このrepoは、NTT系のISP（`v6plus` + **固定IPv4**）を利用する日本国内のユーザー向けに、OpenWrt上で `ipip6` トンネルを構成するための手順を提供します。

本仓库提供一份面向在日本使用 NTT 系运营商（`v6plus` + **固定 IPv4**）的用户，在 OpenWrt 路由器上手动配置 `ipip6` 隧道的详细教程。

This tutorial is based on:

- Use of DHCPv6 Prefix Delegation (PD)
- `/56` IPv6 prefix from the ISP (behavior with `/64` is unknown)
- Manual configuration of `ipip6` to enable 4in6 tunnel by BR Address, IPv4 Adress and interface ID provided by your ISP.

📌 It aims to help users who have received a BR address, static IPv4, and interface ID from their ISP to establish IPv4-over-IPv6 connectivity on their OpenWrt router.

✅ This method has been confirmed to work on both official-like OpenWrt and QWRT builds.

---
## Setup Procedure

### 1. Install ipip6 Package

Use the following repository to install:

> https://github.com/HiraokaHyperTools/openwrt-ipip6

After installation, **reboot your router**.

---

### 2. Configure the `wan6` Interface (DHCPv6 Client)

- Create a new interface  
- Protocol: `DHCPv6 Client`  
- Assign the correct physical interface and firewall zone  
- Confirm that you receive a `/56` IPv6 prefix. e.g., `240b:114:514:8100::/56`

---

### 3. Add the `ipip6` Interface

- Protocol: `IPv4 over IPv6 (ipip6)`  
- This protocol is supported via `opkg` since OpenWrt 23.05

#### 3.1 Basic Settings

- Remote IPv6 address: e.g., `2404:9200:225:100::65`  
- Local IPv4 address: the static IPv4 provided by your ISP  
- Local IPv6 address: combine the obtained IPv6 prefix + Interface ID  
  Example: `240b:114:514:8100:1919:0810:0893:0000`

#### 3.2 Advanced Settings

- Tunnel link: `wan6`  
- Encapsulation limit: `ignore`  
- IPv6 assignment length: `64`  
- IPv6 suffix: `::1919:0810:0893:0000`  
- IPv6 preference: `0`  
- MTU: default is `1280`; can be increased up to `1460`

#### 3.3 Firewall Settings

- Assign to a proper firewall zone  
- Disable both DHCPv6 and RA (Router Advertisement)

---

### 4. Configure the LAN Interface

- IPv6 assignment length: `/64`  
- IPv6 assignment hint: e.g., `1` or `10`  
- Set both IPv6 modes to `server`

---

### 5. Status Check

#### 5.1 Verify ipip6 Interface Address

- The interface should display an address like: `240b:114:514:8100::1/64`  
- Use SSH to confirm the actual tunnel address:

  ```
  link/tunnel6 240b:114:514:8100:1919:0810:0893:0000 peer 2404:9200:225:100::65
  ```

#### 5.2 Verify LAN Address

If assignment hint is `1`: → `240b:114:514:8101::1/64`  
If assignment hint is `10`: → `240b:114:514:8110::1/64`

---

### 6. Tunnel Connection Check

- If the `ipip6` interface shows both TX and RX traffic, the tunnel is working  
- Use an online test site to confirm both IPv4 and IPv6 connectivity

---

## Summary

**The `ipip6` interface must be assigned an IPv6 address within the same prefix obtained via DHCPv6.**

For example, if neither assignment hint nor IPv6 preference is set, then:

- `lan` may receive `8100::/64`  
- `wan` may receive `8101::/64`

With such mismatched prefixes, the tunnel IPv6 address may not be constructed correctly and authentication may fail.

**Solution:**

- Assign an `IPv6 assignment hint` to `lan`  
- Set `IPv6 preference` for the `ipip6` interface

This ensures the correct IPv6 address is assigned and the tunnel will authenticate successfully.

> 🙏 **Acknowledgement**  
> Special thanks to [@missing233](https://github.com/missing233) for initially helping me set up and troubleshoot this configuration.  
> This guide is based on the setup we developed and verified together.

> Keywords: OpenWrt, ipip6, IPv4 over IPv6, 4in6, v6plus, static IP, NTT, Japan, OpenWrt tunneling, 固定IPv4, IPv6トンネル