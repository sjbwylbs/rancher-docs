## Common Resource Fields
---

在API中,最常见和只读的资源域并不在资源视图中显示。为了让用户更容易浏览资源域列表，我们将这些资源域从列表中移除。

域 | 类型 |  注释
---|---|---|---|---|---
accountId | [account]({{site.baseurl}}/rancher/api/api-resources/account/) | 相关联账号的唯一识别符。
created[TS] | date | 资源创建的起始日期/时间。
id | string | 资源的唯一识别符。
kind | string | 资源更具体的类型划分。
removeTime | date | 资源被移除的时间, 如果资源没有被移除,则该值将为null。
removed | bool | 如果资源已经被移除,则为真。已移除的资源不会回到collection列表,但能单独通过ID来检索。
transitioning | enum |  资源转换的状态: `yes`, `no`,或者`error`。
transitioningMessage | string |  转换过程中出现的信息，或者产生的错误。
transitioningProgress | int |  转换期间的进度估算，范围是0-100。
uuid | string |  资源的通用唯一识别码。这在Rancher安装过程中始终保持唯一。
