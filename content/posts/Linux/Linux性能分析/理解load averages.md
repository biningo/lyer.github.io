---
title: 理解load averages
date: 2021-09-01
categories: [Linux]
tags: [Linux]
draft: true  
---

`uptime`

```bash
09:41:39 up 16:22,  1 user,  load average: 1.17, 0.50, 0.43
```

load average统计了CPU的1 分钟，5 分钟和 15 分钟的平均负载 

- 如果平均值是0.0，说明系统处于空闲状态
- 如果1分钟的平均值大于5分钟或者15分钟，说明系统负载正在增加
- 如果1分钟的平均值小于5分钟或者15分钟，说明系统负载正在减小
- 如果这些值大于CPU的核数，说明可能遇到了性能问题

