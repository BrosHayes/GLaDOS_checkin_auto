# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

此文件为 Claude Code (claude.ai/code) 在本仓库中工作提供指导。

---

## Project Overview / 项目概述

GLaDOS Auto Check-in - Automated daily check-in script for GLaDOS VPN service. Performs automatic check-ins to maintain VPN service days and can exchange points for additional days when account is about to expire.

GLaDOS 自动签到 - GLaDOS VPN 服务的自动每日签到脚本。自动执行签到以维持 VPN 服务天数，并可在账户即将到期时用积分兑换额外天数。

## Running the Scripts / 运行脚本

```bash
# Main script (requires GLADOS_COOKIE environment variable)
# 主脚本（需要 GLADOS_COOKIE 环境变量）
python3 glados.py

# Qinglong panel version / 青龙面板版本
python3 glados_Qinglong.py
```

**Required dependency / 必需依赖:** `pip install requests`

## Environment Variables / 环境变量

| Variable / 变量 | Required / 必需 | Description / 描述 |
|----------------|-----------------|-------------------|
| `GLADOS_COOKIE` | ✅ | Account cookie(s), multiple separated by `&` / 账户 Cookie，多账户用 `&` 分隔 |
| `PUSHPLUS_TOKEN` | ❌ | PushPlus notification token / PushPlus 推送通知令牌 |
| `GLADOS_BASE_URLS` | ❌ | Custom base URLs (comma/semicolon separated) / 自定义基础 URL（逗号/分号分隔） |
| `AUTO_EXCHANGE` | ❌ | Enable/disable auto-exchange (default: enabled) / 启用/禁用自动兑换（默认：启用） |
| `GLADOS_EXCHANGE_URLS` | ❌ | Custom exchange API endpoints / 自定义兑换 API 端点 |

## Architecture / 架构

**Two main scripts / 两个主要脚本:**
- `glados.py` - Primary script for GitHub Actions / 主脚本，用于 GitHub Actions 部署
- `glados_Qinglong.py` - Simplified version for Qinglong panel / 简化版本，用于青龙面板

**Execution flow / 执行流程:**
1. Load cookies from environment variables / 从环境变量加载 Cookie
2. Resolve working GLaDOS base URL / 解析可用的 GLaDOS 基础 URL
3. For each account: check-in → query status → extract points/days → auto-exchange if 1 day left / 对每个账户：签到 → 查询状态 → 提取积分/天数 → 剩余1天时自动兑换
4. Send results via PushPlus notification / 通过 PushPlus 发送推送通知

**GLaDOS API endpoints / API 端点:**
- `/api/user/checkin` - Daily check-in (POST) / 每日签到
- `/api/user/status` - Account status (GET) / 账户状态
- `/api/user/points` - Points balance (GET) / 积分余额
- `/api/user/points/exchange` - Exchange points for days (POST) / 积分兑换天数

**Exchange tiers (when only 1 day remaining) / 兑换档位（仅剩1天时）:**
| Points / 积分 | Days / 天数 | planType |
|--------------|------------|----------|
| 500+ | 100 | 2 |
| 200+ | 30 | 1 |
| 100+ | 10 | 0 |

## GitHub Actions

Workflow: `.github/workflows/runGladosAction.yml`
- Scheduled daily at 17:00 UTC / 每天 UTC 17:00 执行（北京时间凌晨 1:00）
- Manual trigger via workflow_dispatch / 支持手动触发
- Triggers on repository star / Star 仓库时触发
- Uses repository secrets: `GLADOS_COOKIE`, `PUSHPLUS_TOKEN` / 使用仓库密钥配置
