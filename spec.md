# eSportClub 电竞陪玩店铺管理系统 — 需求规格文档

> 文档版本：v1.0
> 创建日期：2026-05-23
> 状态：草稿（待确认）

---

## 一、项目概述

### 1.1 产品定位
电竞陪玩店铺管理系统，支撑「客服派单 → 陪玩接单 → 报单 → 验收 → 结算」完整业务闭环。

### 1.2 技术架构
| 层级 | 技术选型 |
| ---- | -------- |
| 数据库 | MySQL（独立库：eSportClub） |
| 后端 | Spring Boot（独立项目） |
| 管理后台前端 | React / Vue |
| 陪玩端前端 | React / Vue |
| 老板端 | React / Vue（独立项目） |
| 小程序 | 暂不做 |

### 1.3 数据库
- 服务器：同现有 wusidai MySQL
- 库名：`eSportClub`
- 字符集：utf8mb4
- 时区：中国大陆（Asia/Shanghai）

### 1.4 鉴权方案
- 管理后台、陪玩端共用一个登录界面
- 陪玩账号由管理后台客服创建
- 复用 wusidai 鉴权逻辑（JWT / Session）
- 老板端本次不做

---

## 二、角色权限矩阵

| 角色 | 职责 | 可访问模块 |
| ---- | ---- | ---------- |
| 店长 / 超级管理员 | 全局管理、结算、店规配置 | 所有模块 |
| 客服 | 接待客户、创建/派发订单、验收、充值录入、老板账号管理 | 客服工作台、售后工作台 |
| 考核官 | 陪玩入职审核、认证审核 | 考核工作台 |
| 财务 | 充值/提现/收支管理 | 财务工作台 |
| 陪玩 | 接单、报单、查看工资流水 | 陪玩端网页 |
| 老板 | 查看累计充值/消费/余额、查看订单明细及折扣 | 老板端网页 |

---

## 三、菜单结构

### 3.1 管理后台菜单

```
店长工作台
  ├── 客户数据看板
  ├── 陪玩数据看板
  └── 管理名单

客服工作台
  ├── 派单订单详情
  ├── 老板名单
  ├── 消费账户
  ├── 充值详情
  └── 老板账号管理（导入/启用/禁用）

考核工作台
  ├── 陪玩名单
  ├── 陪玩入职
  └── 入职应聘

财务工作台
  ├── 充值记录
  ├── 提现记录
  └── 日常收支

售后工作台
  ├── 售后投诉
  └── 用户建议
```

---

## 四、数据库主要实体

### 4.1 admin（管理员表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| username | VARCHAR(50) | 登录用户名 |
| password | VARCHAR(255) | 密码（加密） |
| role | ENUM | super_admin / shop_owner / cs / assessor / finance |
| nickname | VARCHAR(50) | 昵称 |
| status | TINYINT | 1启用 0禁用 |
| created_at | DATETIME | |

