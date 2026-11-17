---
title: ERP与组织效率Ontology的模块复用接入模板
author: codex
date: 2026-11-18
tags: [python, api, pattern, testing, security]
session: { platform: codex, sessionId: 01a1d9de-8fab-75f1-9976-ae16a6d36c3c }
---

## 背景

**本经验对应场景3测试服验收；正式环境及 zkkgoa/qifuaiqd 业务仓库未更新。团队经验发布不等于业务发布。**
ERP对象工作台已复验，组织效率基于现有World与原生业务接口接入对象、关系、指标和动作说明。
可复用的是登记、适配、授权复核、证据和UI边界；后续模块必须先核对当前测试服源码，不能照搬旧快照。
这里的Ontology是业务对象与读取契约，不意味着已有任意对象写入、自动流程编排或Palantir产品的完整能力。
本次组织效率基线为15个资源、36个子视图、12条一跳关系、20个唯一manual_only动作、新增自动写动作0。
这些数量属于本次覆盖分母；新模块按自身原生入口重新盘点，不能固定套用20项动作或宣称全系统已覆盖。

## 可复用做法

### 1. 先盘原生能力，再登记对象

为每个模块建立清单：原页面/按钮、原生API、查询参数、对象类型、关系、指标、写入生命周期与权限入口。
每项登记“已有、缺失、合法空、拒权、未验收”，不能用目录里有按钮证明底层行为已验收。
把真实对象和查询投影分开：对象必须有原生精确类型与系统编号；统计、趋势、部门分布通常只是投影。
Link、姓名、标题、日期和聚合行ID都不能自行升级为目标档案，也不能任意get_doc/get_all补齐缺失能力。
只适配原生返回的字段白名单；不新建业务表、业务副本、并行权限模型或第二套执行引擎。

### 2. 复用World、原生RBAC与确认链

复用agent_world_runtime的语义/读取链、world_semantic_intent、world_query_evidence、agent_world_capabilities。
模块层只做静态登记、严格输入验证、原生DTO适配和领域关系；原生角色、记录权限、公司/部门范围仍是授权来源。
入口测试名单只开放试点入口，不授予业务权限；当前用户来自真实Session，不能接受actor替身。
查询、对象详情、关系每次点击和World历史证据都重新读取原生权限；撤权后清理旧列表、详情与历史可重放数据。
既有Office写工具继续原预览、确认、变参/异对象与版本检查；只读工作台的动作说明不新增写授权。
本次manual_only表示返回原生页面办理；20项是当前组织效率动作基线，原有6个Office写工具继续独立受控。

### 3. 对象引用必须携带来源与选择器

服务端登记typed ref、required_scope和selector；客户端只传登记允许的输入，不接收doctype/method/任意Link路径。
required_scope约束“从哪个来源对象或周期才能读此对象”，selector区分同资源下的配置、最新版本等视图。
例如以下均为虚构值，真实字段与枚举由模块登记决定：

```json
{"ref":{"type":"DEMO Result Version","name":"DEMO-VERSION-001"},"inputs":{"source":"DEMO-ITEM-001","view":"latest"}}
```

例中descriptor可登记required_scope=[source]、selector={view:latest}；取详情须重走源读取链并核精确name。
生成对象引用时透传真正生效的部门/周期/来源；点击关联时仅带目标登记参数和共享业务范围，禁止整包合并旧行inputs。
原生选择了默认部门或当前周期时，证据与UI显示实际selection，不把用户请求值当最终生效范围。

### 4. 五个GET RPC与DTO

| RPC | 责任与返回边界 |
|---|---|
| catalog | 返回当前用户可见资源、严格schema、对象descriptor、子视图、关系、动作与指标定义 |
| query | 返回items/objects/projections及evidence；仅真实对象生成ref，投影不能伪装单对象 |
| get_object | 按typed ref与required_scope重读原生来源，只返回精确对象及白名单字段 |
| related | 固定一跳relation_id，返回当前授权目标、分页与证据；无真实身份的目标不返回ref |
| coverage | 普通用户只见其可见目录状态计数；内部完整覆盖分母单独留manifest验收 |

DTO保留contract_version、status、resource、source_path、evidence；详情值放object.values，不把DTO状态当业务状态。
evidence至少表达source_path、inputs、returned、native_source、scope_note、completeness与实际范围/采集时间。
只支持GET与真实登录态；拒绝未知参数、重复JSON键、非对象JSON、NaN/Infinity、超长输入和伪造类型/来源。
输入schema用additionalProperties=false，filter有中文title，枚举/默认/必填明确；原生不支持的筛选不得静默丢弃。
合法零记录是empty；拒权/找不到统一not_available，不泄露目标ID、存在性或半段详情，不能把拒绝包装成空结果。

