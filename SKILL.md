---
name: weixin-cron-push
description: "通过 OpenClaw Cron 定时任务向微信用户主动推送消息（天气、提醒、新闻摘要等）。当用户需要：设置定时提醒、每日推送、定时通知、微信主动发消息、cron 推送到微信时触发。支持一次性（at）和循环（cron/every）任务，多微信账号场景下自动路由到正确的用户。"
---

# 微信 Cron 推送

通过 Cron 定时任务实现微信主动推送。

## 原理

```
Cron systemEvent → 注入 main session → sessionKey 路由到微信 → 用户手机收到消息
```

- 用户通过微信发过消息后，main session 自动绑定该微信用户的 sessionKey
- Cron 的 systemEvent 注入 main session，回复自动发到微信
- 与当前对话来自哪个渠道（网页/微信）无关

## 前提

- 微信插件 `openclaw-weixin` 已安装启用
- 至少一个微信账号已扫码登录
- Gateway 正在运行
- 用户通过微信给机器人发过至少一条消息（建立 sessionKey 绑定）

## 创建任务

### CLI 方式

```bash
# 一次性提醒（at）
openclaw cron add \
  --name "提醒标题" \
  --at "2026-04-29T10:00:00Z" \
  --system-event "提醒：告诉用户 XXX" \
  --session main \
  --delete-after-run

# 每日定时推送（cron 表达式 + 时区）
openclaw cron add \
  --name "每日天气" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --system-event "提醒：查询今天天气并发送给用户。" \
  --session main

# 循环间隔（every）
openclaw cron add \
  --name "每小时间隔" \
  --every "1h" \
  --system-event "提醒：执行 XXX 检查" \
  --session main
```

### AI 对话方式

用户直接说"明天早上8点提醒我开会"，AI 自动创建任务。

## 关键 CLI 参数

| 参数 | 说明 |
|------|------|
| `--at <ISO时间>` | 一次性，指定时间触发 |
| `--cron <表达式>` | Cron 表达式循环 |
| `--every <间隔>` | 固定间隔循环（如 10m, 1h） |
| `--tz <时区>` | 时区（如 Asia/Shanghai） |
| `--system-event <文本>` | systemEvent 内容，AI 处理后回复用户 |
| `--session main` | 注入主会话（微信推送必须） |
| `--delete-after-run` | 触发后自动删除 |
| `--name <名称>` | 任务名称 |
| `--account <id>` | 多账号时指定微信账号 |

## 注意事项

- **一条就够了**：不需要重复发送，正常延迟几秒到1分钟
- **session 绑定**：用户必须先通过微信给机器人发过消息
- **多账号隔离**：多个微信登录时建议 `openclaw config set session.dmScope per-account-channel-peer`
- **只支持文本**：systemEvent 传递文本，如需图片则在 payload 中指示 AI 调用图片工具

## 管理命令

```bash
openclaw cron list                          # 查看任务
openclaw cron delete <ID>                   # 删除任务
openclaw cron run <ID>                      # 立即触发
openclaw cron list --include-disabled       # 含已禁用
```

## 多账号场景

每个微信用户有独立 sessionKey，Cron 任务路由到绑定用户：

1. 扫码登录：`openclaw channels login --channel openclaw-weixin`
2. 该微信号给机器人发一条消息（建立绑定）
3. 创建 cron 任务时指定 `--account` 或让 AI 自动使用当前会话绑定

## 详细参考

完整的 API 协议、消息结构、CDN 上传等细节见 [references/weixin-api.md](references/weixin-api.md)。
