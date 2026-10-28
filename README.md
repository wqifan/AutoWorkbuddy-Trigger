# AutoWorkbuddy-Trigger

这是一个**公开**仓库，只干一件事：每天定时触发私有仓库 `wqifan/AutoWorkbuddy`
里的「Buddy 加油站 · 每日签到 + 派猫猫」工作流。

## 为什么要单独开一个公开仓库

GitHub 的定时任务（`on: schedule`）在**免费个人账号 + 私有仓库**的组合下
实际上不会触发 —— 官方文档没写清楚，但社区里 2026 年有大量确认：

> Scheduled workflows only run on private repos if you have a GitHub Pro plan
> or if the repository is public.
> —— https://github.com/orgs/community/discussions/210422

私有仓库 `AutoWorkbuddy` 实测连续 12 次 cron 请求全部未投递，而配置逐项核对无误
（默认分支 main、workflow `state=active`、Actions `enabled=all`、cron 语法合法、
远端原文逐行确认）。改用公开仓库后 cron 才会被投递。

所以架构是：**公开仓库当闹钟，私有仓库当工人**。

```
┌─────────────────────────┐         ┌──────────────────────────────┐
│ AutoWorkbuddy-Trigger   │  POST   │ AutoWorkbuddy（私有）        │
│ 【公开】cron 每小时 :17  │ ──────► │ daily.yml                    │
│ 用 TRIGGER_PAT 鉴权      │ dispatch│ → 签到 + 派猫 + 发邮件        │
└─────────────────────────┘         └──────────────────────────────┘
```

## 文件说明

| 文件 | 作用 |
|---|---|
| `.github/workflows/trigger.yml` | cron 触发器，每小时第 17 分钟调私有仓库的 `workflow_dispatch` |
| `secrets.TRIGGER_PAT` | fine-grained PAT，仅 `AutoWorkbuddy` + `Actions: Read and write` |

## 安全边界

`TRIGGER_PAT` 被限死成：

- Repository access：Only select repositories → `AutoWorkbuddy`
- Permissions：Actions = Read and write，其余一律 No access

它读不到代码、看不到 Secret、改不了别的东西。就算这个仓库是公开的，
Secret 依然是加密存储的，不会出现在任何日志里。

## 为什么每小时跑一次

即使公开仓库，GitHub 的 schedule 仍是 best-effort（官方明说会延迟、
负载高时可能整批丢弃）。每小时给 24 次机会，命中一次即可。

私有仓库那边是幂等的，不会因此多签到、多派猫或多发邮件：
- 签到由服务端保证只算一次（`today_checked_in` / `code=10001`）
- 派猫用「上一趟 `depart_at` 是否落在今天（北京时间）」判断
- 只有「真的做了事」的那一轮才推送，一天最多一封

## 出问题时看哪里

1. 本仓库 Actions 页 → 看有没有 `event=schedule` 的运行
   - 一条都没有 → GitHub 还没识别这个新 cron，等 15~60 分钟；或平台侧丢弃
   - 有运行但失败 → 看日志，`HTTP 403/404` 说明 `TRIGGER_PAT` 过期或权限不对
2. 私有仓库 Actions 页 → 确认 `workflow_dispatch` 是否被成功触发
3. 私有仓库那侧还有一道本机兜底（Windows 计划任务「Buddy加油站-每日签到」），
   电脑开机时会先查后补，所以两边不会重复干活
