## 理论篇~第三章 数据模型设计

### 常见数据模型介绍  
#### 1 ER模型

数据仓库之父Bill Inmon提出的建模方法，是从全企业的高度设计一个3NF模型，用实体关系（Entity Relationship，ER）模型描述企业业务。其具有以下几个特点：

- 需要全面了解企业业务和数据
- 实施周期非常长
- 对建模型人员要求非常高

采用ER模型建设数据仓库的出发点是基于整合数据，将各个系统的数据以企业角度按主题进行组合和合并，并进行一致性处理，为数据分析决策服务，但是并不能直接用于分析决策。其建模步骤分为三个阶段：

- 高层模型：一个高度抽象的模型，描述主题与主题之间的关系，用于描述企业的业务总体概况。
- 中层模型：在高层模型的基础上，细化主题的数据项。
- 物理模型：在中层模型的基础上，考虑物理存储，同时基于性能和平台特点进行物理属性的设计，也可能做一些表的合并、分区表的设计等。

#### 2 维度模型

维度模型是数据仓库的Ralph Kimball大师所倡导的，也是数据仓库工程领域最流行 模型。维度建模是从分析决策的需求出发构建模型，为分析需求服务，因此它重点关注用户如何更快速地完成分析，同时具有较好的大规模复杂查询的响应性能。其典型的代表是星形模型和雪花模型。其设计步骤分为以下几个步骤：

- 选择需求进行决策的业务过程。业务过程可以是单个业务事件，比如交易的支付、退款等；也可以是某个事件的状态，比如当前的账户余额等；还可以是一系列相关业务事件组成的业务流程，具体需要看我们分析的是某些事件发生情况，还是当前状态，或是事件流转效率。
- 选择粒度。预判所分析的数据需要细分到什么程度，从而决定选择哪个粒度。
- 识别维度。选择好粒度，则围绕粒度设计维度表，配置维度属性用于分析数据时进行分组和筛选。
- 选择事实。确定分析需要衡量的指标。

#### 3 Data Vault模型

Data Vault是Dan Linstedt发起创建的一种模型，它是ER模型的衍生。它强调建立一个可审计的基础数据层，也就是强调数据的历史性、可追溯性和原子性，而不要求对数据进行过度的一致性处理和整合；同时他基于主题概念将企业数据进行结构化组织，并引入了更进一步的发生处理来优化模型，以应对预案系统变更的扩展性。Data Vault模型由以下几部分组成：

- Hub:是企业的核心业务实体，由实体key、数据仓库序列代理键、装载时间、数据来源组成。
- Link:代表Hub之间的关系。它可以描述1:1、1:n和n:n的关系。它由Hub的代理键、装载时间、数据来源组成。
- Satellite: 是Hub的详细描述内容，一个Hub可以有多个Satellite。它由Hub的代理键、装载时间、来源类型、详细的Hub描述信息组成。

Data Vault模型比ER模型更容易设计和产出，它的ETL加工可实现配置化。Data Vault模型实例如下图：
![]()

#### 4 Anchor模型

Anchor模型是对Data Vault模型做了进一步规范化处理，其核心思想是所有扩展只是添加而不是修改，因此将模型规范到6NF。但是这样大大增加了查询join操作，不太适合有比较多的join操作的数据仓库。

#### 5 阿里巴巴数据模型

阿里巴巴为了满足日益增长的数据，选择了维度建模为核心理念的模型方法论，同时对其进行了一定的升级和扩展，构建了阿里巴巴集团的公共层模型数据架构体系。

数据公共层建设的目的是着力解决数据存储和计算的共享问题，其指导方法是一套统一化的集团数据整合及管理的方法体系（OneData），其包括一致性的指标定义体系、模型设计方法体系及配套工具。

### 阿里数据整合及管理体系

#### 1 模型设计
目前，数据公共层设计最流行是维度建模。它分三层：操作数据层（ODS）、公共维度模型层（CDM）和应用数据层（ADS）。其中，CDM又分明细数据层（DWD）和汇总数据层（DWS）。

**操作数据层**：基于无差别同步数据源表到数据仓库，根据数据业务需求及稽核和审计要求保留历史数据、清洗数据。

**公共维度模型层**：存放明细事实数据、维表数据及公共指标汇总数据，采用一些维度退化手段，将维度退化至事实表中，减少事实表和维表的关联，提高明细数据表的易用性；同时在汇总数据层，加强指标的维度退化，采取更多的宽表化手段构建公共指标数据层，提升公共指标的复用性，减少重复加工。其主要功能如下：

- 组合相关和相似数据：采用明细宽表，复用关联计算，减少数据扫描。
- 公共指标统一加工：构建命名规范、口径一致和算法统一的统计指标，为上层数据产品、应用服务提供公共指标；建立逻辑汇总宽表。
- 建立一致性维度：建立一致的数据分析维表，降低数据计算口径、算法不统一的风险。

**应用数据层**：存放数据产品个性化的统计指标数据，根据CDM层和ODS层加工生成。

- 个性化指标加工：不公共性、复杂性（指标型、比值型、排名型指标）。
- 基于应用的数据组装：大宽表集市、横表转纵表、趋势指标串。

#### 2 基本原则

2.1 高内聚低耦合
        
主要从数据业务特性和访问特性两个角度来考虑：将业务相近或相关、粒度相同的数据设计为一个逻辑或物理模型；将高概率同时访问的数据放在一起，降低概率同时访问的数据分开存储。

2.2 核心模型与扩展模型分离

避免破坏核心模型的架构简洁性和可维护性。

2.3 公共处理逻辑下沉及单一

越是底层共用的处理逻辑越应该在数据调度依赖的底层进行封装与实现，不要让公用的处理逻辑暴露给应用实现，不要让公用逻辑多处同时存在。

2.4 成本与性能平衡

适当的数据冗余可换取查询和刷新性能，不宜过度冗余和数据重复。

2.5 数据可回滚

处理逻辑不变，在不同时间多次运行数据结果确定不变。

2.6 一致性
        具有相同含义的字段在不同表的命名必须相同，必须使用规范定义中的名称。
    2.7 命名清晰、可理解

#### 3 模型实施

3.1 指导方针

首先，进行充分的业务调研和需求分析。

其次，进行数据总体架构设计，主要是根据数据域对数据进行划分；安装维度建模理论，构建总线矩阵、抽象出业务过程和维度。

再次，对报表需求进行抽象整理出相关指标体系。

最后，代码研发和运维。

3.2 实施工作流

> https://blog.csdn.net/wer0735/article/details/78075223