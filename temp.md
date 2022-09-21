[TOC]
# 0918 项目整理
## 电商
### dwd层设计
#### 事务型事实表-增量
>用于只有新增数据，不会对旧数据修改。涉及的表：订单明细表，退单表，评价表
>关于订单明细表：当我们将购买物品时，需要选定好物品后点击“提交订单”，然后就会生成订单号、订单明细，后续还有支付、退款、退单等状态。订单一旦创建，后续的流程并不会修改订单明细表，而是修改订单表的状态，订单明细中存储的是商品id、商品数量、活动id、优惠券id。
```
insert overwrite table dwd_comment_info partition(dt='2020-06-15')
select
    id,
    user_id,
    sku_id,
    spu_id,
    order_id,
    appraise,
    create_time
from ods_comment_info where dt='2020-06-15';
```
#### 周期型快照事实表-全量
>数据量不大，有增加有修改旧数据的。涉及的表：收藏表，加购表，       商品一级品类、二级品类、三级品类，优惠券表，活动表

#### 累积型快照事实表-增量及变化
>数据量大，有增加有旧数据的修改。订单表，支付表，退款表，优惠券领用表。

### 指标计算
#### 用户统计
>用户常规指标聚合：新增用户数、新增下单用户数、下单总金额、下单用户数、未下单用户数
>原本在dwt中，1日，7日，30日粒度的数据在同一行，通过explode可以将array中的元素变为多行。然后再借助login_date_first、order_date_first、order_final_amount计算指标.
```
insert overwrite table ads_user_total
select * from ads_user_total
union
select
    '2020-06-14',
    recent_days,
    sum(if(login_date_first>=recent_days_ago,1,0)) new_user_count,
    sum(if(order_date_first>=recent_days_ago,1,0)) new_order_user_count,
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count,
    sum(if(login_date_last>=recent_days_ago and order_final_amount=0,1,0)) no_order_user_count
from
(
    select
        recent_days,
        user_id,
        login_date_first,
        login_date_last,
        order_date_first,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount,
        if(recent_days=0,'1970-01-01',date_add('2020-06-14',-recent_days+1)) recent_days_ago
    from dwt_user_topic lateral view explode(Array(0,1,7,30)) tmp as recent_days
    where dt='2020-06-14'
)t1
group by recent_days;

```
#### 留存率
>以2022-09-20的7日留存率为例,即第一次登录是7日前，并且最后一次登录是今天。统计时，一般统计多个，如从7日留存，6日一直到1日。sql语句如下：
``` 
# dwt 该日分区存储的是该日粗粒度的统计及部分属性信息。first_login表示该日进行注册。last_login表示最后一次活跃。
#retention_day 表示留存多少天，6天就是6日留存。retention_count表示留存的人数.all_new_count表示n天前总新增人数。retention_rate表示留存率。
select 
  '2022-09-20' dt,
  first_login create_date,
  datediff('2022-09-20',first_login) retention_day,
  sum(if(last_login='2022-09-20',1,0)) retention_count,
  count(*) all_new_count,
  cast(sum(if(last_login='2022-09-20',1,0))/count(*),decimal(16,2)) retention_rate
from dwt_user_topic 
    where dt='2022-09-20'  first_login>=date_add('2022-09-20',-7) and first_login<'2022-09-20'
group by first_login;
```
#### 流失
>就是last_login=七天前，最后登录时间为7天前，那么就计入今天的流失人数。
```
select
  '2022-09-20'  dt,
  count(*) churn_count,
  from dwt_user_topic
  where dt='2022-09-21'
where last_login=date_add('2022-09-20','-7')
```
#### 回流
>7日之内未活跃，那么称为流失了。那么回流的定义就是8日没有活跃，并且今天活跃了，称为回流。也就是说需要每个用户的倒数第二次登录时间为7日前，最后一次登录为今天。.
```
# dwt 该日分区存储的是该日1，7，30粒度的聚合信息。
#那么这样取，得到的就是2022-09-19日的聚合信息，在19日及之前最后一次登录时间，也就是倒数第二次登录时间。倒数第一次登录时间就是2022-09-20，将两者相减得到相距时间，判断是否>=8天，是否为回流用户。
select  
  user_id,
  count(*)
  from
  (
    (select 
        user_id,
            login_date_last login_date_previous
        from dwt_user_topic
        where dt=date_add('2022-09-20')
    )
    join
    (select
        user_id,
        login_date_last login_date_previous
    from dwt_user_topic
    where dt=date_add('2022-09-20',-1)
    )
    on t1.user_id=t2.user_id
  )t3
  where date_gap>=8;
```
#### 行为漏斗
>进行每个行为的用户数各有多少，进入home页、进入详情页、下单。explode将一行扩充为3行，三组数据分别用于计算不同粒度的行为数。然后对count求和，得到进行不同行为的用户数。
```
    select
        '2020-06-14' dt,
        recent_days,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from
    (
        select
            recent_days,
            user_id,
            case
                when recent_days=1 then cart_last_1d_count
                when recent_days=7 then cart_last_7d_count
                when recent_days=30 then cart_last_30d_count
            end cart_count,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then payment_last_1d_count
                when recent_days=7 then payment_last_7d_count
                when recent_days=30 then payment_last_30d_count
            end payment_count
        from dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='2020-06-14'
    )t1
    group by recent_days
```