### 4.2 companion（陪玩表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| companion_no | VARCHAR(20) | 陪玩编号（按入职顺序） |
| nickname | VARCHAR(50) | 陪玩昵称（禁止重复） |
| phone | VARCHAR(20) | 手机号（登录账号） |
| password | VARCHAR(255) | 密码 |
| gender | ENUM | male / female |
| wechat | VARCHAR(50) | 微信号 |
| alipay_account | VARCHAR(50) | 支付宝账号 |
| alipay_name | VARCHAR(50) | 支付宝实名 |
| id_card | VARCHAR(30) | 身份证号 |
| status | ENUM | active / leave / vacation / blacklisted |
| commission_rate | DECIMAL(5,2) | 陪玩提成率（0.70/0.80/0.90） |
| deposit_setting | DECIMAL(10,2) | 设置押金（200/300/500） |
| deposit_self | DECIMAL(10,2) | 自缴押金 |
| deposit_locked | DECIMAL(10,2) | 单抵押金（已锁定） |
| order_income | DECIMAL(10,2) | 订单收入合计 |
| companion_income | DECIMAL(10,2) | 陪玩收入合计 |
| paid_salary | DECIMAL(10,2) | 已发放工资 |
| hire_date | DATE | 入职日期 |
| games | JSON | 可接游戏类别 ["三角洲","瓦","lol"] |
| experience | ENUM | none / under1year / over1year |
| status_audit | ENUM | pending / approved / rejected |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### 4.3 boss（老板表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| boss_no | VARCHAR(20) | 老板编号 |
| nickname | VARCHAR(50) | 老板昵称 |
| gender | ENUM | male / female |
| wechat | VARCHAR(50) | 联系微信 |
| phone | VARCHAR(20) | 联系手机 |
| referrer_type | ENUM | companion / boss / none |
| referrer_id | BIGINT | 推荐人ID（陪玩或老板ID） |
| first_recharge_date | DATE | 首次充值日期 |
| total_recharge | DECIMAL(10,2) | 累计充值 |
| total_gift | DECIMAL(10,2) | 累计赠送 |
| total_consume | DECIMAL(10,2) | 累计消费 |
| total_locked | DECIMAL(10,2) | 累计锁定（未结算订单） |
| balance | DECIMAL(10,2) | 剩余余额 |
| status | TINYINT | 1启用 0禁用 |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### 4.4 boss_account（老板账户表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| boss_id | BIGINT FK | 老板ID |
| account_type | ENUM | retail(散户)/silver/gold/diamond |
| companion_id | BIGINT FK | 存单陪玩（店存时为null） |
| discount | DECIMAL(5,2) | 折扣率 |
| recharge_total | DECIMAL(10,2) | 充值合计 |
| gift_total | DECIMAL(10,2) | 赠送合计 |
| consume_total | DECIMAL(10,2) | 消费合计 |
| locked_total | DECIMAL(10,2) | 锁定合计 |
| balance | DECIMAL(10,2) | 余额合计 |
| created_at | DATETIME | |

### 4.5 recharge_record（充值记录表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| boss_id | BIGINT FK | 老板ID |
| account_id | BIGINT FK | 账户ID（散户时为null） |
| type | ENUM | new_card / old_recharge / old_card |
| account_type | ENUM | 账户类型（散户/银卡/金卡/钻石卡） |
| companion_id | BIGINT FK | 存单陪玩（店存时为null） |
| discount | DECIMAL(5,2) | 折扣 |
| recharge_amount | DECIMAL(10,2) | 充值金额 |
| gift_amount | DECIMAL(10,2) | 赠送金额 |
| payment_method | SET | 支付宝/微信/其他 |
| screenshot | VARCHAR(255) | 收款截图URL |
| cs_id | BIGINT FK | 操作客服 |
| status | ENUM | pending / confirmed / cancelled |
| created_at | DATETIME | |

### 4.6 order（订单表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| order_no | VARCHAR(30) | 订单编号（日期+序号） |
| order_date | DATE | 订单日期 |
| cs_id | BIGINT FK | 客服ID |
| boss_id | BIGINT FK | 老板ID |
| account_id | BIGINT FK | 账户ID |
| companion_id | BIGINT FK | 陪玩ID |
| game_category | VARCHAR(30) | 游戏类别 |
| game_item | VARCHAR(50) | 游戏项目 |
| unit_price | DECIMAL(10,2) | 陪玩单价 |
| extra_fees | JSON | 附加费用 [{"name":"过年+10","amount":10}] |
| duration | DECIMAL(5,1) | 时长（支持0.5） |
| discount | DECIMAL(5,2) | 折扣 |
| order_total | DECIMAL(10,2) | 订单总计 |
| discounted_total | DECIMAL(10,2) | 折后总计 |
| commission_rate | DECIMAL(5,2) | 陪玩提成率 |
| companion_income | DECIMAL(10,2) | 陪玩收入 |
| shop_income | DECIMAL(10,2) | 店内收入 |
| referrer_type | ENUM | companion/boss/none |
| referrer_id | BIGINT | 推荐人ID |
| specific_time | VARCHAR(50) | 具体时间（陪玩填写） |
| report_screenshot | VARCHAR(255) | 报单确认截图 |
| status | ENUM | dispatched/pending_confirm/confirmed/confirmed_error/settled/cancelled |
| reject_reason | TEXT | 驳回原因 |
| cs_remark | TEXT | 客服备注 |
| source | ENUM | miniapp/cs_manual/renewal |
| created_at | DATETIME | |
| updated_at | DATETIME | |

