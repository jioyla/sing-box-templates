<p align="center">
    <img src="https://sing-box.sagernet.org/assets/icon.svg" width="100px" align="center" />
    <h2 align="center">sing-box-templates</h2>
    <p align="center">
        自己用的一些 <a href="https://sing-box.sagernet.org/zh/">sing-box</a> 配置文件模板, 支持 <a href="https://github.com/Toperlock/sing-box-subscribe">Toperlock/sing-box-subscribe</a> 远程调用。
    </p>
</p><br />

- [1. 使用示例](#1-使用示例)
- [2. 分类](#2-分类)
  - [2.1 入站方式](#21-入站方式)
    - [2.1.1 tun 入站](#211-tun-入站)
    - [2.1.2 mixed 入站](#212-mixed-入站)
  - [2.2 DNS 协议](#22-dns-协议)
  - [2.3 DNS 服务商](#23-dns-服务商)
  - [2.4 CDN](#24-cdn)
- [3 模板推荐](#3-模板推荐)
  - [3.1 Linux 和 Windows](#31-linux-和-windows)
  - [3.2 Android 和 Apple](#32-android-和-apple)
- [4. 注意事项](#4-注意事项)
  - [4.1 下载进程分流](#41-下载进程分流)
  - [4.2 TUN 模式的问题](#42-tun-模式的问题)
  - [4.3 三网线路分流](#43-三网线路分流)

## 1. 使用示例

```bash
#!/bin/bash

url_gene="https://example.com"  # 生成配置的后端地址
url_sub="https://example.com"   # 来自机场的订阅链接
url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-template/normal/tun/doh/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
url_dl="$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"

curl -L -o config.json "$url_dl"
```

总之拼接出自己的 `$url_dl` 即可。更多参数信息, 阅读 https://github.com/Toperlock/sing-box-subscribe/blob/main/instructions/README.md

## 2. 分类

文件按照 "入站方式 → DNS 协议 → DNS 服务商 → CDN" 进行层级划分。

<details>
<summary>目录结构参考</summary>

```
├── mixed
│   ├── doh
│   │   └── 8.8.8.8
│   │       ├── ghproxy.net
│   │       │   └── config.json
│   │       └── testingcf.jsdelivr.net
│   │           └── config.json
│   └── dot
│       └── 8.8.8.8
│           ├── ghproxy.net
│           │   └── config.json
│           └── testingcf.jsdelivr.net
│               └── config.json
└── tun
    ├── doh
    │   └── 8.8.8.8
    │       ├── ghproxy.net
    │       │   └── config.json
    │       └── testingcf.jsdelivr.net
    │           └── config.json
    └── dot
        └── 8.8.8.8
            ├── ghproxy.net
            │   └── config.json
            └── testingcf.jsdelivr.net
                └── config.json
```

</details>

### 2.1 入站方式

#### 2.1.1 tun 入站

```json
"inbounds": [
  {
    "type": "tun",
    "inet4_address": "172.19.0.1/30",
    "inet6_address": "fdfe:dcba:9876::1/126",
    "auto_route": true,
    "strict_route": true,
    "stack": "mixed",
    "exclude_package": ["com.android.captiveportallogin"],
    "platform": {
      "http_proxy": {
        "enabled": true,
        "server": "127.0.0.1",
        "server_port": 7890
      }
    },
    "sniff": true
  },
  {
    "type": "mixed",
    "listen": "127.0.0.1",
    "listen_port": 7890,
    "sniff": true
  }
],
```

#### 2.1.2 mixed 入站

```json
"inbounds": [
  {
    "type": "mixed",
    "listen": "127.0.0.1",
    "listen_port": 7890,
    "sniff": true
  }
],
```

### 2.2 DNS 协议

DNS 协议只用 `DNS over TLS` 或 `DNS over HTTPS`, 更多 DNS 协议与格式参考 [sing-box](https://sing-box.sagernet.org/zh/configuration/dns/server/#address) 文档。

### 2.3 DNS 服务商

所有文件的 `国内DNS` 都使用 `223.5.5.5`; 
`国外DNS` 使用 `1.1.1.1`, `1.0.0.1`, `8.8.8.8`, `8.8.4.4`, `9.9.9.9`, `149.112.112.112` 中的一个。
更多 DNS 服务商参考 [公共DNS](https://senzyo.net/2022-22/)。

```json
"dns": {
  "servers": [
    {
      "tag": "国际DNS",
      "address": "https://8.8.8.8/dns-query",
      // "address": "tls://8.8.8.8",
      "strategy": "prefer_ipv4",
      "detour": "🚀 默认出站"
    },
    {
      "tag": "国内DNS",
      "address": "https://223.5.5.5/dns-query",
      // "address": "tls://223.5.5.5",
      "strategy": "prefer_ipv4",
      "detour": "🐢 直连"
    },
...
  ],
...
},
```

### 2.4 CDN

仅影响客户端下载规则集的速度。

```json
"route": {
...
  "rule_set": [
...
    {
      "tag": "download-process",
      "type": "remote",
      "format": "binary",
      "url": "https://ghproxy.net/https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.srs",
      "download_detour": "🐢 直连",
      "update_interval": "3d"
    },
...
  ]
}
```

规则源地址举例: 

```
https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.srs
```

CDN 格式列举:

```
https://fastly.jsdelivr.net/gh/senzyo/sing-box-rules@master/download-process.srs
```

```
https://gcore.jsdelivr.net/gh/senzyo/sing-box-rules@master/download-process.srs
```

```
https://testingcf.jsdelivr.net/gh/senzyo/sing-box-rules@master/download-process.srs
```

```
https://mirror.ghproxy.com/https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.srs
```

```
https://ghproxy.net/https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.srs
```

```
https://ghproxy.senzyo.net/https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.srs
```

可自行替换模板中使用的 CDN, 替换前推荐对这些 CDN 的域名进行 [网站测速](https://www.itdog.cn/http/)。不推荐 `cdn.jsdelivr.net`。

## 3 模板推荐

### 3.1 Linux 和 Windows

推荐使用入站方式为 `tun` 的模板:

```
https://raw.githubusercontent.com/senzyo/sing-box-template/normal/tun/doh/8.8.8.8/ghproxy.net/config.json
```

或者使用入站方式为 `mixed` 的模板:

```
https://raw.githubusercontent.com/senzyo/sing-box-template/normal/mixed/doh/8.8.8.8/ghproxy.net/config.json
```

### 3.2 Android 和 Apple

只推荐使用入站方式为 `tun` 的模板:

```
https://raw.githubusercontent.com/senzyo/sing-box-template/normal/tun/doh/8.8.8.8/ghproxy.net/config.json
```

## 4. 注意事项

### 4.1 下载进程分流

由于无法准确分流 BitTorrent 流量, 干脆匹配 [下载软件的进程](https://raw.githubusercontent.com/senzyo/sing-box-rules/master/download-process.json) 来一刀切。使用 Bittorrent 方式下载时, 手动切换 `📥 下载` 分组的策略, 改用 `🐢 直连`。

### 4.2 TUN 模式的问题

根据 [issue#883](https://github.com/SagerNet/sing-box/issues/883), 如果在 TUN 模式下无法使用进行 SSH 访问, , 需要关闭严格路由:

```json
"inbounds": [
  {
    "type": "tun",
...
    "strict_route": false,
...
  },
...
],
```

如果关闭了严格路由, Linux 平台在 TUN 模式下还是无法使用 IPv6 进行 SSH 访问, 根据 [issue#458](https://github.com/SagerNet/sing-box/issues/458) 得知:

> 由于技术限制, 在 Linux 平台中 tun 的自动路由会阻止 IPv6 入站连接, 您可以选择手动配置路由。

### 4.3 三网线路分流

我使用的机场对节点这样命名: `XXX-电信/联通`, `XXX-联通/移动`, `XXX-电信/移动`, `XXX-全网优化`。

所以我对节点这样过滤: 

```json
"outbounds": [
...
  {
    "tag": "⚡ 日韩新-移动",
    "type": "urltest",
    "outbounds": ["{all}"],
    "interrupt_exist_connections": true,
    "filter": [
      {
        "action": "include",
        "keywords": [
          "🇯🇵|日本|JP|Japan|🇰🇷|韩国|KR|South Korea|🇸🇬|新加坡|SG|Singapore"
        ]
      },
      {
        "action": "exclude",
        "keywords": ["电信/联通"]
      }
    ],
    "url": "https://www.gstatic.com/generate_204",
    "interval": "10m",
    "tolerance": 0
  },
...
],
```

如果你的节点命名规则与此不同, 请只使用 `normal` 分支的模板, 不要使用其他模板。