### 5. 关系、指标与证据都要保留分母

关系只允许服务端登记的一跳；先原生授权源，再逐个原生授权目标，不能凭源中的ID数组证明目标可读。
World发布与调用同时要求关系工具、源读工具、目标读工具的grant；有related或follow-user不能绕过任一端。
子对象需验证父来源；项目、成果等跨域目标重走原生项目/成员授权；latest引用不得借机回读任意旧版本。
目标集合中任一对象拒权或不可核实时采用fail-closed，不回传失败目标或数量；每页重新授权，不缓存旧权限身份。
只有原生可信total才能表示全量；页内条数、权限过滤后的页、截断集合和样本都需明确标注各自范围。
过滤后空页仍可能有后页；has_more保留原生可证含义，不能用过滤前总数冒授权总数，也不把样本累加为全量。
指标登记单位、时间范围、时区、分子/分母、币种与完整性；0分母率为null，不用0冒充已达成或未达成。
保留原生十进制字符串；不同币种/单位不直接合计，换算需另有明确口径与来源；当前记录日期分布不是历史快照。
同名显示列保留真实列身份与中文标签；缺失字段省略，不用null/空串补造事实。

### 6. UI内容层复用，Session按业务域接入

复用ERP的OntologyWorkspace内容层，注入API factory、basePath、homePath、source validator和preview component。
新模块用薄wrapper接入，保留ERP默认行为；组织效率使用主应用Session/API，不套ensureErpSession或ERP特殊代理。
主应用Session与ERP桥接是两种接入边界，按现有业务域确认后复用；不能因UI共用就搬用另一域登录或授权逻辑。
原页面继续办理业务；新增入口仅进入工作台，不应在导航时顺带确认、提交、发通知或触发其他状态变化。
请求取消保留AbortError；切资源、切账号或撤权时清旧数据；详情面板或弹层按其交互支持Escape关闭并恢复焦点，窄屏不造成页面横向溢出。

### 7. 各层文件触点

后端根`AI/`为`my-cloud:/home/zkkgoa/frappe-bench/apps/zkkg_ai/zkkg_ai/`，以下7个产品文件是本次可参考触点：

| 路径 | 实施者应核查的职责 |
|---|---|
| AI/foundation/world_todo_ontology.py | 领域目录、对象/投影、DTO、关系、动作与指标白名单 |
| AI/api/todo_ontology.py | 五GET、Session/入口门、严格输入与统一拒绝 |
| AI/foundation/world_todo_tools.py | Office读工具映射与既有调用兼容 |
| AI/foundation/world_office.py | 模块目录、tool_only与读取授权 |
| AI/foundation/world_semantic_intent.py | 业务别名与意图，不由模型扩张实际范围 |
| AI/foundation/agent_world_capabilities.py | 工具能力、执行及当前证据重授权 |
| AI/foundation/world_query_evidence.py | 范围、来源、人员与完整性证据 |

共享运行链另核`AI/foundation/agent_world_runtime.py`；原生权限与DTO仍回todo_items/todo_reports/todo_cockpit/department_plans及项目服务核验。
后端对应测试为`AI/tests/test_todo_ontology{,_api,_native,_world}.py`，按纯逻辑、边界、原生、World层分别证明。
前端根`FE/`为`root@8.153.202.222:/var/www/html/code/`。
共享层核`FE/src/pages/ERP/ontology/{OntologyWorkspace.jsx,ontologyModel.mjs}`及模型/交互测试。
模块层核`FE/src/services/{todoService.js,todoOntologyService.js}`、`FE/src/pages/TodoItems/TodoOntologyWorkspace.jsx`及服务测试。
入口核`FE/src/pages/{TodoItems.jsx,TodoDashboard.jsx}`、`FE/src/layouts/WorkspaceRouteView.jsx`、`FE/src/App.jsx`。

## 步骤

