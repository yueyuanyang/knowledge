## 理论篇~第二章 数据仓库的命名规范

大家可能对命名规范重视不太够。在多年的工作中，碰到太多由于命名不规范，导致代码混乱和数据管理困难等问题。这个问题在元数据管理时，体现得特别重要。

当然，每个公司的命名规范不一样，只要做到易理解、易管理就行。接下来，说说自己的理解。

### 1 表
表的命名首先按数据分层和主题来划分来定规范。

#### 1.1 ODS层（操作数据层）
ODS层作为最底层，应该表名和字段保存来源表一致，字段类型可以都使用STRING。

#### 1.2 DWD层（明细数据层）
DWD层命名规范：dw_主题+common+detail

#### 1.3 DWS层（汇总数据层）
DWD层命名规范：dw_主题+common+summary

#### 1.4 ADS层 （应用数据层）
ADS层命名规范：ads_主题+common

### 2 脚本
命名规范：功能+表名
例如，统计表dw_user_car_info的总数据，命名为count_dw_user_car_info.sh

### 3 job

job的命名和脚本一样，只是后缀不同，如count_dw_user_car_info.job

### 4 元数据

命名规范：meta+功能+表名
例如，监控表数据是否为空，meta_monitor_table_null

