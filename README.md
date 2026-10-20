# 餐饮业油烟在线大数据监控平台（前端）

这是一个基于 Vue 2 + Ant Design Vue 的餐饮业油烟监管前端项目，面向餐饮企业、排口测点、实时监测、超标告警、风险评估和运维管理等场景，主要用于构建“餐饮业油烟在线大数据监控平台”。该项目基于 Jeecg 体系的前端模板进行定制，采用前后端分离架构，并结合菜单权限、表格、地图/数据可视化组件与统一的后台接口调用方式。

## 项目定位

该系统覆盖了餐饮业油烟排放监管的典型业务流程，包括：

- 餐饮企业档案录入与管理
- 即时接入与测点注册
- 在线监测与历史数据查询
- 排放超标告警与风险评估
- 告警申诉、审核与报备审核
- 数据汇总、趋势分析和报表展示
- 运维工单、权限管理与系统维护

本仓库仅包含前端应用，实际运行需要配合后端接口服务。

## 核心功能

- 企业与测点管理
  - 餐饮企业录入、查询、列表展示、注册新企业
  - 监测点设备接入、编辑和查询

- 实时监控
  - 在线监测页面展示各测点状态、排放值和趋势
  - 数据地图、卡片、列表等多视图展示

- 告警与风险
  - 排放超标告警
  - 疑似故障告警与设备停机事件
  - 风险企业评估和标签管理

- 监管与审核
  - 告警反馈、申诉审核、报备申请审核
  - 监管人员工作台与处理流程

- 数据分析
  - 饼图、柱状图、曲线图等可视化统计
  - 设备联网率、数据上报量、企业接入趋势等分析

- 运维支持
  - 工单处理、系统监控和维护管理

## 技术栈

- Vue 2.6
- Vue CLI 3
- Ant Design Vue 1.x
- Vue Router / Vuex
- Axios
- ECharts / AntV / Viser-vue
- Less
- ESLint

## 目录结构

```text
.
├── public                 # 静态资源
├── src
│   ├── api                # 接口封装
│   ├── assets             # 图片、样式、静态资源
│   ├── components         # 公共组件
│   ├── config             # 全局配置、路由配置
│   ├── mixins             # 混入
│   ├── router             # 路由定义
│   ├── store              # Vuex 状态管理
│   ├── utils              # 工具函数
│   ├── views              # 业务页面
│   ├── App.vue
│   ├── main.js
│   └── permission.js
├── .env                  # 环境变量模板
├── .env.development      # 开发环境配置
├── .env.production       # 生产环境配置
├── babel.config.js
├── Dockerfile
├── package.json
├── vue.config.js
├── README.md
└── LICENSE
```

## 环境要求

- Node.js 12+（建议使用稳定版本）
- npm 或 yarn
- 兼容 Vue CLI 3 的前端构建环境

## 安装与运行

1. 安装依赖

```bash
npm install
# 或
yarn install
```

2. 启动开发服务器

```bash
npm run serve
# 或
yarn run serve
```

3. 构建生产包

```bash
npm run build
# 或
yarn run build
```

4. 代码检查

```bash
npm run lint
# 或
yarn run lint
```

## 环境变量

项目中使用了基础环境变量配置，主要包括后端接口地址，例如：

```env
NODE_ENV=development
VUE_APP_API_BASE_URL=http://localhost:8080/psms
VUE_APP_CAS_BASE_URL=http://cas.example.org:8443/cas
VUE_APP_ONLINE_BASE_URL=http://fileview.jeecg.com/onlinePreview
```

说明：

- `VUE_APP_API_BASE_URL` 为后端 API 基础地址
- 应与实际后端服务地址保持一致
- 生产环境请根据部署情况修改 `.env.production`

## 运行说明

- 本项目默认依赖后台接口服务，前端不能单独完成业务闭环
- 需先启动后端服务，并保证 API 地址与前端配置一致
- 项目使用了 `cross-env` 和 `NODE_OPTIONS=--openssl-legacy-provider`，在较新版本 Node 环境下可能需要注意兼容性

## 常用脚本

```json
{
  "scripts": {
    "serve": "cross-env NODE_OPTIONS=--openssl-legacy-provider vue-cli-service serve",
    "build": "cross-env NODE_OPTIONS=--openssl-legacy-provider vue-cli-service build",
    "build:test": "cross-env NODE_OPTIONS=--openssl-legacy-provider vue-cli-service build --mode test",
    "lint": "vue-cli-service lint"
  }
}
```

## 备注

- 这是一个融合监管业务流程与前端管理系统的定制化项目，不是通用模板示例
- 代码中包含大量业务模块，如企业录入、在线监测、告警审核、风险判定等
- 若要进行二次开发，建议优先确认后端接口协议和数据字段结构

## 相关资源

- [Vue 2 官方文档](https://cn.vuejs.org/v2/guide/)
- [Ant Design Vue](https://ant-design-vue.gitee.io/docs/vue/introduce-cn/)
- [Vue CLI](https://cli.vuejs.org/zh/)
- [Jeecg 低代码平台](https://github.com/zhangdaiscott/jeecg-boot)

如果你需要，我也可以继续把这个 README 改成更偏“企业级项目说明”或“中文部署手册”风格。