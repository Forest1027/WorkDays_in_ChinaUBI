# MySQL

**[concat、concat_ws、group_concat](https://www.cnblogs.com/xbblogs/p/6066386.html)**

```sql
# group_concat函数示例
SELECT
        GROUP_CONCAT(
                CASE prize.activity_reward_type
                WHEN 1 THEN
                        concat(
                                prize.release_points,
                                '积分'
                        )
                ELSE
                        coupon.coupon_name
                END
        ) AS prizeName,
        prize.activity_id AS prize_activity_id
FROM
        biz_pj_9999.tp_activity_reward prize
LEFT JOIN biz_pj_9999.tp_coupon coupon ON coupon.coupon_id = prize.coupon_id
WHERE prize.is_deleted=0
GROUP BY
        prize_activity_id

# concat_ws函数示例
SELECT
        concat_ws(
                ' ',
                coupon.coupon_name,
                COUNT(user_coupon.coupon_id),
                '张'
        ) AS temp_prize,
        user_coupon.activity_id,
        coupon.coupon_name
FROM
        biz_pj_9999.tp_user_coupon user_coupon
LEFT JOIN biz_pj_9999.tp_coupon coupon ON user_coupon.coupon_id = coupon.coupon_id
WHERE
        user_coupon. STATUS = 2
GROUP BY
        user_coupon.activity_id,
        coupon.coupon_name
```

**[FIND_IN_SET的使用方法](https://www.cnblogs.com/manongxiaobing/p/4682698.html)**

tp_coupon表结构

|coupon_id|coupon_range|
|:-----   |:-----
|1|1,2|

tp_code_detail表结构

|code_detail_value|code_detail_discription|
|:-----   |:-----
|1|疯狂油卡|
|2|普通油卡|

考虑到tp_coupon的coupon_range字段数据存储的形式，不能直接与tp_code_detail建立连接，因此使用find_in_set函数

```sql
# 示例
SELECT c3.coupon_id,group_concat(c2.code_detail_display Separator ',') AS code_detail_display,c3.coupon_range
from  biz_pj_9999.tp_code_detail c2,  biz_pj_9999.tp_coupon c3
where c2.code_name='COUPON_RANGE'
AND FIND_IN_SET(c2.code_detail_value  , c3.coupon_range)
GROUP BY c3.coupon_id
```

**[使用explain优化sql语句](https://www.jianshu.com/p/73f2c8448722)**




