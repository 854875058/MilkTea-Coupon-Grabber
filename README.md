<div align="center">

# MilkTea Coupon Grabber

**奶茶优惠券自动抢券工具**

*High-concurrency coupon grabber for Chinese milk tea brands with async I/O and reverse-engineered signing*

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python)](https://python.org/)
[![asyncio](https://img.shields.io/badge/asyncio-Concurrent-FFD43B?logo=python)](https://docs.python.org/3/library/asyncio.html)
[![JavaScript](https://img.shields.io/badge/JavaScript-Signing-F7DF1E?logo=javascript)](https://developer.mozilla.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

> **Disclaimer**: 本项目仅供学习交流与技术研究使用，请勿用于商业用途或恶意抢券。使用本工具产生的一切后果由使用者自行承担，相关活动规则以官方为准。

## Overview

奶茶品牌优惠券活动往往在开抢瞬间被秒光，手动操作根本抢不到。核心难点在于：**毫秒级定时精度**、**高并发请求能力**、**签名算法逆向**。

本工具通过逆向 H5 端签名算法，结合 Python asyncio 异步并发（单账号 1000+ 并发请求）、服务器时间同步、代理池轮换，实现毫秒级精准抢券。支持蜜雪冰城、茶百道等主流品牌，多账号同时作战。

```
┌─────────────────────────────────────────────────────────────┐
│                   Server Time Sync                           │
├─────────────────────────────────────────────────────────────┤
│              Millisecond Precision Timer                      │
├──────────────┬──────────────────┬────────────────────────────┤
│  JS Signing  │  Async Request   │  Proxy Pool                │
│  算法逆向     │  Pool (1000+)    │  IP 轮换                   │
├──────────────┴──────────────────┴────────────────────────────┤
│         Multi-Account Token Management                        │
└─────────────────────────────────────────────────────────────┘
```

## Key Features

### Async High Concurrency
基于 Python asyncio + aiohttp，单账号可发起 1000+ 并发请求。多账号场景下并发数线性叠加，最大化抢券成功率。

### Millisecond Precision Timing
通过服务器时间同步 + 延迟补偿，在活动开抢的精确毫秒发起请求。避免本地时钟偏差导致的抢券失败。

### JS Signature Reversal
逆向还原 H5 端签名算法，生成合法请求签名。蜜雪冰城使用 MD5 + 固定盐值，茶百道使用微信小程序 CSESSION 认证。

### Proxy Pool Integration
集成第三方代理池 API，自动轮换 IP 地址，规避频率限制与 IP 封禁。

### Multi-Account Management
支持多账号 Token 批量导入，自动追踪已完成/失效 Token，避免重复请求浪费资源。

## Supported Brands

| Brand | Script | Auth | Concurrency | Proxy |
|-------|--------|------|-------------|-------|
| 蜜雪冰城 (Mixue) | `蜜雪冰城.py` | MD5 签名 (type__1286) | 1000+ async | - |
| 蜜雪冰城 (Full) | `mx.py` / `mx1.py` | MD5 签名 + JS Runtime | 1000+ async | Proxy Pool |
| 茶百道 (ChabaDao) | `茶百道.py` | WeChat CSESSION | 1000+ async | - |
| 茶百道 (Full) | `cbd1.py` | WeChat CSESSION | Multi-thread + async | - |

## Tech Stack

```
Python                            JavaScript                       Infrastructure
─────────────────                 ─────────────────               ─────────────────
asyncio (Event Loop)              mx.js (Mixue Signing)            Proxy Pool API
aiohttp (Async HTTP)              mx1.js (Mixue v2)                Token Management
requests (Sync HTTP)              F3.js (Frontend Crypto)          Server Time Sync
execjs (JS Runtime)               params.js (Param Build)
fake_useragent (UA Pool)
```

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      Token File (multi-account)                   │
│                    token.txt / token1.txt / token2.txt             │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────┐    ┌──────────────┐    ┌────────────────────┐   │
│  │ Server Time │    │  JS Signing  │    │   Proxy Pool       │   │
│  │ Sync        │    │  Engine      │    │   Rotation         │   │
│  │ (ms精度)     │    │  (execjs)    │    │   (IP轮换)         │   │
│  └──────┬──────┘    └──────┬───────┘    └────────┬───────────┘   │
│         └──────────────────┼──────────────────────┘               │
│                            │                                       │
│                   ┌────────▼─────────┐                            │
│                   │  asyncio         │                            │
│                   │  Event Loop      │                            │
│                   │  (1000+ tasks)   │                            │
│                   └────────┬─────────┘                            │
│                            │                                       │
│              ┌─────────────┼─────────────┐                        │
│              │             │             │                         │
│         ┌────▼───┐   ┌────▼───┐   ┌────▼───┐                    │
│         │ Req #1 │   │ Req #2 │   │Req #N  │  ... (concurrent)  │
│         └────┬───┘   └────┬───┘   └────┬───┘                    │
│              └─────────────┼─────────────┘                        │
│                            │                                       │
│                   ┌────────▼─────────┐                            │
│                   │  Result Tracker  │                            │
│                   │  completed/      │                            │
│                   │  invalid tokens  │                            │
│                   └──────────────────┘                            │
└──────────────────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# 1. Clone
git clone https://github.com/854875058/MilkTea-Coupon-Grabber.git
cd MilkTea-Coupon-Grabber

# 2. Install dependencies
pip install aiohttp requests PyExecJS fake_useragent

# 3. Prepare tokens
# Edit token.txt: one token per line

# 4. Run (Mixue simple version)
python 蜜雪冰城.py

# 5. Run (Mixue full version with proxy)
python mx.py

# 6. Run (ChabaDao)
python 茶百道.py
```

## Project Structure

```
MilkTea-Coupon-Grabber/
├── 蜜雪冰城.py                # Mixue simple async grabber
├── mx.py                      # Mixue full version (proxy + timing)
├── mx1.py                     # Mixue alternative version
├── 茶百道.py                  # ChabaDao async grabber
├── cbd1.py                    # ChabaDao multi-thread + async
├── mx.js                      # Mixue signing algorithm
├── mx1.js                     # Mixue signing v2
├── F3.js                      # Frontend encryption analysis
├── test.py                    # Test harness
├── token.txt                  # Token list (one per line)
├── completed_tokens.txt       # Successfully used tokens
└── invalid_tokens.txt         # Expired/invalid tokens
```

## Usage

| Script | Brand | Features |
|--------|-------|----------|
| `python 蜜雪冰城.py` | 蜜雪冰城 | 基础异步抢券 |
| `python mx.py` | 蜜雪冰城 | 代理池 + 定时 + 多账号 |
| `python mx1.py` | 蜜雪冰城 | 代理池 + 签名 v2 |
| `python 茶百道.py` | 茶百道 | 基础异步抢券 |
| `python cbd1.py` | 茶百道 | 多线程 + 异步混合 |

## Disclaimer

本项目仅供学习交流与安全研究使用。请遵守相关平台的使用条款与法律法规，不得将本工具用于任何商业目的或恶意行为。因使用本工具产生的一切法律责任由使用者自行承担。

## License

MIT