### 4.7 salary_settlement（工资结算表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| companion_id | BIGINT FK | 陪玩ID |
| period_start | DATE | 结算周期开始 |
| period_end | DATE | 结算周期结束 |
| total_orders | INT | 总单数 |
| total_income | DECIMAL(10,2) | 总流水 |
| commission_rate_avg | DECIMAL(5,2) | 平均提成率 |
| gross_salary | DECIMAL(10,2) | 应发工资 |
| fines | DECIMAL(10,2) | 罚款 |
| rewards | DECIMAL(10,2) | 带新奖励 |
| net_salary | DECIMAL(10,2) | 实发工资 |
| status | ENUM | pending/settled |
| settled_at | DATETIME | 结算时间 |
| created_at | DATETIME | |

### 4.8 withdraw_record（提现记录表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| companion_id | BIGINT FK | 陪玩ID |
| amount | DECIMAL(10,2) | 提现金额 |
| alipay_account | VARCHAR(50) | 支付宝账号 |
| alipay_name | VARCHAR(50) | 支付宝实名 |
| status | ENUM | pending/approved/rejected/paid |
| cs_id | BIGINT FK | 处理客服 |
| remark | TEXT | 备注 |
| created_at | DATETIME | |

### 4.9 complaint（售后投诉表）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | BIGINT PK | |
| order_no | VARCHAR(30) | 关联订单编号 |
| companion_id | BIGINT FK | 陪玩ID |
| boss_id | BIGINT FK | 老板ID |
| type | ENUM | complaint / suggestion |
| content | TEXT | 具体内容 |
| screenshot | VARCHAR(255) | 备注截图 |
| status | ENUM | pending/processed |
| fine_status | ENUM | fined / not_fined |
| fine_amount | DECIMAL(10,2) | 罚款金额 |
| cs_remark | TEXT | 客服备注 |
| created_at | DATETIME | |

### 4.10 dict_game_category（游戏类别字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| name | 三角洲 / 瓦 / 英雄联盟 / steam |
| sort | 排序 |
| status | 启用/禁用 |

### 4.11 dict_game_item（游戏项目字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| category_id | 关联游戏类别 |
| name | 项目名称 |
| default_price | 默认单价 |

### 4.12 dict_extra_fee（附加费用字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| name | 1陪2+20 / 过年+10 等 |
| amount | 金额 |

### 4.13 dict_discount（折扣字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| value | 0.95 / 0.9 / 0.85 / 0.8 / 0.75 / 0.7 / 0.5 |

### 4.14 dict_commission_rate（陪玩提成字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| value | 0.70 / 0.80 / 0.90 |

### 4.15 dict_deposit（押金字典）
| 字段 | 说明 |
| ---- | ---- |
| id | |
| value | 200 / 300 / 500 |

---

## 五、功能模块详细说明

### 5.1 店长工作台

#### 5.1.1 客户数据看板
- 筛选：日期范围、老板编号、老板昵称
- 充值状态：已查收 / 未查收
- 订单状态：已完成 / 已锁定
- 散户账户（可选）：张数
- 赠送/实充 占比、 消费/充值 占比、 个存/总余额 占比
- 会员账户（可选）：张数
- 店存/总余额 占比

