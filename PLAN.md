# eSportClub 实施计划

> 版本：v1.0 | 日期：2026-05-23

---

## 第一阶段：基础设施（第 1-2 天）

### 1.1 数据库初始化
**产出物：**
- [ ] `sql/init.sql` — 建库脚本，包含所有数据表 DDL
- [ ] `sql/dict.sql` — 字典表初始化数据（游戏类别/折扣/提成/押金等）

**步骤：**
1. 连接 MySQL 服务器（root/Wsd2026@Haber）
2. 创建数据库 `eSportClub`，字符集 utf8mb4
3. 按实体建表（admin / companion / boss / boss_account / recharge_record / order / salary_settlement / withdraw_record / complaint / dict_* 等）
4. 插入字典表初始数据

### 1.2 后端项目初始化
**产出物：**
- [ ] `backend/` — Spring Boot 项目

**步骤：**
1. 在 GitHub 创建 `eSportClub-backend` 仓库
2. 初始化 Spring Boot 项目（Maven，依赖：Spring Web / MyBatis-Plus / MySQL / JWT / Lombok）
3. 配置 `application.yml` 连接 eSportClub 数据库
4. 实现 JWT 鉴权逻辑（复用 wusidai 模式）
5. 统一响应结构 `Result<T>` + 全局异常处理
6. 推送 GitHub

### 1.3 前端项目初始化
**产出物：**
- [ ] `frontend-admin/` — React/Vue 管理后台项目
- [ ] `frontend-companion/` — React/Vue 陪玩端项目

**步骤：**
1. 创建 React 项目（Vite + Ant Design / Element Plus）
2. 配置路由结构
3. 配置 Axios 封装（带 JWT token）
4. 统一布局组件（侧边导航 + 顶部栏）
5. 推送 GitHub

---

## 第二阶段：陪玩端核心（第 3-4 天）

### 2.1 陪玩端登录页
**页面：**
- 统一登录页（手机号 + 密码）
- 路由：`/login` → 陪玩端首页

**API：**
- `POST /api/auth/login` — 登录（手机号+密码，返回 JWT）
- `GET /api/auth/me` — 当前用户信息

**步骤：**
1. 登录页 UI
2. 登录 API 对接
3. JWT 存储（localStorage）
4. 路由守卫

### 2.2 陪玩端首页
**页面：**
- 个人信息卡片（昵称/等级/状态）
- 当前接单数/流水/收入统计
- 快捷入口

### 2.3 派单列表
**页面：**
- 订单列表（状态筛选 + 日期筛选 + 搜索）
- 订单详情弹窗
- 填写具体时间（陪玩操作）
- 上传授单截图（陪玩操作）

**API：**
- `GET /api/companion/orders` — 我的订单列表
- `GET /api/companion/orders/{id}` — 订单详情
- `PUT /api/companion/orders/{id}` — 更新具体时间/截图

---

## 第三阶段：管理后台核心（第 5-8 天）

### 3.1 框架搭建
- 统一布局（侧边菜单 + 顶部栏）
- 菜单权限控制（店长/客服/考核官/财务）
- 统一表格 + 分页组件

### 3.2 客服工作台 — 派单订单
**页面：**
- 订单创建表单（含所有字段联动计算）
- 订单列表（管理视角，多条件筛选）
- 订单验收（确认/驳回）
- 订单编辑/取消

**API：**
- `POST /api/admin/orders` — 创建订单
- `GET /api/admin/orders` — 订单列表
- `PUT /api/admin/orders/{id}` — 更新订单
- `PUT /api/admin/orders/{id}/confirm` — 确认验收
- `PUT /api/admin/orders/{id}/reject` — 驳回（带原因）
- `PUT /api/admin/orders/{id}/settle` — 结算

**自动计算逻辑：**
```
订单总计 = (陪玩价格 + Σ附加费用) × 时长
折后总计 = 订单总计 × 折扣
陪玩收入 = 折后总计 × 陪玩提成
店内收入 = 折后总计 - 陪玩收入
```

### 3.3 客服工作台 — 老板名单
**页面：**
- 老板列表
- 老板详情（账户列表 + 充值/消费记录）

**API：**
- `GET /api/admin/bosses` — 老板列表
- `GET /api/admin/bosses/{id}` — 老板详情
- `PUT /api/admin/bosses/{id}` — 更新老板信息

### 3.4 客服工作台 — 消费账户
**页面：**
- 账户列表
- 账户详情（展开式）

**API：**
- `GET /api/admin/boss-accounts` — 账户列表
- `GET /api/admin/boss-accounts/{id}` — 账户详情

### 3.5 客服工作台 — 充值详情
**页面：**
- 充值记录列表
- 充值表单（新客户开卡/续存 + 老客户续存/开卡）

