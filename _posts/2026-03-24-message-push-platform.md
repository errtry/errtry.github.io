---
layout: post
title: 从零搭建一个消息推送平台：架构设计与实现
categories: [Project]
description: 详细讲解如何从零设计并实现一个支持多渠道的消息推送平台
keywords: 消息推送, 消息队列, RabbitMQ, Go, 微服务
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇实战项目教程，手把手教你搭建消息推送平台

## 项目背景

为什么要自己搭推送平台？

| 场景 | 需求 |
|------|------|
| 系统告警 | 服务器挂了，要第一时间通知 |
| 用户通知 | 订单状态变更、发货提醒 |
| 营销推送 | 活动通知、优惠券发放 |
| 内部协作 | 工单处理、审批流通知 |

买第三方的贵，自己搭的灵活。

---

## 一、整体架构

```
                    ┌─────────────────┐
                    │   Web 管理后台   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    API 网关     │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
┌────────▼────────┐ ┌────────▼────────┐ ┌────────▼────────┐
│   邮件发送服务   │ │   短信发送服务  │ │  企业微信服务    │
│   (SMTP)        │ │   (第三方API)   │ │   (Wecom MCP)   │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### 技术选型

| 组件 | 选型 | 原因 |
|------|------|------|
| 语言 | Go | 高并发、部署简单 |
| 数据库 | MySQL | 存储模板和发送记录 |
| 消息队列 | Redis Stream | 轻量、支持持久化 |
| API | Gin | 轻量高性能 |
| 配置 | Viper | 配置管理 |

---

## 二、数据库设计

```sql
CREATE TABLE channels (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL COMMENT '渠道名称',
    code VARCHAR(20) NOT NULL UNIQUE COMMENT '渠道编码',
    config JSON COMMENT '渠道配置',
    status TINYINT DEFAULT 1 COMMENT '状态 0禁用 1启用',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE templates (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    channel_id BIGINT NOT NULL COMMENT '渠道ID',
    code VARCHAR(50) NOT NULL COMMENT '模板编码',
    name VARCHAR(100) NOT NULL COMMENT '模板名称',
    subject VARCHAR(255) COMMENT '标题模板',
    content TEXT NOT NULL COMMENT '内容模板，支持变量{{name}}',
    status TINYINT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_channel_code (channel_id, code)
);

CREATE TABLE messages (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    channel_id BIGINT NOT NULL,
    template_id BIGINT,
    send_type TINYINT NOT NULL COMMENT '1即时 2定时',
    send_time TIMESTAMP NULL COMMENT '定时发送时间',
    receiver VARCHAR(255) NOT NULL COMMENT '接收人',
    subject VARCHAR(255) COMMENT '标题',
    content TEXT COMMENT '实际发送内容',
    status TINYINT DEFAULT 0 COMMENT '0待发送 1发送中 2成功 3失败',
    send_at TIMESTAMP NULL COMMENT '实际发送时间',
    error_msg TEXT COMMENT '错误信息',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_status (status),
    INDEX idx_send_time (send_time)
);

CREATE TABLE send_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    message_id BIGINT NOT NULL,
    channel_code VARCHAR(20) NOT NULL,
    request_data TEXT COMMENT '请求数据',
    response_data TEXT COMMENT '响应数据',
    cost_ms INT COMMENT '耗时毫秒',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 三、核心代码实现

### 项目结构

```
austin/
├── cmd/
│   ├── api/           # API 服务
│   │   └── main.go
│   └── worker/       # 消费者服务
│       └── main.go
├── internal/
│   ├── config/        # 配置
│   ├── handler/      # HTTP 处理
│   ├── model/        # 数据模型
│   ├── service/     # 业务逻辑
│   ├── repository/  # 数据访问
│   └── queue/       # 队列操作
├── pkg/
│   ├── channel/     # 渠道实现
│   │   ├── mail.go
│   │   ├── sms.go
│   │   ├── wecom.go
│   │   └── interface.go
│   └── template/    # 模板引擎
└── configs/
    └── config.yaml
```

### 配置管理

```go
// internal/config/config.go
package config

import "github.com/spf13/viper"

type Config struct {
    Database DatabaseConfig `mapstructure:"database"`
    Redis    RedisConfig    `mapstructure:"redis"`
    Server   ServerConfig   `mapstructure:"server"`
    Channels ChannelsConfig  `mapstructure:"channels"`
}

type DatabaseConfig struct {
    Host     string `mapstructure:"host"`
    Port     int    `mapstructure:"port"`
    User     string `mapstructure:"user"`
    Password string `mapstructure:"password"`
    Database string `mapstructure:"database"`
}

type RedisConfig struct {
    Host     string `mapstructure:"host"`
    Port     int    `mapstructure:"port"`
    Password string `mapstructure:"password"`
    DB       int    `mapstructure:"db"`
}

type ServerConfig struct {
    Port int `mapstructure:"port"`
}

func Load(path string) (*Config, error) {
    viper.SetConfigFile(path)
    viper.SetConfigType("yaml")
    
    if err := viper.ReadInConfig(); err != nil {
        return nil, err
    }
    
    var cfg Config
    if err := viper.Unmarshal(&cfg); err != nil {
        return nil, err
    }
    return &cfg, nil
}
```

### 渠道接口

```go
// pkg/channel/interface.go
package channel

// SendRequest 发送请求
type SendRequest struct {
    Receiver string            // 接收人
    Subject  string            // 标题
    Content  string            // 内容
    Extra    map[string]string // 额外参数
}

// SendResponse 发送响应
type SendResponse struct {
    Code    int    // 0成功 其他失败
    Message string // 错误信息
    MsgID   string // 第三方消息ID
}

// Sender 发送者接口
type Sender interface {
    Send(req *SendRequest) *SendResponse
    Name() string // 渠道名称
}

// ChannelFactory 渠道工厂
type ChannelFactory struct {
    senders map[string]Sender
}

func NewChannelFactory() *ChannelFactory {
    return &ChannelFactory{
        senders: make(map[string]Sender),
    }
}

func (f *ChannelFactory) Register(name string, sender Sender) {
    f.senders[name] = sender
}

func (f *ChannelFactory) Get(name string) (Sender, bool) {
    s, ok := f.senders[name]
    return s, ok
}
```

### 邮件渠道实现

```go
// pkg/channel/mail.go
package channel

import (
    "crypto/tls"
    "fmt"
    "net/smtp"
)

type MailConfig struct {
    Host     string
    Port     int
    Username string
    Password string
    From     string
}

type MailSender struct {
    config MailConfig
}

func NewMailSender(config MailConfig) *MailSender {
    return &MailSender{config: config}
}

func (m *MailSender) Name() string {
    return "mail"
}

func (m *MailSender) Send(req *SendRequest) *SendResponse {
    // 构建邮件
    to := []string{req.Receiver}
    msg := buildMailMsg(m.config.From, to, req.Subject, req.Content)
    
    // SMTP 认证
    auth := smtp.PlainAuth("", m.config.Username, m.config.Password, m.config.Host)
    
    // 发送
    addr := fmt.Sprintf("%s:%d", m.config.Host, m.config.Port)
    err := smtp.SendMail(addr, auth, m.config.From, to, []byte(msg))
    
    if err != nil {
        return &SendResponse{
            Code:    1,
            Message: err.Error(),
        }
    }
    
    return &SendResponse{Code: 0, Message: "success"}
}

func buildMailMsg(from string, to []string, subject, body string) string {
    header := make(map[string]string)
    header["From"] = from
    header["To"] = to[0]
    header["Subject"] = subject
    header["MIME-Version"] = "1.0"
    header["Content-Type"] = "text/html; charset=UTF-8"
    
    msg := ""
    for k, v := range header {
        msg += fmt.Sprintf("%s: %s\r\n", k, v)
    }
    msg += "\r\n" + body
    return msg
}

// 如果用 TLS
func (m *MailSender) SendWithTLS(req *SendRequest) *SendResponse {
    // 使用 net/smtp 配合 tls.Dial
    // 完整实现略...
    return &SendResponse{Code: 0, Message: "success"}
}
```

### 消息队列

```go
// internal/queue/queue.go
package queue

import (
    "context"
    "encoding/json"
    "fmt"
    "time"
    
    "github.com/redis/go-redis/v9"
)

type Message struct {
    ID         int64             `json:"id"`
    Channel    string            `json:"channel"`
    Receiver   string            `json:"receiver"`
    Subject    string            `json:"subject"`
    Content    string            `json:"content"`
    Extra      map[string]string `json:"extra"`
    SendTime   time.Time         `json:"send_time"`
}

type Queue struct {
    redis *redis.Client
}

func NewQueue(redis *redis.Client) *Queue {
    return &Queue{redis: redis}
}

const (
    StreamKey   = "austin:messages"
    ConsumerGroup = "austin-workers"
    ConsumerName = "worker-1"
)

// Push 添加消息到队列
func (q *Queue) Push(ctx context.Context, msg *Message) error {
    data, err := json.Marshal(msg)
    if err != nil {
        return err
    }
    
    return q.redis.XAdd(ctx, &redis.XAddArgs{
        Stream: StreamKey,
        Values: map[string]interface{}{
            "data": string(data),
        },
    }).Err()
}

// Consume 消费消息
func (q *Queue) Consume(ctx context.Context, handler func(*Message) error) error {
    // 创建消费者组
    q.redis.XGroupCreateMkStream(ctx, StreamKey, ConsumerGroup, "0")
    
    for {
        // 读取新消息
        result, err := q.redis.XReadGroup(ctx, &redis.XReadGroupArgs{
            Group:    ConsumerGroup,
            Consumer: ConsumerName,
            Streams:  []string{StreamKey, ">"},
            Count:    10,
            Block:    time.Second * 5,
        }).Result()
        
        if err != nil && err != redis.Nil {
            return err
        }
        
        for _, stream := range result {
            for _, msg := range stream.Messages {
                var m Message
                if err := json.Unmarshal([]byte(msg.Values["data"].(string)), &m); err != nil {
                    continue
                }
                
                if err := handler(&m); err != nil {
                    // 失败处理，可以重试或记录
                    fmt.Printf("handle message %d failed: %v\n", m.ID, err)
                }
                
                // 确认消息
                q.redis.XAck(ctx, StreamKey, ConsumerGroup, msg.ID)
            }
        }
    }
}
```

### API 接口

```go
// internal/handler/send.go
package handler

import (
    "net/http"
    "strconv"
    "strings"
    "time"
    
    "github.com/gin-gonic/gin"
    "austin/internal/model"
    "austin/internal/repository"
    "austin/internal/queue"
)

type SendHandler struct {
    repo  *repository.MessageRepository
    queue *queue.Queue
}

func NewSendHandler(repo *repository.MessageRepository, q *queue.Queue) *SendHandler {
    return &SendHandler{repo: repo, queue: q}
}

// SendRequest 发送请求
type SendRequest struct {
    Channel    string            `json:"channel" binding:"required"`
    Receiver   string            `json:"receiver" binding:"required"`
    TemplateID int64             `json:"template_id"`
    Subject    string            `json:"subject"`
    Content    string            `json:"content"`
    Params     map[string]string `json:"params"` // 模板变量
    SendType   int               `json:"send_type"`  // 1即时 2定时
    SendTime   string            `json:"send_time"`  // 定时发送时间
}

func (h *SendHandler) Send(c *gin.Context) {
    var req SendRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"code": 1, "message": err.Error()})
        return
    }
    
    // 解析模板
    if req.TemplateID > 0 && req.Content == "" {
        tmpl, err := h.repo.GetTemplate(req.TemplateID)
        if err != nil {
            c.JSON(http.StatusInternalServerError, gin.H{"code": 1, "message": "模板不存在"})
            return
        }
        req.Subject = renderTemplate(tmpl.Subject, req.Params)
        req.Content = renderTemplate(tmpl.Content, req.Params)
    }
    
    // 构建消息
    msg := &model.Message{
        ChannelID: 1,
        SendType:   req.SendType,
        Receiver:   req.Receiver,
        Subject:    req.Subject,
        Content:    req.Content,
        Status:     0,
        CreatedAt:  time.Now(),
    }
    
    if req.SendType == 2 {
        t, _ := time.ParseInLocation("2006-01-02 15:04:05", req.SendTime, time.Local)
        msg.SendTime = t
    }
    
    // 保存到数据库
    if err := h.repo.CreateMessage(msg); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"code": 1, "message": err.Error()})
        return
    }
    
    // 如果是即时发送，加入队列
    if req.SendType == 1 {
        qmsg := &queue.Message{
            ID:       msg.ID,
            Channel:  req.Channel,
            Receiver: req.Receiver,
            Subject:  req.Subject,
            Content:  req.Content,
            SendTime: time.Now(),
        }
        h.queue.Push(c.Request.Context(), qmsg)
    }
    
    c.JSON(http.StatusOK, gin.H{
        "code":    0,
        "message": "success",
        "data": gin.H{
            "message_id": msg.ID,
        },
    })
}

// 简单的模板渲染 {{name}} -> value
func renderTemplate(template string, params map[string]string) string {
    result := template
    for k, v := range params {
        result = strings.ReplaceAll(result, "{{"+k+"}}", v)
    }
    return result
}
```

---

## 四、部署

### Docker Compose

```yaml
version: '3.8'

services:
  api:
    build: ./cmd/api
    ports:
      - "8080:8080"
    depends_on:
      - mysql
      - redis
    volumes:
      - ./configs:/app/configs
    environment:
      - CONFIG_PATH=/app/configs/config.yaml

  worker:
    build: ./cmd/worker
    depends_on:
      - mysql
      - redis
    volumes:
      - ./configs:/app/configs
    environment:
      - CONFIG_PATH=/app/configs/config.yaml
    deploy:
      replicas: 2  # 多个 worker 消费者

  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: austin
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  mysql_data:
```

### 启动

```bash
# 构建并启动
docker-compose up -d --build

# 查看日志
docker-compose logs -f

# 查看状态
docker-compose ps
```

---

## 五、使用示例

### 发送邮件

```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "mail",
    "receiver": "user@example.com",
    "subject": "订单通知",
    "content": "您的订单 {{order_no}} 已发货",
    "params": {"order_no": "A123456"}
  }'
```

### 发送企业微信

```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "wecom",
    "receiver": "user001",
    "content": "【告警】服务器 CPU 使用率超过 90%"
  }'
```

### 定时发送

```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "mail",
    "receiver": "user@example.com",
    "subject": "定时任务",
    "content": "这是一条定时消息",
    "send_type": 2,
    "send_time": "2024-01-15 10:00:00"
  }'
```

---

## 六、扩展方向

| 方向 | 内容 |
|------|------|
| 短信渠道 | 接入阿里云/腾讯云短信 |
| 钉钉/飞书 | 接入钉钉/飞书机器人 |
| WebSocket | 支持实时推送 |
| 发送限制 | 防刷、频率限制 |
| 模板管理 | 可视化模板编辑 |
| 数据统计 | 发送量、成功率分析 |

---

## 总结

这个推送平台覆盖了：
1. ✅ 多渠道支持（邮件、企业微信）
2. ✅ 消息队列异步处理
3. ✅ 模板变量渲染
4. ✅ 定时发送
5. ✅ 发送日志
6. ✅ Docker 部署

代码结构清晰，可以根据需求扩展其他渠道。

---

*作者：牛马便利店一号店员*
