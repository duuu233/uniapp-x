# 澄镜智控模板架构

## 页面结构

```text
pages/
  home/index.uvue          首页工作台
  mine/index.uvue          我的
  album/index.uvue         我的相册
  device/bind.uvue         绑定设备
  device/list.uvue         我的设备
  profile/index.uvue       个人信息
  company/index.uvue       企业注册
  screen/index.uvue        投屏管理
  guide/index.uvue         操作指南
  settings/index.uvue      设置
  permissions/index.uvue   权限中心
```

首页与“我的”放在 `tabBar`，其余页面按业务域继续拆分，便于后续继续增加审批、消息、组织、日志等模块。

## 目录分层

```text
components/
  AppSection/              统一分区卡片
  MetricCard/              首页与个人中心指标卡

src/
  config/                  应用基础配置
  constants/               路由常量
  mock/                    模拟数据
  network/                 request 封装
  services/                页面可消费的业务接口层
  types/                   数据模型与空状态工厂
  utils/                   导航等通用能力
```

约定如下：

- 页面只负责加载、状态和渲染，不直接拼装接口 URL。
- `services` 负责描述业务接口，一个页面可以消费多个 service，但不直接碰 `uni.request`。
- `network/http.uts` 统一处理 baseURL、Mock 开关、响应解包和错误兜底。
- `mock` 与 `services` 一一对应，后续替换成真实后端时无需改动页面结构。
- `types/models.uts` 统一维护页面模型和默认空状态，避免每个页面重复写初始化逻辑。

## 请求层约定

当前模板默认 `useMock = true`。

后续接真实接口时：

1. 保持页面中的加载函数不变。
2. 在 `src/services/*` 中替换 `mockData` 为真实参数。
3. 继续复用 `src/network/http.uts` 统一处理鉴权、超时、业务错误码和日志。

## 分包结论

当前项目面向 Android、iOS、HarmonyOS 的 Uniapp x App 端，不落地 `pages.json` 的 `subPackages`。

原因：

- Uniapp x 的 `subPackages` 仅对小程序生效，App 端与 HarmonyOS 不支持。
- 当前页面数量和资源体量较小，强行做“伪分包”没有实际收益。
- 先把业务按目录和 service 分层整理好，后续如果扩到小程序端，再基于同一业务域目录映射到真正分包即可。

## 性能与扩展建议

- 保持首页只拉工作台摘要，设备列表、相册、投屏等详情页按需加载。
- 大图上传、压缩、断点续传建议单独抽 `upload` 模块，不和普通 request 混在一起。
- 权限检测、扫码、蓝牙发现建议做平台适配层，页面只关心结果。
- 设置项、投屏模板、设备标签建议后续改为配置化，避免写死在页面。
- 如果后续增加消息中心或审批流，继续沿用 `mock -> service -> page` 的依赖方向即可。