#### 5.1.2 陪玩数据看板
- 筛选：日期范围、陪玩编号、陪玩昵称
- 订单状态：已完成 / 已锁定
- 接单总流水 / 接单总佣金（占比）

#### 5.1.3 管理名单
- 可添加店长权限管理员
- 管理 N（可命名）：筛选日期、已派单（可选订单编号）、待验收（可选订单编号）

### 5.2 客服工作台

#### 5.2.1 派单订单详情（创建订单）
| 字段 | 类型 | 必填 | 说明 |
| ---- | ---- | ---- | ---- |
| 订单日期 | date | 是 | 默认当前日期 YYYYmmDD |
| 客服 | select | 是 | 从客服列表选择 |
| 老板账户 | select | 是 | 从老板列表选择 |
| 陪玩 | select | 是 | 从陪玩列表选择 |
| 游戏类别 | select | 是 | 三角洲/瓦/英雄联盟/steam，支持新建 |
| 游戏项目 | select | 是 | 关联游戏类别，支持新建 |
| 陪玩价格 | select/input | 是 | 支持自定义，支持新建 |
| 附加费用 | multi-select | 否 | 从附加费用列表选择，支持新建 |
| 时长 | number | 是 | 支持0.5加减，无上限，最低0.5 |
| 折扣 | select | 是 | 从折扣列表选择，支持新建 |
| 具体时间 | text | 否 | 陪玩端填写 |
| 订单总计 | auto | — | （陪玩价格+附加费）× 时长 |
| 折后总计 | auto | — | 订单总计 × 折扣 |
| 陪玩提成 | select | 是 | 百分率 70%/80%/90%，支持新建 |
| 陪玩收入 | auto | — | 折后总计 × 陪玩提成 |
| 推荐人 | select/input | 否 | 从陪玩/老板列表选择，支持手动 |
| 店内收入 | auto | — | 折后总计 - 陪玩收入 |
| 报单确认截图 | file | 否 | jpg/png，最大10MB（陪玩端可上传） |

#### 5.2.2 老板名单
| 字段 | 说明 |
| ---- | ---- |
| 序号 | |
| 注册日期 | 首次充值日期 YYYY-mm-DD |
| 老板编号 | 按注册日期自动生成顺序 |
| 老板昵称 | 支持手动修改 |
| 性别 | 支持手动修改 |
| 联系微信 | 支持手动填写 |
| 联系手机 | 支持手动填写 |
| 卡数量 | 充值详情自动同步，支持手动修改 |
| 累计充值 | 充值详情自动同步 |
| 累计赠送 | 充值详情自动同步 |
| 累计消费 | 充值详情自动同步 |
| 累计锁定 | 未完成结算订单折后总计自动同步 |
| 剩余余额 | 自动：累计充值+累计赠送-累计消费-累计锁定 |
| 推荐人 | 支持手动填写 |

#### 5.2.3 消费账户
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| 开户日期 | date | 默认首次充值日期 |
| 老板 | select | 选择后展开账户详情 |
| 账户详情（展开） | — | 开户日期、编号、昵称、类型、存单陪玩、折扣、充值/赠送/消费/锁定/余额合计、充值明细、消费明细 |
| 类型 | select | 散户/银卡/金卡/钻石卡，支持创建 |
| 存单陪玩 | select | 店存/陪玩列表 |
| 折扣 | select | 支持创建 |
| 充值合计 | auto | |
| 赠送合计 | auto | |
| 消费合计 | auto | |
| 锁定合计 | auto | |
| 余额合计 | auto | |
| 备注 | text | 客服手动填写 |

#### 5.2.4 充值详情（充值/开卡）

