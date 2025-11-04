# 餐厅点餐小程序（微信云开发）

一个基于微信小程序 + 云开发的单店点餐系统。最小闭环：扫码入桌 → 浏览菜单 → 加入购物车 → 下单 →（可选）评价。支持管理员管理菜品与厨师端扩展。

## 功能概览
- 顾客端：扫码入桌、菜单浏览、购物车、确认下单、（可选）评价
- 管理端：账号登录、菜品新增/改价/上架下架、图片上传（云存储）
- 后厨端：厨师登录（V2：订单流转与看板）
- 云函数：二次验价、特色菜限量（事务扣减）、顾客入库、评价

## 技术栈
- 微信小程序（Skyline/基础库≥2.25）
- 微信云开发（云数据库/云函数/云存储）
- Git 协作（GitHub 分支+PR 流程）

## 目录结构（建议）
- pages/
  - index/ 菜单点餐
  - cart/ 购物车
  - order-confirm/ 确认下单
  - review/ 评价（可选）
  - admin/dishes/ 管理端-菜品管理
  - chef/dashboard/ 后厨看板（V2）
  - login/ 登录（管理员/厨师）
- components/
  - navigation-bar/ 自定义导航栏
- cloudfunctions/
  - createOrder/ 下单（验价+限量扣减）
  - ensureCustomerLogin/ 顾客入库（扫码）
  - loginByPassword/ 账号密码登录（admin/staff）
  - getUserRole/ 读取角色（staff）
  - adminUpsertDish/ 新增或更新菜品
  - adminSetDishStatus/ 上/下架
  - updateDishPrice/ 改价
  - addReview/ 评价（可选）
- docs/ 协作文档（schema/env/backlog）
- scripts/seed/ 示例数据（categories.json、dishes.json）

## 数据库集合（V1）
- admin：{ username, password(或passworld), openid? }
- staff：{ role:'admin'|'chef', account, password, status, openid? }
- customers：{ openid, tableId, visitCount, createdAt, lastSeenAt }
- categories：{ name, sort, status }
- dishes：{ name, categoryId, price, pics[], status:'on'|'off', specialtyQuota? }
- orders：{ tableId, userOpenId, items[], amount, orderStatus, createdAt, remark }
- reviews（可选）：{ orderId, dishId, star, text, userOpenId, createdAt }
- chefs（V2 展示/扩展）：{ name, level, specialties[] }

## 云函数清单（需部署）
- createOrder：后端二次验价；按 specialtyQuota 事务扣减；写入 orders
- ensureCustomerLogin：按 openid 建/更 customers，绑定 tableId
- loginByPassword：
  - 管理员：admin 集合（username + password/passworld）
  - 厨师：staff 集合（role+account+password+status:on）
  - 首次登录绑定 openid
- getUserRole：从 staff 读取当前 openid 的 role（默认 customer）
- adminUpsertDish：新增/更新菜品（名称/价格/图片/分类/状态）
- adminSetDishStatus：上/下架
- updateDishPrice：改价
- addReview（可选）：写入 reviews

## 快速开始（本地与云开发）
1. 云环境
   - 微信开发者工具 → 右上角“云开发”选择环境（建议每人一个 dev 环境）
   - 在 app.js 固定环境：`wx.cloud.init({ env: '你的环境ID' })`
2. 部署云函数（每个函数都需要）
   - 右键函数 → 云端安装依赖 → 上传并部署（使用当前环境）
   - 若失败，改本地安装再部署：
     ```bash
     cd cloudfunctions/<fnName>
     npm i --production --registry=https://registry.npmmirror.com
     ```
3. 初始化数据
   - 新建集合：admin, staff, customers, categories, dishes, orders（reviews/chefs 可后加）
   - 导入示例数据：开发者工具 → 数据库 → 导入 scripts/seed/*.json
4. 账号准备
   - 管理员：admin：`{ username:'admin001', password:'123456' }`
   - 厨师：staff：`{ role:'chef', account:'chef001', password:'123456', status:'on' }`
5. 运行与验证
   - 管理页：/pages/admin/dishes/index（未登录会跳至 /pages/login/index）
   - 顾客扫码：/pages/index/index?tableId=T01 → 加购 → 确认下单

## 分工与流程（三人建议）
- 成员A（顾客端）：pages/index|cart|order-confirm|review，金额/样式优化
- 成员B（管理端）：pages/admin/dishes，新增/改价/上下架、图片上传；V2 分类管理
- 成员C（云函数&后厨）：createOrder/login/ensureCustomerLogin/admin*，后续订单流转与实时推送

### Git 分支
- main（可发布）、dev（集成）、feature/<owner>-<topic>（个人开发）
- 提交流程：feature → PR 到 dev → 评审 → 合并 → 阶段性合并到 main

## 开发路线（里程碑）
- M1：闭环可用（扫码→下单→管理）与云函数稳定部署
- M2：管理端完善（分类管理、菜品编辑、限量配置、导入导出）
- M3：后厨订单流转与实时看板；顾客“我的订单/详情/评价列表”

## 安全与规范（V1 最低要求）
- 管理操作必须走云函数并在后端用 OPENID 鉴权
- 下单金额后端二次验价，关键写入使用事务或幂等键
- 前端统一 rpx 与安全区（env(safe-area-inset-bottom)），固定底部需留 padding

## 常见问题排查
- cloud.callFunction: fail -504002
  - 函数未部署到当前环境或 env 不一致 → 固定 env，重新“云端安装依赖 → 上传并部署”
- 缺少 wx-server-sdk
  - 对每个函数安装依赖（云端/本地），再部署
- 管理员登录失败
  - admin 集合字段名需匹配：username + password（或 passworld，已在函数兼容）

## License
此仓库仅用于教学/内测，可据此二次开发。

