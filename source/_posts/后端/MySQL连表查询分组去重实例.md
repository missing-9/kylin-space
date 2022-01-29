---
title: MySQL连表查询分组去重实例
date: 2021-06-27 22:48:52
tags: [MySQL]
categories: 后端
---

## 业务逻辑
通过多种渠道将小程序的活动页链接发布出去，比如通过多多种短信附带链接( channel 就记为 sms1，sms2，sms3 )，或者海报上面贴微信小程序的二维码( channel 记为 qrcode1，qrcode2，qrcode3 )，线下会员通过扫描二维码也能进入小程序指定的活动页，亦或者是通过其他会员分享的小程序链接也可以进入小程序( channel 记为 share)。这些不同的进入方式在我这篇文章统称为不同的渠道，也就是提到的 channel 字段。从不同的渠道进入活动页就会产生一条页面访问记录。会被计入 page_view 这张表里。

会员进入小程序的指定活动页后，在页面上面触发一系列操作后，会得到相应的反馈，比如获得积分，或者获得优惠券等等。这步操作称为参与活动。这条数据会被记入 activity_record 这张表里。

现在呢，运营小姐姐要求得到一份数据报表。每位参与活动的会员是从什么时间,哪个渠道里面进活动的？

## 数据表结构
| 表名 | member_id |participate_time|
| --- | --- |---|
| activity_record | 会员号 |活动参与时间|

| 表名 | member_id |channel|view_time|
| --- | --- |---|---|
| page_view | 会员号 |渠道|页面访问时间|

## 查询逻辑

因为每位会员只能参加一次活动，也就是活动期间只能获得过一次积分，或者领取过一次优惠券等等这种意思，也就是每位会员最多只会产生一条 activity_record 记录。

可是 page_view 这张表的记录方式就不一样了。会员可能既收到过短信链接，又扫描过活动二维码，又被好友分享过活动链接，这下，对于这位会员来说，就会产生多条页面访问记录，即在 page_view 里产生多条数据。

你想想，会员肯定是先通过某一个渠道进入到活动页面，才能去参加活动。也就是有多条 page_view 的数据，按照 view_time 倒序排列，总有一条的 view_time 是小于且最接近于 activity_record 的 participate_time，下一条 page_view 的 view_time 就会大于 activity_record 的 participate_time。

## SQL脚本
```sql
select c.member_id,c.view_time,.channel from (
SELECT
	member_id,
	SUBSTRING_INDEX( GROUP_CONCAT( view_time ORDER BY view_time DESC ), ',', 1 ) AS view_time,
	SUBSTRING_INDEX( GROUP_CONCAT( channel ORDER BY channel DESC ), ',', 1 ) AS channel
FROM
	page_view a LEFT JOIN activity_record b
        on a.member_id = b.member_id
        where a.view_time < b.participate_time
GROUP BY
	member_id) c;
```

## 脚本说明
- GROUP_CONCAT：通过使用distinct可以排除重复值； group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )
- SUBSTRING_INDEX：字符串截取函数。substring_index(str,delim,count)。str:要处理的字符串；delim:分隔符；count:计数




