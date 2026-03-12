# f-message-forwarder

来自于channel的消息转发

## 概述

whatsapp/飞书等channels有消息到达时，根据消息内容中的@标识，将消息转发给对应的agent进行处理，并独立返回每个agent的回复。
适用于：
(1) 用户 @特定agent 向单个agent转发消息
(2) 用户 @all 向多个agent转发消息

## 前置配置

使用此 skill 前，必须在 `openclaw.json` 中为 _[agent名]_ 添加 `subagents.allowAgents` 配置：

```json
{
  "agents": {
    "list": [
      {
        "id": "_[agent名]_",
        "subagents": {
          "allowAgents": ["main", "coding", "trading", "xuqiu", "pr", "spider"]
        }
      }
    ]
  }
}
```

配置路径: `agents.list[_[agent名]_].subagents.allowAgents`

## 核心规则

### 规则1

用户在消息中 @某个agent → 转发给对应的 agent

```
channel发送: @main 请帮我写个总结
期望输出: 这是main agent的回复
```

### 规则2

用户 @多个agent → 转发给所有被@的 agent

```
channel发送: @main @pr 请帮我写个总结
期望输出: 这是main agent的回复
期望输出: 这是pr agent的回复
```

### 规则3

用户 @all → 转发给所有 agent，排除当前 _[agent名]_ agent

```
channel发送: @all 请帮我写个总结
期望输出: 这是main agent的回复
期望输出: 这是pr agent的回复
期望输出: 这是coding agent的回复
（回复来自所有agent，排除当前agent）
```

### 规则4

@all和@特定用户混用时，不要重复转发给被@的用户

```
channel发送: @all @main 请帮我写个总结
期望输出: 这是main agent的回复（此处只有一条main的回复）
期望输出: 这是pr agent的回复
期望输出: 这是coding agent的回复
（回复来自所有agent，排除当前agent）
```

### 规则5

_[agent名]_ 在回复消息前加上 **[agent名]** 前缀作为标识

```
channel发送: @main 请帮我写个总结
期望输出: [main]: 这是main agent的回复
```

```
channel发送: @all 请帮我写个总结
期望输出: [main]: 这是main agent的回复
期望输出: [pr]: 这是pr agent的回复
期望输出: [coding]: 这是coding agent的回复
（回复来自所有agent，排除当前agent）
```

## 注意事项

- 每个 agent 单独回复，互不关联
- 转发过程中如果达到最大并发，就排队等待，不要丢失消息
- 不输出任何过程消息：
  - 无转发提示
  - 无"正在转发给xxx"
  - 无完成确认