**新客户 - 开卡（会员）**
- 老板昵称（必填）
- 性别（必填：男/女）
- 联系微信
- 联系手机
- 推荐人（陪玩/老板列表，或手动）
- 账户类型（散户/银卡/金卡/钻石卡）
- 存单陪玩（店存/陪玩列表）
- 折扣（0.95/0.9/0.85/0.8/0.75/0.7/0.5）
- 充值方式（多选：支付宝/微信/其他）
- 充值金额（必填）
- 赠送金额（必填）
- 收款截图（jpg/png，≤10MB）

**新客户 - 不开卡（散户）**
- 同上，去除账户类型、存单陪玩、折扣

**老客户 - 不开卡（续存）**
- 搜索老板（必填）
- 选择账户
- 充值方式
- 充值金额
- 赠送金额
- 收款截图

**老客户 - 开卡**
- 搜索老板
- 账户类型
- 存单陪玩
- 折扣
- 充值方式
- 充值金额
- 赠送金额
- 收款截图

### 5.3 考核工作台

#### 5.3.1 陪玩名单
| 字段 | 说明 |
| ---- | ---- |
| 绑定陪玩登录 | 同步入职手机号 |
| 入职日期 | YYYYmmDD |
| 陪玩编号 | 按顺序自动生成 |
| 陪玩昵称 | 禁止重复，支持手动修改 |
| 性别 | 支持手动修改 |
| 陪玩状态 | 在职/离职/请假/黑名单 |
| 微信号 | 支持手动填写 |
| 手机号 | 支持手动修改 |
| 支付宝账号 | 支持手动修改 |
| 支付宝实名 | 支持手动修改 |
| 身份证 | 支持手动修改 |
| 陪玩提成 | 百分率 70%/80%/90%，支持创建 |
| 设置押金 | 200/300/500，支持添加 |
| 自缴押金 | 支持手动填写 |
| 单抵押金 | 自动：已完成单陪玩收入总和，限额=设置押金-自缴押金 |
| 订单收入 | 所有单折后总计 |
| 陪玩收入 | 所有单陪玩收入合计 |
| 已发放工资 | 已结算工资合计 |
| 详情入口 | 侧边展开：个人基本信息+所有派单详情+备注 |

#### 5.3.2 陪玩入职（注册申请）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| 申请日期 | date | 默认当前日期 |
| 陪玩昵称 | text | 必填，禁止与名单重复 |
| 性别 | select | 男/女 |
| 游戏类别 | multi-select | 三角洲/瓦/lol |
| 工作经验 | select | 没接触过/一年以内/一年以上 |
| 联系微信 | text | 必填 |
| 联系手机号 | text | 必填（登录账号） |
| 支付宝账号 | text | 选填 |
| 支付宝姓名 | text | 选填 |
| 身份证号 | text | 选填 |
| 设置密码 | password | 登录密码 |
| 重填 | button | 清空表单 |
| 提交 | button | |

#### 5.3.3 入职应聘
- 同步陪玩入职内容
- 审核状态：通过/不通过

### 5.4 财务工作台

#### 5.4.1 充值记录
- 同步充值详情全量数据

#### 5.4.2 提现记录
| 字段 | 说明 |
| ---- | ---- |
| 陪玩 | 陪玩名称 |
| 支付宝账号 | |
| 支付宝实名 | |
| 提现金额 | |
| 申请时间 | |
| 状态 | 待审核/已通过/已拒绝/已打款 |
| 处理客服 | |
| 备注 | |

#### 5.4.3 日常收支
- 日期范围筛选
- 类型：收入/支出
- 金额
- 说明
- 关联订单/充值记录

### 5.5 售后工作台

#### 5.5.1 售后投诉
| 字段 | 说明 |
| ---- | ---- |
| 订单编号 | 可选 |
| 日期 | 可选 |
| 陪玩昵称 | 可选 |
| 状态 | 已处理/待审核 |
| 罚款情况 | 已罚款/未罚款 |
| 具体内容 | 文本 |
| 备注 | 可传截图 |

