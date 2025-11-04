概述
目标：基于微信小程序+云开发的单店点餐系统。最小闭环：扫码入桌→浏览菜单→加购→下单→（可选）评价。
技术栈：微信小程序（Skyline/基础库≥2.25）、微信云开发（云函数/云数据库/云存储）。
设计原则：先跑通闭环，再扩展；强约束数据模型；接口最少化；严格分工与 Git 流程。
功能范围（V1）
顾客端：扫码入桌、菜单浏览、加入购物车、下单。
管理端：管理员登录、菜品增删改、上/下架、图片上传。
后厨端：厨师登录（先保留入口与权限，V2 再做订单流转）。
云函数：下单二次验价、管理员鉴权（按 OPENID）、顾客入库、评价（可选）。
目录结构（建议）
pages/ 小程序页面
index/ 菜单点餐
cart/ 购物车
order-confirm/ 确认下单
admin/dishes/ 菜品管理（增删改/上下架/图片）
chef/dashboard/ 厨师看板（V2扩展）
login/ 登录（管理员/厨师）
components/ 公共组件（如自定义导航栏）
cloudfunctions/ 云函数
createOrder/ 下单与限量校验
ensureCustomerLogin/ 顾客入库（扫码）
loginByPassword/ 账号密码登录（admin/staff）
getUserRole/ 角色读取（staff）
adminUpsertDish/ 新增/更新菜品
adminSetDishStatus/ 上/下架
updateDishPrice/ 改价
addReview/ 提交评价（可选V1-）
docs/ 协作文档（schema/env/backlog）
scripts/seed/ 示例数据（categories/dishes）
数据库集合（V1必需）
admin：{ username, password(或passworld), openid? }
staff：{ role:'admin'|'chef', account, password, status, openid? }
categories：{ name, sort, status }
dishes：{ name, categoryId, price, pics[], status:'on'|'off', specialtyQuota? }
orders：{ tableId, userOpenId, items[], amount, orderStatus, createdAt, remark }
customers：{ openid, tableId, visitCount, createdAt, lastSeenAt }
（选）reviews：{ orderId, dishId, star, text, userOpenId, createdAt }
云函数清单（V1）
createOrder：后端二次验价；按 specialtyQuota 事务扣减；写入 orders。
ensureCustomerLogin：按 openid 建/更 customers，绑定 tableId。
loginByPassword：管理员查 admin(username+password|passworld)；厨师查 staff(role+account+password+status:on)；首次绑定 openid。
getUserRole：从 staff 读取当前 openid 的 role，未命中默认 customer。
adminUpsertDish：管理员新增/更新菜品（名称/价格/图片/分类/状态）。
adminSetDishStatus：管理员上/下架菜品。
updateDishPrice：管理员改价。
（选）addReview：写入 reviews。
环境与部署
打开开发者工具 → 右上角“云开发”选择环境（建议每人一个 dev 环境）。
在 app.js 固定环境：wx.cloud.init({ env: '你的环境ID' })。
对每个云函数：右键 → 云端安装依赖 → 上传并部署（使用当前环境）。
导入示例数据：数据库 → 导入 scripts/seed/categories.json、scripts/seed/dishes.json。
常见错误：
-504002：函数未部署或环境不一致 → 固定 env、重新部署。
缺少 wx-server-sdk：对每个函数“云端安装依赖”或本地 npm i --production 后再部署。
开发路线（执行顺序）
初始化项目与环境
创建云环境；在 app.js 固定 env。
新建集合：admin/staff/customers/categories/dishes/orders（reviews可晚点加）。
管理员与厨师登录（先跑通）
建 admin 文档：{ username:'admin001', password:'123456' }
建 staff 文档（厨师）：{ role:'chef', account:'chef001', password:'123456', status:'on' }
部署 loginByPassword、getUserRole，前端登录页调通并回跳。
管理端菜品管理
adminUpsertDish、adminSetDishStatus、updateDishPrice 部署。
pages/admin/dishes/：新增菜品（名称/价格/图片上传fileID）、改价、上/下架 → 列表刷新。
顾客端闭环
ensureCustomerLogin 部署，首页解析 ?tableId=T01 时调用。
createOrder 部署（先不接支付），下单成功清空购物车。
体验修缮
统一 rpx 与安全区；列表底部为固定栏留 padding；格式化金额显示。
分工建议（三人组）
A 顾客端与体验
pages/index|cart|order-confirm，金额/展示优化；（选）pages/review。
里程碑：跑通加购→下单闭环；优化样式与格式化。
B 管理端
pages/admin/dishes、图片上传、改价、上下架、（V2）分类管理。
里程碑：跑通新增菜品→首页可见→上下架/改价生效。
C 云函数与权限
部署与维护：createOrder/ensureCustomerLogin/loginByPassword/getUserRole/admin*。
里程碑：完成管理员鉴权（按 OPENID）、下单验价与限量扣减。
Git 流程
分支：main（发布）、dev（集成）、feature/<owner>-<topic>（个人开发）。
提交流程：本地 feature → PR 到 dev → 评审（不提交 node_modules/对齐 schema）→ 合并 → 阶段合入 main。
PR 检查：函数名=目录名；每个函数安装 wx-server-sdk；前端不直接写库；管理员操作必须走云函数并鉴权。
V2 展望（可后续排期）
管理端：分类增删改、菜品编辑页、特色菜每日限量重置任务、批量导入导出。
后厨端：订单流转（new/doing/done）、实时看板、催单处理。
顾客端：我的订单/详情、再次评价入口、手机号绑定、订阅消息。
安全：密码哈希（bcryptjs）；getUserRole 兼容 admin 集合；所有管理函数二次鉴权与操作日志。
提交到 GitHub 的建议
首次提交：当前 README + 基础目录（空页面+函数目录+docs+scripts）。
新建 Issues：按“开发路线与分工”拆分为 8-12 个可完成任务，指派到 A/B/C。
建 Project（看板）：ToDo / In Progress / Review / Done，PR 关联 Issue。
有需要我可以把这份 README 直接细化为你仓库首页版（含最小命令与页面入口路径清单），并生成一组建议的 GitHub Issues 模板（标题+描述+验收标准）。
