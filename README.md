# Dinney-System
Create a dinner system
# 餐厅点餐小程序（微信云开发）

## 项目简介
- 顾客：扫码入桌 → 浏览菜单 → 加购 → 下单 → 评价
- 管理：菜品新增/改价/上架下架（图片上传云存储）
- 后厨：厨师列表与特色菜余量（后续扩展订单流转）
- 云开发：云函数二次验价、特色菜限量事务扣减、顾客入库、评价等

## 目录结构
- pages/ 顾客、管理、厨师、小程序页面
- components/ 公共组件（自定义导航栏）
- cloudfunctions/ 云函数（Node.js）
- docs/ 协作文档（schema、env、backlog）
- scripts/seed/ 示例数据（可在开发者工具导入）

## 快速开始
1. 打开微信开发者工具，选择云开发环境（如 cloud1-xxxx）
2. 修改 app.js → wx.cloud.init({ env: '你的环境ID' })
3. 部署云函数：在云函数面板依次“云端安装依赖”→“上传并部署（使用当前环境）”
4. 导入示例数据：开发者工具 → 数据库 → 导入 scripts/seed/*.json
5. 运行小程序，使用示例账号登录管理/厨师端

## 云函数列表（需部署）
- createOrder：下单验价 + 特色菜限量扣减（事务）
- ensureCustomerLogin：扫码入桌创建/更新顾客
- loginByPassword：管理员(admin集合)与厨师(staff集合)登录
- getUserRole：启动识别角色（staff）
- updateDishPrice：改价
- adminUpsertDish：新增/更新菜品
- adminSetDishStatus：上/下架
- addReview：评价写入

## 常见问题
- -504002：函数未部署到当前环境或 env 不一致 → 写死 env、重新部署
- 缺少 wx-server-sdk：对每个函数“云端安装依赖”，或本地 npm i --production 后部署
- 管理登录失败：确认 admin 集合字段名（username + password/passworld），已在函数兼容

更多细节见 docs/ 目录。
