# ipip6 Configuration Guide for OpenWrt (NTT + v6plus + Static IPv4)

[🇺🇸 English](./README.md) | [🇨🇳 中文](./zh_CN.md) | [🇯🇵 日本語](./ja_JP.md)

---

このリポジトリは、NTT系のISP（`v6plus` + **固定IPv4**）を利用する日本国内のユーザー向けに、OpenWrt上で `ipip6` トンネルを構成するための詳細な手順を提供します。

本チュートリアルは以下を前提としています：

- DHCPv6 Prefix Delegation（PD）の利用  
- ISPから配布された `/56` のIPv6プレフィックス（`/64`の場合の動作は不明）  
- ISPから提供されたBRアドレス・固定IPv4アドレス・Interface IDを使用して `ipip6` を手動設定し、4in6トンネルを有効にする

📌 目的は、OpenWrtルーターでIPv4 over IPv6接続を確立することです。

✅ この方法は、official-like OpenWrt および QWRT の両方で正常に動作することが確認されています。

---

## 設定手順

### 1. ipip6パッケージのインストール

以下のリポジトリを使ってインストールします：

> https://github.com/HiraokaHyperTools/openwrt-ipip6

インストール後、**ルーターを再起動**してください。

---

### 2. `wan6`インターフェースの設定（DHCPv6 Client）

- 新しいインターフェースを作成  
- プロトコル：`DHCPv6 Client`  
- 対応する物理インターフェースとFirewall Zoneを割り当て  
- 例：`240b:114:514:8100::/56` のような `/56` プレフィックスが取得できるか確認

---

### 3. `ipip6`インターフェースの追加

- プロトコル：`IPv4 over IPv6 (ipip6)`  
- OpenWrt 23.05以降は `opkg` により対応

#### 3.1 基本設定

- Remote IPv6 address：例）`2404:9200:225:100::65`  
- Local IPv4 address：ISPが提供する固定IPv4アドレス  
- Local IPv6 address：取得したプレフィックス + Interface ID  
  例：`240b:114:514:8100:1919:0810:0893:0000`

#### 3.2 詳細設定

- Tunnel link：`wan6`  
- Encapsulation limit：`ignore`  
- IPv6 assignment length：`64`  
- IPv6 suffix：`::1919:0810:0893:0000`  
- IPv6 preference：`0`  
- MTU：デフォルト `1280`、最大 `1460` に設定可能

#### 3.3 Firewall設定

- 適切なFirewall Zoneに割り当て  
- `DHCPv6`と`Router Advertisement`は無効に設定

---

### 4. LANインターフェースの設定

- IPv6 assignment length：`/64`  
- IPv6 assignment hint：例）`1` または `10`  
- 両方のIPv6モードを `server` に設定

---

### 5. ステータス確認

#### 5.1 ipip6アドレスの確認

- 表示例：`240b:114:514:8100::1/64`  
- 実際のトンネルアドレスはSSHで確認：

  ```
  link/tunnel6 240b:114:514:8100:1919:0810:0893:0000 peer 2404:9200:225:100::65
  ```

#### 5.2 LANアドレスの確認

assignment hintが `1` の場合：→ `240b:114:514:8101::1/64`  
assignment hintが `10` の場合：→ `240b:114:514:8110::1/64`

---

### 6. トンネル接続の確認

- `ipip6`インターフェースにTX/RXトラフィックが表示されていれば成功  
- IPv4/IPv6接続は外部テストサイトで確認可能

---

## まとめ

**`ipip6` インターフェースには、DHCPv6で取得したプレフィックスと一致するIPv6アドレスを設定する必要があります。**

assignment hintやIPv6 preferenceを設定しないと：

- `lan` に `8100::/64`
- `wan` に `8101::/64`

のような異なるプレフィックスが割り当てられる可能性があり、トンネルアドレスが正しく生成されず認証に失敗します。

**対策：**

- `lan` にIPv6 assignment hintを設定  
- `ipip6` にIPv6 preferenceを設定

正しいアドレスが取得できれば、トンネルは正常に確立されます。

> 🙏 **謝辞**  
> 初期の構成とトラブルシューティングに協力してくれた [@missing233](https://github.com/missing233) さんに深く感謝いたします。  
> 本ガイドは、私たちが共同で設定し検証した内容に基づいて作成されています。

> Keywords: OpenWrt, ipip6, IPv4 over IPv6, 4in6, v6plus, static IP, NTT, Japan, OpenWrt tunneling, 固定IPv4, IPv6トンネル