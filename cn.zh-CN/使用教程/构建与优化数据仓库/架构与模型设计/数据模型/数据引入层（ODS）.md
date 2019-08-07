# 数据引入层（ODS） {#concept_226674 .concept}

ODS层存放您从业务系统获取的最原始的数据，是其他上层数据的源数据。业务数据系统中的数据通常为非常细节的数据，经过长时间累积，且访问频率很高，是面向应用的数据。

**说明：** 在构建MaxCompute数据仓库的表之前，您需要首先了解MaxCompute支持的[数据类型](../../../../cn.zh-CN/开发/数据类型.md#)。

## 数据引入层表设计 {#section_hvt_7rj_ngj .section}

本教程中，在ODS层主要包括的数据有：交易系统订单详情、用户信息详情、商品详情等。这些数据未经处理，是最原始的数据。逻辑上，这些数据都是以二维表的形式存储。虽然严格的说ODS层不属于数仓建模的范畴，但是合理的规划ODS层并做好数据同步也非常重要。本教程中，使用了6张ODS表：

-   记录用于拍卖的商品信息：s\_auction。
-   记录用于正常售卖的商品信息：s\_sale。
-   记录用户详细信息：s\_users\_extra。
-   记录新增的商品成交订单信息：s\_biz\_order\_delta。
-   记录新增的物流订单信息：s\_logistics\_order\_delta。
-   记录新增的支付订单信息：s\_pay\_order\_delta。

**说明：** 

-   表或字段命名尽量保持和业务系统保持一致，但是需要通过额外的标识来区分增量和全量表。例如，我们通过\_delta来标识该表为增量表。
-   命名时需要特别注意冲突处理，例如不同业务系统的表可能是同一个名称。为区分两个不同的表，您可以将这两个同名表的来源数据库名称作为后缀或前缀。例如，表中某些字段的名称刚好和关键字重名了，可以通过规范定义后缀添加\_col1解决。

## ODS层设计规范 {#section_hez_rw4_ja2 .section}

ODS层表命名、数据同步任务命名、数据产出及生命周期管理及数据质量规范请参见[ODS层设计规范](../../../../cn.zh-CN/规范/数仓建设指南/ODS层设计规范.md#)。

## 建表示例 {#section_177_ad4_3ic .section}

为方便您使用，集中提供建表语句如下，示例数据请参见附录。

``` {#codeblock_vv1_t84_6rs}
CREATE TABLE IF NOT EXISTS s_auction
(
    id                             STRING COMMENT '商品ID',
    title                          STRING COMMENT '商品名',
    gmt_modified                   STRING COMMENT '商品最后修改日期',
    price                          DOUBLE COMMENT '商品成交价格，单位元',
    starts                         STRING COMMENT '商品上架时间',
    minimum_bid                    DOUBLE COMMENT '拍卖商品起拍价，单位元',
    duration                       STRING COMMENT '有效期，销售周期，单位天',
    incrementnum                   DOUBLE COMMENT '拍卖价格的增价幅度',
    city                           STRING COMMENT '商品所在城市',
    prov                           STRING COMMENT '商品所在省份',
    ends                           STRING COMMENT '销售结束时间',
    quantity                       BIGINT COMMENT '数量',
    stuff_status                   BIGINT COMMENT '商品新旧程度 0 全新 1 闲置 2 二手',
    auction_status                 BIGINT COMMENT '商品状态 0 正常 1用户删除 2下架 3 从未上架',
    cate_id                         BIGINT COMMENT '商品类目id',
    cate_name                        STRING COMMENT '商品类目名称',
    commodity_id                     BIGINT COMMENT '品类ID',
    commodity_name                    STRING COMMENT '品类名称',
    umid                              STRING COMMENT '买家umid'
)
COMMENT '商品拍卖ODS'
PARTITIONED BY (ds         STRING COMMENT '格式：YYYYMMDD')
LIFECYCLE 400;

CREATE TABLE IF NOT EXISTS s_sale
(
    id                             STRING COMMENT '商品ID',
    title                          STRING COMMENT '商品名',
    gmt_modified                   STRING COMMENT '商品最后修改日期',
    starts                         STRING COMMENT '商品上架时间',
    price                          DOUBLE COMMENT '商品价格，单位元',
    city                           STRING COMMENT '商品所在城市',
    prov                           STRING COMMENT '商品所在省份',
    quantity                       BIGINT COMMENT '数量',
    stuff_status                   BIGINT COMMENT '商品新旧程度 0 全新 1 闲置 2 二手',
    auction_status                 BIGINT COMMENT '商品状态 0 正常 1户删除 2下架 3 从未上架',
    cate_id                      BIGINT COMMENT '商品类目id',
    cate_name                    STRING COMMENT '商品类目名称',
    commodity_id                 BIGINT COMMENT '品类ID',
    commodity_name                STRING COMMENT '品类名称',
    umid                          STRING COMMENT '买家umid'
)
COMMENT '商品正常购买ODS'
PARTITIONED BY (ds      STRING COMMENT '格式：YYYYMMDD')
LIFECYCLE 400;


CREATE TABLE IF NOT EXISTS s_users_extra
(
    id                STRING COMMENT '用户id',
    logincount        BIGINT COMMENT '登录次数',
    buyer_goodnum     BIGINT COMMENT '作为买家的好评数',
    seller_goodnum    BIGINT COMMENT '作为卖家的好评数',
    level_type        BIGINT COMMENT '1 一级店铺 2 二级店铺 3 三级店铺',
    promoted_num      BIGINT COMMENT '1 A级服务　2 B级服务　3 C级服务',
    gmt_create        STRING COMMENT '创建时间',
    order_id          BIGINT COMMENT '订单ID',
    buyer_id          BIGINT COMMENT '买家id',
    buyer_nick        STRING COMMENT '买家昵称',
    buyer_star_id     BIGINT COMMENT '买家星级 ID',
    seller_id         BIGINT COMMENT '卖家ID',
    seller_nick       STRING COMMENT '卖家昵称',
    seller_star_id    BIGINT COMMENT '卖家星级id',
    shop_id           BIGINT COMMENT '店铺id',
    shop_name         STRING COMMENT '店铺名称'
)
COMMENT '用户扩展表'
PARTITIONED BY (ds       STRING COMMENT 'yyyymmdd')
LIFECYCLE 400;

CREATE TABLE IF NOT EXISTS s_biz_order_delta
(
    biz_order_id         STRING COMMENT '订单ID',
    pay_order_id         STRING COMMENT '支付订单id',
    logistics_order_id   STRING COMMENT '物流订单id',
    buyer_nick           STRING COMMENT '买家昵称',
    buyer_id             STRING COMMENT '买家id',
    seller_nick          STRING COMMENT '卖家昵称',
    seller_id            STRING COMMENT '卖家id',
    auction_id           STRING COMMENT '商品id',
    auction_title        STRING COMMENT '商品标题 ',
    auction_price        DOUBLE COMMENT '商品价格',
    buy_amount           BIGINT COMMENT '购买数量',
    buy_fee              BIGINT COMMENT '购买金额',
    pay_status           BIGINT COMMENT '支付状态 1 未付款  2 已付款 3 已退款',
    logistics_id         BIGINT COMMENT '物流订单id',
    mord_cod_status      BIGINT COMMENT '物流状态 0 初始状态 1 接单成功 2 接单超时3 揽收成功 4揽收失败 5 签收成功 6 签收失败 7 用户取消物流订单',
    status               BIGINT COMMENT '状态 0 订单正常 1 订单不可见',
    sub_biz_type         BIGINT COMMENT '业务类型 1 拍卖 2 购买',
    end_time             STRING COMMENT '交易结束时间',
    shop_id              BIGINT COMMENT '店铺id'
)
COMMENT '交易成功订单日增量表'
PARTITIONED BY (ds       STRING COMMENT 'yyyymmdd')
LIFECYCLE 7200;


CREATE TABLE IF NOT EXISTS s_logistics_order_delta
(
    logistics_order_id STRING COMMENT '物流订单id ',
    post_fee           DOUBLE COMMENT '物流费用',
    address            STRING COMMENT '收货地址',
    full_name          STRING COMMENT '收货人全名',
    mobile_phone       STRING COMMENT '移动电话',
    prov               STRING COMMENT '省份',
    prov_code          STRING COMMENT '省份ID',
    city               STRING COMMENT '市',
    city_code          STRING COMMENT '城市ID',
    logistics_status   BIGINT COMMENT '物流状态
1 - 未发货
2 - 已发货
3 - 已收货
4 - 已退货
5 - 配货中',
    consign_time       STRING COMMENT '发货时间',
    gmt_create         STRING COMMENT '订单创建时间',
    shipping           BIGINT COMMENT '发货方式
1，平邮
2，快递
3，EMS',
    seller_id          STRING COMMENT '卖家ID',
    buyer_id           STRING COMMENT '买家ID'
)
COMMENT '交易物流订单日增量表'
PARTITIONED BY (ds                 STRING COMMENT '日期')
LIFECYCLE 7200;


CREATE TABLE IF NOT EXISTS s_pay_order_delta
(
    pay_order_id     STRING COMMENT '支付订单id',
    total_fee        DOUBLE COMMENT '应支付总金额 （数量*单价）',
    seller_id STRING COMMENT '卖家id',
    buyer_id  STRING COMMENT '买家id',
    pay_status       BIGINT COMMENT '支付状态
1等待买家付款，
2等待卖家发货，
3交易成功',
    pay_time         STRING COMMENT '付款时间',
    gmt_create       STRING COMMENT '订单创建时间',
    refund_fee       DOUBLE COMMENT '退款金额(包含运费)',
    confirm_paid_fee DOUBLE COMMENT '已经确认收货的金额'
)
COMMENT '交易支付订单增量表'
PARTITIONED BY (ds        STRING COMMENT '日期')
LIFECYCLE 7200;
```

## 数据引入层存储 {#section_vbp_cpg_4v2 .section}

为了满足历史数据分析需求，您可以在ODS层表中添加时间维度作为分区字段。实际应用中，您可以选择采用增量、全量存储或拉链存储的方式。

-   增量存储

    以天为单位的增量存储，以业务日期作为分区，每个分区存放日增量的业务数据。 举例如下：

    -   1月1日，用户A访问了A公司电商店铺B，A公司电商日志产生一条记录t1。1月2日，用户A又访问了A公司电商店铺C，A公司电商日志产生一条记录t2。采用增量存储方式，t1将存储在1月1日这个分区中，t2将存储在1月2日这个分区中。
    -   1月1日，用户A在A公司电商网购买了B商品，交易日志将生成一条记录t1。1月2日，用户A又将B商品退货了，交易日志将更新t1记录。采用增量存储方式，初始购买的t1记录将存储在1月1日这个分区中，更新后的t1将存储在1月2日这个分区中。
    **说明：** 交易、日志等事务性较强的ODS表适合增量存储方式。这类表数据量较大，采用全量存储的方式存储成本压力大。此外，这类表的下游应用对于历史全量数据访问的需求较小（此类需求可通过数据仓库后续汇总后得到）。例如，日志类ODS表没有数据更新的业务过程，因此所有增量分区UNION在一起就是一份全量数据。

-   全量存储

    以天为单位的全量存储，以业务日期作为分区，每个分区存放截止到业务日期为止的全量业务数据。 举例如下：

    -   1月1日，卖家A在A公司电商网发布了B、C两个商品，前端商品表将生成两条记录t1、t2。1月2日，卖家A将B商品下架了，同时又发布了商品D，前端商品表将更新记录t1，同时新生成记录t3。采用全量存储方式， 在1月1日这个分区中存储t1和t2两条记录，在1月2日这个分区中存储更新后的t1以及t2、t3记录。
    **说明：** 对于小数据量的缓慢变化维度数据，例如商品类目，可直接使用全量存储。

-   拉链存储

    拉链存储通过新增两个时间戳字段（start\_dt和end\_dt），将所有以天为粒度的变更数据都记录下来，通常分区字段也是这两个时间戳字段。

    拉链存储举例如下。

    |商品|start\_dt|end\_dt|卖家|状态|
    |--|---------|-------|--|--|
    |B|20160101|20160102|A|上架|
    |C|20160101|30001231|A|上架|
    |B|20160102|30001231|A|下架|

    这样，下游应用可以通过限制时间戳字段来获取历史数据。例如，用户访问1月1日数据，只需限制 stat\_dt<=20160101 and end\_dt\>20160101。


## 缓慢变化维度 {#section_q4y_1e5_c4a .section}

MaxCompute不推荐使用代理键，推荐使用自然键作为维度主键，主要原因有两点:

1.  MaxCompute是分布式计算引擎，生成全局唯一的代理键工作量非常大。当遇到大数据量情况下，这项工作就会更加复杂，且没有必要。
2.  使用代理键会增加ETL的复杂性，从而增加ETL任务的开发和维护成本。

在不使用代理键的情况下，缓慢变化维度应该通过两种方式处理:

1.  快照方式。

    数据的计算周期通常为每天一次。基于该周期，处理维度变化的方式为每天一份全量快照。

    例如商品维度，每天保留一份全量商品快照数据。任意一天的事实表均可以取到当天的商品信息，也可以取到最新的商品信息，通过限定日期，采用自然键进行关联即可。该方式的优势主要有以下两点:

    -   处理缓慢变化维度的方式简单有效，开发和维护成本低。
    -   使用方便，易于理解。数据使用方只需要限定日期即可取到当天的快照数据。任意一天的事实快照与任意一天的维度快照通过维度的自然键进行关联即可。
    该方法的弊端主要是存储字样的极大浪费。例如某维度每天的变化量占总体数据量比例很低，极端情况下，每天无变化，这种情况下存储浪费严重。该方法主要实现了通过牺牲存储获取ETL效率的优化和逻辑上的简化。请避免过度使用该方法，且必须要有对应的数据生命周期制度，清除无用的历史数据。

2.  拉链存储（极限存储）

    前文已经为您介绍了拉链存储。该方法不利于数据使用者对数仓的理解，同时因为限定生效日期，产生大量分区，考虑到MaxCompute对分区数量有上限，不利于长远的数仓维护。

    极限存储底层的数据进行拉链存储，但上层通过视图操作或Hook，借助分析语句的语法树，将对极限存储之前的表查询转化成对极限存储表的查询。对于下游用户而言，极限存储表和全量存储方式是一样的。此外，为解决历史拉链分区大量产生的问题，极限存储中把日分区缩减为月份区，即以月为周期完成拉链存储。

    极限存储虽然可以压缩大量的存储空间，但使用麻烦。主要难点在于全量表的维护和过滤变更频繁的维度属性。


综上，通常情况下推荐您使用快照方式处理缓慢变化维。在数据量巨大的情况下，建议您使用拉链存储（极限存储）。

## 数据同步加载与处理 {#section_nlx_naw_1ox .section}

ODS的数据需要由各数据源系统同步到MaxCompute，才能用于进一步的数据开发。本教程建议您使用DataWorks数据集成功能完成数据同步，详情请参见[数据集成概述](../../../../cn.zh-CN/使用指南/数据集成/数据集成简介/数据集成概述.md#)。在使用数据集成的过程中，建议您遵循以下规范：

-   一个系统的源表只允许同步一次到MaxCompute，保持表结构的一致性。
-   数据集成仅用于离线全量数据同步，实时增量数据同步需要您使用数据传输服务DTS实现，详情请参见[数据传输服务DTS](https://help.aliyun.com/document_detail/26592.html)。
-   数据集成全量同步的数据直接进入全量表的当日分区。
-   ODS层的表建议以统计日期及时间分区表的方式存储，便于管理数据的存储成本和策略控制。
-   数据集成可以自适应处理源系统字段的变更：
    -   如果源系统字段在MaxCompute上目标表不存在，可以由数据集成自动添加不存在的表字段。
    -   如果目标表的字段在源系统不存在，数据集成填充NULL。