#### 访客统计
>这里需要划分会话，判断那些记录属于同一个会话。
>当按照ts排序后，每当出现一个last_page_id为空的，说明是新开了会话。所以会话id通过取最后一个last_page_id为空的的记录的时间戳计算，将mid_id和ts拼接起来表示一个会话。
```
 concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by recent_days,mid_id order by ts)) session_id
```

#### 路径分析
>先通过开窗划分会话，然后通过开窗取下一行作为target， lead(page_id,1,null) over (partition by recent_days,session_id order by ts) target。并且取到step，row_number() over (partition by recent_days,session_id order by ts) step。
```
insert overwrite table ads_page_path
select * from ads_page_path
union
select
    '2020-06-14',
    recent_days,
    source,
    target,
    count(*)
from
(
    select
        recent_days,
        concat('step-',step,':',source) source,
        concat('step-',step+1,':',target) target
    from
    (
        select
            recent_days,
            page_id source,
            lead(page_id,1,null) over (partition by recent_days,session_id order by ts) target,
            row_number() over (partition by recent_days,session_id order by ts) step
        from
        (
            select
                recent_days,
                last_page_id,
                page_id,
                ts,
                concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by mid_id,recent_days order by ts)) session_id
            from dwd_page_log lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('2020-06-14',-30)
            and dt>=date_add('2020-06-14',-recent_days+1)
        )t2
    )t3
)t4
group by recent_days,source,target;

```

#### 品牌复购率
>用户购买多次某个商品/用户购买一次某个商品。
>先统计每个用户购买了各种商品多少次，然后按照商品group by，通sum(if(order_count>=2,1,0))/sum(if(order_count>=1,1,0))
```
insert overwrite table ads_repeat_purchase
select * from ads_repeat_purchase
union
select
    '2020-06-14' dt,
    recent_days,
    tm_id,
    tm_name,
    cast(sum(if(order_count>=2,1,0))/sum(if(order_count>=1,1,0))*100 as decimal(16,2))
from
(
    select
        recent_days,
        user_id,
        tm_id,
        tm_name,
        sum(order_count) order_count
    from
    (
        select
            recent_days,
            user_id,
            sku_id,
            count(*) order_count
        from dwd_order_detail lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('2020-06-14',-29)
        and dt>=date_add('2020-06-14',-recent_days+1)
        group by recent_days, user_id,sku_id
    )t1
    left join
    (
        select
            id,
            tm_id,
            tm_name
        from dim_sku_info
        where dt='2020-06-14'
    )t2
    on t1.sku_id=t2.id
    group by recent_days,user_id,tm_id,tm_name
)t3
group by recent_days,tm_id,tm_name;
```