#### 5.5.2 用户建议
| 字段 | 说明 |
| ---- | ---- |
| 老板编号 | 可选 |
| 日期 | 可选 |
| 具体内容 | 文本 |
| 备注 | 可传截图 |

---

## 六、陪玩端（网页端）

### 6.1 登录
- 手机号 + 密码登录
- 账号由管理后台客服创建

### 6.2 首页
- 个人信息概览
- 当前订单状态

### 6.3 派单情况
- 订单列表（按状态筛选）
- 订单详情（可填写具体时间、上传报单截图）

### 6.4 投诉情况
- 投诉记录及处理状态

### 6.5 工资查询
- 本周/本月/累计流水
- 本周/本月/累计收入
- 工资结算记录

### 6.6 总流水
- 按日期/游戏模式查看流水明细

---

## 七、老板端（网页端）

### 7.1 登录
- 手机号 + 密码登录
- 账号由管理后台客服导入

### 7.2 首页
- 累计充值总额
- 累计消费总额
- 剩余余额
- 账户状态（正常/已禁用）

### 7.3 订单明细
- 订单列表（按日期范围筛选）
- 订单详情：
  - 订单编号、日期、游戏类别、游戏项目
  - 陪玩昵称、时长、单价
  - 附加费用明细
  - 折扣率、折扣金额
  - 折后总价
  - 订单状态（已派单/待确认/已确认/已结算/已取消）
- 订单状态筛选（全部/进行中/已完成/已取消）

---

## 八、管理后台补充：老板账号管理

### 8.1 老板账号导入
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| 老板昵称 | text | 必填 |
| 性别 | select | 男/女 |
| 联系微信 | text | 必填 |
| 联系手机 | text | 必填（登录账号） |
| 密码 | password | 初始密码（客服可见） |
| 推荐人类型 | select | 陪玩/老板/无 |
| 推荐人 | select | 关联陪玩或老板 |

### 8.2 老板账号管理列表
| 字段 | 说明 |
| ---- | ---- |
| 老板编号 | BOSS+序号，自动生成 |
| 老板昵称 | |
| 性别 | |
| 联系微信 | |
| 联系手机 | |
| 累计充值 | 自动汇总 |
| 累计消费 | 自动汇总 |
| 余额 | 自动计算 |
| 状态 | 启用/禁用 |
| 操作 | 编辑/禁用/启用 |

---

## 八、技术约定

### 8.1 订单编号规则
格式：`YYYYMMDD + 4位序号`
例如：`202605230001`

### 8.2 陪玩编号规则
格式：`COMP + 4位序号`
例如：`COMP0001`

### 8.3 老板编号规则
格式：`BOSS + 4位序号`
例如：`BOSS0001`

### 8.4 抽成比例规则
| 单类型 | 公式 | 示例 |
| ---- | ---- | ---- |
| 正常单 | 折后总计 × 0.80 | |
| 礼物单/送单 | 折后总计 × 0.70 | |
| 预存九五折 | 折后总计 × 0.76 | |
| 预存九折 | 折后总计 × 0.72 | |
| 超时单（付2打3+） | 折后总计 × 0.70 | |

### 8.5 充值/消费自动计算
```
老板余额 = 累计充值 + 累计赠送 - 累计消费 - 累计锁定
```

---

## 九、项目模块优先级

### 第一期（核心闭环）
1. 数据库设计与初始化
2. Spring Boot 后端基础框架（鉴权、字典、基础CRUD）
3. 陪玩端：登录页 + 派单列表
4. 管理后台：客服工作台（派单订单） + 老板名单 + 充值详情
5. 管理后台：考核工作台（陪玩入职 + 陪玩名单）
6. 结算功能

### 第二期（完善功能）
7. 管理后台：店长工作台
8. 管理后台：财务工作台
9. 管理后台：售后工作台
10. 陪玩端：工资查询

### 第三期（高级功能）
11. 数据统计与看板
12. 消息推送集成
13. 积分系统
