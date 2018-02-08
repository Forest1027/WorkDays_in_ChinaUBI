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


**sql优化**

1. 索引

        创建了索引，不一定会走索引。如下：

        ```sql
        -- uc.user_id创建了索引，而此sql并不会走这个索引
        SELECT  tc.coupon_id ,tc.coupon_money ,uc.order_id ,u.tel,uc.user_id
        FROM biz_pj_9999.tp_user_coupon uc
        LEFT  JOIN biz_pj_9999.tp_coupon tc ON tc.coupon_id= uc.coupon_id
        LEFT JOIN biz_pj_9999.tp_user u  on u.user_id = uc.user_id

        -- 除非加了where筛选。下面的sql就可以走索引
        SELECT  tc.coupon_id ,tc.coupon_money ,uc.order_id ,u.tel,uc.user_id
        FROM biz_pj_9999.tp_user_coupon uc
        LEFT  JOIN biz_pj_9999.tp_coupon tc ON tc.coupon_id= uc.coupon_id
        LEFT JOIN biz_pj_9999.tp_user u  on u.user_id = uc.user_id
        WHERE uc.user_id = 1

        -- 另外，当此语句查询结果作为关联表又被leftjoin的情况，即使没有where筛选，索引也会生效 
        SELECT card.fk_oil_card_id ,card.card_type ,card.fk_oil_card_no as oil_card_no ,card.status,user_coupon.tel as device_id ,user_coupon.coupon_money from biz_pj_9999.tp_fk_oil_card card
        LEFT JOIN (
                -- 此处的语句是上面的例句
                SELECT  tc.coupon_id ,tc.coupon_money ,uc.order_id ,u.tel,uc.user_id
                FROM biz_pj_9999.tp_user_coupon uc
                LEFT  JOIN biz_pj_9999.tp_coupon tc ON tc.coupon_id= uc.coupon_id
                LEFT JOIN biz_pj_9999.tp_user u  on u.user_id = uc.user_id
            ) user_coupon  ON user_coupon.user_id=card.user_id

        ```

2. left join

        使用left join的时候，将小数据量的查询 left join 大数据量的查询。这样效率会得以提升

> 索引只能将原有的sql效率提高三倍，在加了索引还不能满足需求的情况下，可以考虑重写sql