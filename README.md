# AutoWorkbuddy-Trigger

这是一个**公开**仓库，只干一件事：每天定时触发私有仓库 `wqifan/AutoWorkbuddy`
里的「Buddy 加油站 · 每日签到 + 派猫猫」工作流。

## 为什么要单独开一个公开仓库

GitHub 的定时任务（`on: schedule`）在**免费个人账号 + 私有仓库**的组合下
实际上不会触发 —— 官方文档没写清楚，但社区里 2026 年有大量确认：

> Scheduled workflows only run on private repos if you have a GitHub Pro plan
> or if the repository is public.

私有仓库 `AutoWorkbuddy` 实测连续 12 次 cron 请求全部未投递（配置逐项核对无误：
默认分支 main、workflow state=active、Actions enabled=all、cron 语法合法）。
改用公开仓库后 cron 才会被投递。

所以：**公开仓库当闹钟，私有仓库当工人**。

## 安全边界

`TRIGGER_PAT` 是一条 fine-grained PAT，权限被限死成：

- Repository access：Only select repositories → `AutoWorkbuddy`
- Permissions：Actions = Read and write，其余一律 No access

它读不到代码、看不到 Secret、改不了别的东西。即使这个仓库是公开的，
Secret 依然是加密存储的，不会出现在任何日志里。
