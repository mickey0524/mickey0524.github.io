---
layout:     post
title:      "dw-methodology"
subtitle:   "数仓方法论"
date:       2019-08-24 23:00:00
author:     "Mickey"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
tags:
    - 大数据
---

最近面试的时候，面试官建议我去看看数据仓库的方法论，我思考了一下，觉得自己以往搭建数仓确实是凭借自己对数据的感觉的，缺乏理论的支持，于是决定看一下数据仓库工具箱这本书，这篇文章用来记录一下感悟

* 维度模型设计的四个步骤

    1. 选择业务过程：我认为这就是理解自己的业务是在做什么
    2. 声明粒度：精确定义某个事实表的每一行在表示什么
    3. 确定纬度：确定度量环境中所有可能出现的单值描述符
    4. 确定事实：确定过程的度量是什么

    ![dw_1](/img/in-post/dw-methodology/dw_1.png)

* 零售业务
	
	拆解最细的粒度，POS 机上扫过的一件商品
	
	日期维度，产品维度，商店维度，出纳员维度，支付方式维度，金额，促销维度等等
	
	在 DW 中，尽量不要存储 NULL，可以用 UNKNOWN、Not Applicable 等标识显示表示

* 库存模型

	1. 库存周期快照（粒度是仓库中的产品）

		日期维度、产品维度、商店维度、库存数量、周期内出库数量...
		
	2. 库存事务（粒度是改变仓库的事务）

		日期维度、产品维度、仓库维度、库存事务类型维度...
		
	3. 库存累积快照（用于定义过程开始、结束以及期间的可区分的里程碑，在一条 record 上修改）

		接收日期维度、验收日期维度、入库日期维度、初始装运日期维度、最终装运日期维度，产品维度，仓库维度，供应商维度，两个阶段之间的时间差...

* 企业数据仓库总线矩阵

	纵轴是业务过程，横轴是维度，这样有助于实现 DW 的**一致性**，一致性非常重要
	
	![dw_2](/img/in-post/dw-methodology/dw_2.png)

* 有时候，在设计事实表的时候，会面对一个设计决策，是为所有的事务类型建立混合事务事实表，还是为每个事务类型建立不同的事实表，这个时候，可以思考下列问题

    * 用户的分析需求是什么？目标是通过最有效的方式给商业用户展现数据。商业用户最常用的分析数据的方式是什么？哪种方法能够最自然地与他们的业务相契合

    * 是否存在多个独特的业务过程

    * 多个源系统获取同样粒度的度量吗

* SCD（Slowly Changing Dimension，缓慢变化维度）的解决方法

    * 重写，直接在原始表上改写数据

    * 增加新行，新增 start\_time、end\_time 列来表示当前行的有效期，可以根据业务需求建立 change\_reason 的列来指代修改原因

    * 增加新属性，在当前行上新增字段来表示历史属性

    * 增加微型维度，将频繁变化的属性，抽成一个微型维度表（属性的排列组合）

    * 微型维度 + 重写支架
        
        ![scd_5](/img/in-post/dw-methodology/scd-5.png)        

    * 重写 + 增加新行 + 新增新属性

        ![scd_6](/img/in-post/dw-methodology/scd-6.png)        

    * 重写 + 增加新行 + 新表

        ![scd_7](/img/in-post/dw-methodology/scd-7.png)        

* 当某个维度在单一事实表中同时出现多次时，则会存在维度模型的角色扮演，基本维度可能作为单一物理表存在，但是每种角色应该被当成标识不同的视图展现到 BI 工具中

    例如订单事务中，事实表中的一条记录可能有两个时间维度 —— 订单日期维度和请求发货日期维度

    ![order_1](/img/in-post/dw-methodology/order_1.png)

* 当实体之间存在固定的、不随时间变化的、强关联的关系时，它们应该被建模到单一维度中去

* 事实表中仅包含维度键和度量，以及偶尔存在的退化维度

* 杂项维度是对低粒度标志和指标的分组。通过建立杂项维度，将标志和指标从事实表中移除，并将它们放入有用的多维框架中，关于杂项维度的一个微妙的问题是，是否应该事先为所有组合的完全笛卡尔积建立行，或者是建立杂项维度行，用于保存那些遇到情况的数据。答案要看大概有多少可能的组合，最大行数是多少。一般来说，理论上组合的数量大而且不太可能用到全体组合的时候，当遇到新标志或指标组合时建立杂项维度行

    ![order_2](/img/in-post/dw-methodology/order_2.png)

* 不要在单一事实表中混淆类似订单表头或订单明细事实粒度。相反，要么将高层次事实分解到更详细的层次，要么建立两个不同的事实表来处理不同粒度的事实，分配是最好的解决方案

    ![order_3](/img/in-post/dw-methodology/order_3.png)

* 累积快照事实表适用于流水线业务，周期性且时间范围长的业务适合用周期快照事实表
    
* 网站点击流网页事实

    ![web_1](/img/in-post/dw-methodology/web_1.png)

    上图中可以看到步骤维度，我们知道网页粒度上也有很多 area，或者在一个网页上执行操作需要有一个步骤，这就是步骤维度的用处，可以用于分析在当前网页上操作的漏斗

* 设计者总是喜欢在事实表中存储 "到目前为止" 列，他们认为在每个事实行中存储 "到目前为止季度" 或 "到目前为止年" 等相加后的总计是比较有益的，因为他们可以直接获取，不需要计算，请不要这样做，事实表必须和粒度保持一致，应该将他们从设计的模式中拿走，通过计算获得

* 维度属性层次

    * 轻微不整齐的可变深度层次

        例如地理层次，最简单的位置具有4个级别：地址、城市、州和国家，比较复杂一点的情况增加了区域，最复杂的情况增加了区和区域级别

        可以统一处理，用城市名填在区和区域的位置上，也可以填入 Not Applicable

        ![account_1](/img/in-post/dw-methodology/account_1.png)

    * 不整齐可变深度层次

        例如组织维度，很多时候是一个组织树

        ![account_2](/img/in-post/dw-methodology/account_2.png)

        这个时候可以建立一个桥接表

        ![account_3](/img/in-post/dw-methodology/account_3.png)

        ![account_3](/img/in-post/dw-methodology/account_4.png)

    * 随时间变化的不规则层次

        新增有效时间和失效时间

        ![account_5](/img/in-post/dw-methodology/account_5.png)

* 应用于多值维度的桥接表

    ![client_1](/img/in-post/dw-methodology/client_1.png)

* 桥接表和组合字符串

    * 桥接表

        ![employee_1](/img/in-post/dw-methodology/employee_1.png)

    * 组合字符串

        可以将多数值存储为维度表中的属性 'A|B|C' 或 ["A", "B", "C"]，如果使用比较多的话，也可以抽取支架维度

        ![employee_2](/img/in-post/dw-methodology/employee_2.png)

* 桥接表有时候可以增加权重因子，这些权重因子是可加事实，相加等于一，可以用于校验是否正确

    ![bank_1](/img/in-post/dw-methodology/bank_1.png)

    有些人可能会建议根据权重因子改变事实表的粒度，比如银行账户，乘上权限因子，拆成按客户粒度的账户事实

    一般我们不建议采用此方法，事实表可能不止一个多值维度，行数无法控制