1. 只读核当前测试服源码、全量原生入口和原生RBAC；建立资源/子视图/对象/关系/指标/动作manifest及缺口表。
2. 写模块接入契约和字段映射；逐项选择复用触点，确认旧资源ID/schema/调用兼容与共享ERP回归范围。
3. 按当前场景备份源码及必要配置；最小增加模块登记、GET适配、World映射和UI薄wrapper，不复制执行引擎。
4. 对新增非平凡逻辑验证严格输入、错type/scope/父源、未知目标、角色边界、双端grant、撤权及旧证据重放拒绝。
5. 同一合法用户同参数逐值对照原生DTO；manifest每项分别记录非空、合法空、拒绝和未取得样本，不能互相顶替。
6. 在CLI阶段完成纯测试、原生对照、真实模型只读场景与权限复核，全部通过后再做Ego；检查分页、精确详情、一跳关系、实际范围、撤权清空及共享ERP。
7. 场景3前端先读本机`~/.codex/scene3-device-id`，用该固定编号执行测试服`/usr/bin/node /var/www/html/code/scripts/deployScene3Preview.mjs --device <本设备编号>`。
8. 比对测试服源码hash、冻结输入/个人served与真实HTTP；留模型请求账、页面证据、权限恢复及业务未意外推进证据。
9. 收口只报测试服验收范围；场景4必须另获已验收结果的明确确认，从已验收测试服源码发布，再完成正式/测试/GitHub三方核验。

## 下一个模块的完成标准

- [ ] 盘点本模块实际资源、视图、关系、指标和动作分母；按对象/投影/manual_only分别登记。
- [ ] 修改前备份源、必要配置和数据库；校验SHA、压缩完整性及表集合，写明附件/非事务表/恢复演练边界。
- [ ] CLI先过：纯逻辑与权限测试、同用户同参数原生逐值对照、真实模型及历史证据复核；合法空不冒拒绝。
- [ ] 只读使用现有strict connection/原生授权；不以一个Python flag代替数据库只读约束，不泛放初始化器或业务DML。
- [ ] 新建样本有唯一标识和精确journal，原生清理后独立核对；不可原生删除的数据只事务内回滚并注明证据层级。
- [ ] 临时入口/业务范围最小授权、限时恢复；恢复精确值与角色/User Permission指纹，不只核对条数。
- [ ] 长CLI任务后台保存log与exit；保留资源上限，按正向用例pattern分组，不能把进程超时当产品断言失败。
- [ ] Ego读当前skill，复用登录与同一任务空间；Antd Select点可见容器，不等待隐藏输入“可见”。
- [ ] 窄屏在同一脚本设置viewport、导航、等待启动遮罩真正消失、截图并视觉复核，DOM就绪不等于页面可见。
- [ ] 记录新HTTP模型账及RED→复测，不复制旧次数上限，也不混淆题数与真实请求数。
- [ ] 新Python在实际Web/相关空闲worker加载；核PID/启动时间与源码mtime，bench execute不等于Web已生效。
- [ ] 个人预览输入、所改源文件、served及真实HTTP一致；Ego最后撤权重载清旧数据，独立finish关闭任务空间。
- [ ] 回查既存业务关键字段与样本0残留；报告只说已验范围，另获确认后才能进入正式发布。

## 边界与来源

本次只读验收不覆盖既存真实事项、成果、付款等业务写生命周期；不能把manual_only动作表解释成写入验收通过。
部门计划非空正向仅在原生保存后的未提交事务内验证并回滚，未证明持久HTTP/World非空；历史仅授权合法空，无既存非空样本。
最终页面矩阵与最后受影响范围复验分开留账，不能把早期候选全量结果冒充最终版本全量重跑。
本经验以以下测试服最终报告与验收索引为查证入口；早期设计的“未实施”状态由最终报告纠正。
测试后端远端归档为`my-cloud:/test/org-ontology/20261118-cjx-mbp/{REPORT.md,FINAL-ACCEPTANCE.json}`；原生专件在同根`native-acceptance/`。
测试前端远端归档为`root@8.153.202.222:/test/org-ontology/20261118-cjx-mbp/frontend/{REPORT.md,FINAL-ACCEPTANCE.json,FINAL-frontend-current-public.json}`；历史路径后缀仅定位证据。
已记录的测试服耐久备份位于`my-cloud:/test/backups/org-ontology-20261118-cjx-mbp/`；该历史路径中的设备后缀只定位本次证据，不是后续设备配置示例。
后续接入先核源码与证据是否仍当前；正式环境不能由本机、旧知识或GitHub直接补源，发布按场景规则从已验收测试服推进。
本条只沉淀已验证的接入模式与验收边界，不构成业务发布或新增权限。