**API：**
- `POST /api/admin/recharges` — 创建充值记录（触发余额更新）
- `GET /api/admin/recharges` — 充值列表
- `PUT /api/admin/recharges/{id}/confirm` — 确认充值

**余额更新触发器：**
```
boss.total_recharge += amount
boss.total_gift += gift
boss.balance = total_recharge + total_gift - total_consume - total_locked
boss_account.recharge_total += amount
boss_account.gift_total += gift
boss_account.balance = recharge_total + gift_total - consume_total - locked_total
```

---

## 第四阶段：考核工作台（第 9-11 天）

### 4.1 陪玩入职
**页面：**
- 入驻表单
- 入驻申请列表

**API：**
- `POST /api/admin/companions/register` — 提交入驻申请（生成待审核账号）
- `GET /api/admin/companions/pending` — 待审核列表

### 4.2 入职应聘（审核）
**页面：**
- 待审核列表
- 审核详情
- 通过/驳回操作

**API：**
- `PUT /api/admin/companions/{id}/approve` — 审核通过（生成正式账号）
- `PUT /api/admin/companions/{id}/reject` — 审核驳回

### 4.3 陪玩名单
**页面：**
- 陪玩列表（在职/离职/请假/黑名单）
- 陪玩详情（侧边展开）
  - 基本信息
  - 派单记录
  - 备注
- 状态变更

**API：**
- `GET /api/admin/companions` — 陪玩列表
- `GET /api/admin/companions/{id}` — 陪玩详情
- `PUT /api/admin/companions/{id}` — 更新陪玩信息
- `PUT /api/admin/companions/{id}/status` — 变更状态

---

## 第五阶段：结算与工资（第 12-14 天）

### 5.1 工资结算
**规则：**
- 结算周期：每周一 00:00 结算上一周（周一 00:00 ~ 周日 24:00）
- 统计该陪玩周期内所有已验收订单
- 工资 = Σ(折后总计 × 提成率) - 罚款 + 奖励

**页面：**
- 结算周期选择
- 陪玩结算明细表
- 一键结算功能

**API：**
- `GET /api/admin/settlements/preview` — 结算预览（试算）
- `POST /api/admin/settlements` — 执行结算
- `GET /api/admin/settlements` — 结算历史

### 5.2 陪玩端工资查询
**页面：**
- 本周/本月/累计流水
- 工资明细
- 已结算记录

---

## 第六阶段：其他工作台（第 15-18 天）

### 6.1 财务工作台
- 充值记录（复用充值详情）
- 提现记录（列表 + 审核/打款）
- 日常收支统计

### 6.2 售后工作台
- 售后投诉（列表 + 处理 + 罚款）
- 用户建议（列表 + 备注）

### 6.3 店长工作台
- 客户数据看板（统计卡片 + 图表）
- 陪玩数据看板
- 管理名单（管理员 CRUD）

---

## 第七阶段：完善与部署（第 19-20 天）

### 7.1 数据字典管理
- 游戏类别/项目 CRUD
- 附加费用/折扣/提成/押金 CRUD

### 7.2 陪玩端完善
- 投诉记录查看
- 总流水查看

### 7.3 部署
- [ ] 后端部署到服务器（JAR）
- [ ] 前端编译打包（Nginx 静态托管）
- [ ] 域名解析（esport.wusidai.com 或独立域名）

---

## GitHub 仓库规划

| 仓库 | 内容 |
| ---- | ---- |
| `Haber8023/eSportClub` | 需求文档 + 项目协调（docs/PLAN.md） |
| `Haber8023/eSportClub-backend` | Spring Boot 后端 |
| `Haber8023/eSportClub-admin` | 管理后台前端 |
| `Haber8023/eSportClub-companion` | 陪玩端前端 |

---

## 里程碑

| 阶段 | 完成标准 |
| ---- | -------- |
| M1（第2天） | 数据库建好，后端能跑，前端项目初始化完成 |
| M2（第4天） | 陪玩端登录+派单列表可用 |
| M3（第8天） | 客服可完成完整派单+充值业务流程 |
| M4（第11天） | 陪玩入驻审核全流程通 |
| M5（第14天） | 工资结算功能上线 |
| M6（第18天） | 所有管理后台工作台上线 |
| M7（第20天） | 陪玩端完整 + 部署上线 |

---

## 技术栈汇总

| 层级 | 技术 |
| ---- | ---- |
| 数据库 | MySQL 8 + utf8mb4 |
| 后端 | Spring Boot 3.x + MyBatis-Plus + JWT |
| 管理后台前端 | React 18 + Vite + Ant Design 5 + Axios |
| 陪玩端前端 | React 18 + Vite + Ant Design 5 + Axios |
| 鉴权 | JWT（Bearer Token） |
| 构建 | Maven + npm |
| 部署 | Nginx（静态托管后端JAR运行） |
