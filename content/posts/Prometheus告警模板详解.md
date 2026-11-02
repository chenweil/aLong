---
title: Prometheus告警模板详解
date: 2019-12-04
tags: [alertmanager, prometheus]
type: "post"
description: "之前配置告警之后,可以发送告警信息.但对于数据具体的结构信息,在模板中数据读取都比较懵.原因是不太清除警报都提供了哪些数据,除了我们设置的信息,还有没有其他的信息."
lastmod: 2019-12-04
---
## 目的

之前配置告警之后,可以发送告警信息.但对于数据具体的结构信息,在模板中数据读取都比较懵.原因是不太清除警报都提供了哪些数据,除了我们设置的信息,还有没有其他的信息.



### 告警数据结构

[官方docs](https://prometheus.io/docs/alerting/notifications/#alert)

推送数据结构:
| 名称 | 类型 | 解释 | 备注 |
| --- | --- | --- | --- |
| Receiver | string | 定义将通知发送到的接收者的名称(slack，电子邮件等) | 接收者名称 |
| Status | string | 如果至少一个警报正在触发，则定义为触发，否则解决(Firing/Resolved) | 通过状态可以知道,是触发警报,还是警报已经恢复.便于区分状态.只有这两个状态,要么触发,要么恢复. |
| Alerts | Alert | 该组中所有警报对象的列表(请参见下文) | 警报提供的数据都在此列表中.列表下面会详细解读 |
| GroupLabels | KV | 这些警报按标签分组 | 键值对数据,里面是我们在alertManager中配置分组(group_by)的数据 |
| CommonLabels | KV | 所有警报共有的标签 | 很好理解,所有警报都共同有的标签. |
| CommonAnnotations | KV | 所有警报的通用注释集。用于获取有关警报的更多其他信息字符串 | 通用的注释(Annotations) |
| ExternalURL | string | 反向链接到发送通知的Alertmanager | 此连接,点击会跳转到AlertManager.警报管理地址. |


### Alerts数据
| 名称 | 类型 | 解释 | 备注 |
| --- | --- | --- | --- |
| Status | string | 定义警报是否已解决或当前正在触发 | 警报状态(Firing/Resolved) |
| Labels | KV | 警报上要贴的一组标签 | 警报设置中附加的标签 |
| Annotations | KV | 警报的一组注释 | 顾名思义 |
| StartsAt | time.Time | 警报开始触发的时间。如果省略，则当前时间由Alertmanager分配 | 警报触发时间 |
| EndsAt | time.Time | 仅在知道警报的结束时间时设置。否则设置为自收到最后一个警报以来的时间的可配置超时时间 | 警报恢复时间(这个时间我不太清除到底是不是正确的,配置中也会有恢复超时时间,如果超时了,应该也会被认为恢复吧) |
| GeneratorURL | string | 标识此警报的原因实体的反向链接 | 跳转警报图形数据的连接 |

### KV数据的处理方式

KV结构很简单,键值对.通过键获取对应的值.下面提供了一些方法处理这种结构的一些方法.
| 方法名 | 参数 | 返回值 | 解释 | 说明 |
| --- | --- | --- | --- | --- |
| SortedPairs | - | 对（键/值字符串对的列表） | 返回键/值对的排序列表 | 通过此方法可以获得排序后的键值对列表数据 |
| Remove | []string | KV | 返回没有给定键的键/值映射的副本 | 当我们不希望KV数据种出现某些字段,通过此方法可以得到 |
| Names | - | []string | 返回LabelSet中标签名称的名称 | 通过此方法可以获得KV数据中所有键 |
| Values | - | []string | 返回LabelSet中值的列表 | 通过此方法可以获得KV数据中所有值 |

### 字符串相关方法

警报提供的数据是通过GO模板解析的,GO模板的功能通过[GO模板文档](https://golang.org/pkg/text/template/#hdr-Functions)可以了解.

下面提供了一些处理字符串的方法:

![](https://t1.picb.cc/uploads/2019/12/05/kngppe.png)



### 微信通知的DEMO

![](https://t1.picb.cc/uploads/2019/12/05/kngxOt.png)

上图中是微信接受的通知,下面展示通知模板的代码.

```tmpl
{{- define "_alert_list" -}}
{{- range .Alerts.Firing -}}
-----------------------
告警类型：{{ .Labels.alertname }}
告警主题: {{ .Annotations.summary }}
告警详情: {{ .Annotations.description }}
触发时间: {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
{{ end -}}
---------结束-----------------
{{- end -}}

{{- define "_resolve_list" -}}
{{- range .Alerts.Resolved -}}
**************************
告警类型：{{ .Labels.alertname }}
告警主题: {{ .Annotations.summary }}
告警详情: {{ .Annotations.description }}
触发时间: {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
恢复时间: {{ (.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
{{ end -}}
************结束*****************
{{- end -}}


{{- define "wechat.message" -}}
{{- if and (gt (len .Alerts.Firing) 0) (gt (len .Alerts.Resolved) 0) -}}
告警数量:{{.Alerts.Firing | len}}
告警设备:{{ .GroupLabels.server}}
{{ template "_alert_list" . }}
====================================
告警恢复:{{len .Alerts.Resolved}}
告警设备:{{ .GroupLabels.server}}
{{ template "_resolve_list" . }}
{{- else -}}
  {{- if gt (len .Alerts.Firing) 0 -}}
告警数量:{{.Alerts.Firing | len}}
告警设备:{{ .GroupLabels.server}}  
{{ template "_alert_list" . }}
  {{- end -}}
  {{- if gt (len .Alerts.Resolved) 0 -}}
告警恢复:{{len .Alerts.Resolved}}
告警设备:{{ .GroupLabels.server}}
{{ template "_resolve_list" . }}
  {{- end -}}
{{- end -}}
{{- end -}}
```

**其中告警设备 server 这个标签是自定义的.如果没有可以删除此行或根据自己标签定义.**


