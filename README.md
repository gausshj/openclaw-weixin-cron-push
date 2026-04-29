# weixin-cron-push

OpenClaw Skill — 通过 Cron 定时任务向微信用户主动推送消息。

## 功能

- 一次性提醒（`at`）、每日定时推送（`cron`）、固定间隔循环（`every`）
- 自动路由：Cron 的 systemEvent 注入 main session，通过 sessionKey 投递到对应微信用户
- 支持多微信账号隔离
- 可通过 CLI 或 AI 对话创建任务

## 前提

- 已安装并启用 `openclaw-weixin` 插件
- 至少一个微信账号已扫码登录
- Gateway 正在运行
- 目标用户通过微信给机器人发过至少一条消息

## 安装

将本仓库作为 skill 添加到 OpenClaw：

```bash
openclaw skill add https://github.com/gausshj/openclaw-weixin-cron-push
```

## 使用示例

```bash
# 一次性提醒
openclaw cron add \
  --name "开会提醒" \
  --at "2026-04-30T09:00:00+08:00" \
  --system-event "提醒：10点有产品评审会议" \
  --session main \
  --delete-after-run

# 每日天气推送
openclaw cron add \
  --name "每日天气" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --system-event "提醒：查询今天天气并发送给用户。" \
  --session main
```

也可以直接对 AI 说"明天早上8点提醒我开会"，自动创建任务。

## 文件说明

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Skill 定义文件，包含触发条件和使用文档 |
| `references/weixin-api.md` | 微信 API 协议细节（待补充） |
| `LICENSE` | MIT License |

## License

MIT
