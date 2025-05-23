# 订单支付时序图示例

## Mermaid代码

```mermaid
sequenceDiagram
    participant U as 用户
    participant FE as 前端
    participant API as API网关
    participant OS as 订单服务
    participant PS as 支付服务
    participant WP as 微信支付
    participant NS as 通知服务
    
    Note over U,NS: 电商订单支付完整流程
    
    U->>FE: 点击立即支付
    FE->>API: 提交订单支付请求
    API->>OS: 创建订单
    
    alt 订单创建成功
        OS->>OS: 验证商品库存和价格
        OS->>OS: 生成订单号
        OS-->>API: 返回订单信息
        
        API->>PS: 发起支付请求
        PS->>PS: 生成支付单号
        PS->>WP: 调用微信支付统一下单
        
        alt 微信下单成功
            WP-->>PS: 返回支付参数
            PS-->>API: 返回支付参数
            API-->>FE: 返回支付参数
            FE->>WP: 调起微信支付
            
            U->>WP: 用户完成支付
            WP->>PS: 支付结果回调
            
            alt 支付成功
                PS->>OS: 更新订单状态为已支付
                OS->>OS: 扣减库存
                OS->>NS: 发送支付成功通知
                PS-->>WP: 确认回调处理成功
                
                par 并行处理
                    NS->>U: 发送支付成功短信
                and
                    NS->>FE: 推送支付成功消息
                and
                    OS->>OS: 触发发货流程
                end
                
                Note over U,NS: 支付成功，订单进入发货状态
                
            else 支付失败
                PS->>OS: 更新订单状态为支付失败
                OS->>OS: 释放锁定库存
                PS-->>WP: 确认回调处理成功
                NS->>U: 发送支付失败通知
                
                Note over U,NS: 支付失败，库存已释放
            end
            
        else 微信下单失败
            WP-->>PS: 返回错误信息
            PS-->>API: 返回支付失败
            API-->>FE: 返回错误信息
            FE-->>U: 提示支付失败，请重试
        end
        
    else 订单创建失败
        OS-->>API: 返回创建失败
        API-->>FE: 返回错误信息
        FE-->>U: 提示订单创建失败
    end
    
    Note over U,NS: 支付流程结束
```

## 使用说明

1. 复制上述Mermaid代码
2. 在 [Mermaid Live Editor](https://mermaid.live/) 中粘贴
3. 即可看到完整的支付流程时序图

## 关键特点

- **完整流程**：覆盖从用户点击支付到支付完成的全流程
- **异常处理**：包含订单创建失败、支付失败等异常情况
- **并行处理**：展示支付成功后的并行通知流程
- **状态管理**：清晰展示订单状态的变化过程

## 适用场景

- 系统设计评审
- 开发团队技术交流
- 测试用例设计
- 问题排查和优化