---
title: "Ontology语义证据接入的易复发陷阱"
author: codex
date: 2026-11-18
tags: [python, troubleshooting, api, testing, security]
session: { platform: codex, sessionId: 01a1d9de-8fab-75f1-9976-ae16a6d36c3c }
---

# Ontology语义证据接入的易复发陷阱

适用：将已有业务接入对象工作台、World只读工具、历史证据复核和一跳关联。
结论：原生返回中的值、当前对象身份、当前授权和集合完整性必须分别证明。
新增模块先核当前测试服源码、原生DTO及权限链，再复用本清单；不要照搬旧字段或旧测试数量。
本文是脱敏团队经验，不是业务发布记录，也不证明既存业务的自动写生命周期全部完成。

## 查源码的稳定位置

- 测试后端根：`my-cloud:/home/zkkgoa/frappe-bench/apps/zkkg_ai/zkkg_ai`，下文称 `AI/`。
- 项目后端根：`my-cloud:/home/zkkgoa/frappe-bench/apps/zkkg_project_management/zkkg_project_management`，下文称 `PM/`。
- 测试前端根：`root@8.153.202.222:/var/www/html/code/src`，下文称 `FE/`。
- 领域与接口：`AI/foundation/world_todo_ontology.py`、`AI/api/todo_ontology.py`。
- 语义与证据：`AI/foundation/world_semantic_intent.py`、`world_query_evidence.py`、`agent_world_capabilities.py`、`agent_world_runtime.py`。
- 目录与原生读取：`AI/foundation/world_office.py`、`world_todo_tools.py`；原生实现见 `AI/api/todo_items.py`、`todo_reports.py`、`todo_cockpit.py`、`department_plans.py`。
- 项目链：`PM/services/projects.py`、`plans.py`；共享界面：`FE/pages/ERP/ontology/OntologyWorkspace.jsx`。

## 1. 源Link值不等于关联目标档案

- 触发：源记录含 `project_id`、执行人部门、前置事项编号等Link值。
- 错误行为：把该值直接升级为目标对象，或用源记录的可见性证明目标可读。
- 根因：字段只证明源记录保存过一个引用，不证明目标仍存在、匹配或当前有权读取。
- 最小修复：按固定登记关系重读源，取真实Link，再调用目标原生入口，核类型、精确name及来源范围；目标拒绝时整条关系不可用。
- 必留回归：源可读而目标无权、目标已变更、错误来源、无Link合法空；拒绝响应不得泄露目标字段或存在性。

## 2. 精确name详情被UI自动page污染

- 触发：列表页复用查询构造器，用户填写精确系统编号后仍自动加 `page`。
- 错误行为：详情请求携带分页及列表筛选，被严格schema拒绝，或详情界面继续显示分页。
- 根因：把列表的默认输入当成所有查询的通用输入，忽略详情的互斥契约。
- 最小修复：精确name走详情允许的参数集，不自动附加分页；UI按已返回的对象详情状态隐藏列表分页。
- 必留回归：name详情与翻页列表分别命中正确原生入口，切换后不残留旧page/filters，非法混参仍被后端拒绝。

## 3. 原生先分页、权限后过滤时，空页不等末页

- 触发：原生报告列表先取一页，再按当前原生权限逐条过滤。
- 错误行为：过滤后0行就关闭续翻，或把过滤前数量宣称为授权总数。
- 根因：当前页可见条数与原生后续是否仍有候选记录是两个事实。
- 最小修复：保留 `continuation unknown` 语义和继续查看后页能力；沿登记分页/扫描上限，按schema最大页限制停止翻页，不把达到上限解释为完整遍历，也不伪造完整总数。
- 必留回归：首个可见页为空而后页有合法记录、连续空页、真实末页、到达上限；拒权不能记成合法空。

## 4. 跨resource时旧view和filters污染目标

- 触发：从成果配置等资源点击关联，目标是待办或另一种成果对象。
- 错误行为：沿用来源 `view`、局部filters或旧父ID，覆盖目标登记的selector，漏掉 `required_scope`。
- 根因：将整个页面输入对象透传，未区分共享业务范围与来源资源专属参数。
- 最小修复：只带明确允许的共享范围，再应用目标descriptor的selector和必需范围；对象inputs来自本次原生实际选择。
- 必留回归：config→items、config→latest、不同父todo、不同部门周期；目标请求精确包含所需参数，未知参数继续拒绝。

## 5. 父ID或“集合仅1行”不等单对象

- 触发：查询携带todo父ID，或当前集合恰好返回一行。
- 错误行为：把父事项当子对象身份，或将任意单行集合升级为精确详情证据。
- 根因：唯一性必须来自登记的对象类型与作用域，不能由样本行数倒推。
- 最小修复：仅对声明 `unique_in_scope` 的类型，结合真实typed ref、实际row与已核scope定位；get_object重新走原生链并核精确name。
- 必留回归：0/1/多行子集合、错type、错row、错scope、过期latest；无真实name的投影始终不生成ref。

## 6. DTO.status=ok不是业务status

- 触发：传输DTO和业务行都使用 `status` 字段。
- 错误行为：回答“事项已完成”却依据顶层 `status=ok`，或给缺失业务状态补一个默认值。
- 根因：运输成功元数据与白名单业务值未隔离。
- 最小修复：顶层status仅解释读取结果；业务状态只取当前原生行/对象values中的登记字段，原生缺失即省略。
- 必留回归：DTO成功但业务未完成、业务状态缺失、empty、not_available；语义证据不得把运输状态当业务事实。

## 7. as_of采集时间变化不等权限变化

- 触发：驾驶舱历史证据重读跨过一秒，当前采集时间自然更新。
- 错误行为：业务值和权限均未变，却将整个证据判为撤权或失效。
- 根因：对完整DTO做机械相等比较，把采集时钟纳入授权/业务指纹。
- 最小修复：仅对 `todos.cockpit` 排除其原生采集时间差异；仍重新执行当前原生授权并比较业务值，不将例外推广到其他字段或资源。
- 必留回归：只跨秒仍可复核、业务字段变化应失效、当前权限撤销应拒绝；日期/周期/selection变化不得被时间例外吞掉。

## 8. raw.allowed_tools标记不等展开后的可执行目录

- 触发：原始配置含目录或授权marker，查询related能力时只看到某个元数据标记。
- 错误行为：据marker推断目标工具已获授权，或只检查related入口而跳过源、目标读工具。
- 根因：混淆原始声明、展开后的目录和本次调用授权三层含义。
- 最小修复：以当前展开授权结果检查related工具本身、源resource工具及目标resource工具；执行时两端仍走原生记录授权。
- 必留回归：只有related元数据、缺源工具、缺目标工具、全授权；缺grant必须在业务读取前失败，不能降级拼接旧证据。

## 9. 只读接口和合法空必须有严格边界

- 触发：将业务API包装成通用读取RPC，前端统一catch异常后显示“暂无数据”。
- 错误行为：接受任意method/doctype/actor，透出整段原生DTO，或把HTTP错误、参数错和拒权压成空集合。
- 根因：适配层没有承担输入校验、来源限定、输出白名单和错误分类。
- 最小修复：RPC仅GET与当前session；严格schema拒未知参数/重复键/非有限数等；仅调登记原生入口、投影源字段白名单；HTTP失败保留错误语义。
- 必留回归：POST、伪造actor、错type/scope、畸形JSON、原生拒绝/超时；敏感内部字段不透出，只有有权且原生确实无数据才empty。

## 10. typed字段不能为显示去重而丢类型

- 触发：多个对象类型都有 `name/status`，前端把字段拼成列后重复显示。
- 错误行为：在后端全局按name去重，令其他type的字段契约消失，或界面保留重复同名列。
- 根因：对象类型的字段定义与单次表格的展示列被混为一体。
- 最小修复：后端每个object_type内按顺序去重字段name并保留各type；前端显示列按name去重，中文label只改展示映射。
- 必留回归：跨type同名字段均保留、同type无重复、界面无重复列；标签调整不改变schema、scope、原生权限和运行函数。

## 11. 草稿筛选不能冒充已应用范围

- 触发：用户修改日期/部门/周期但未查询，原生又可能采用默认可读部门或归一化周期起点。
- 错误行为：证据区显示表单草稿，或显示请求值代替原生真实selection，让旧结果看起来属于新范围。
- 根因：草稿输入、已应用请求和本次原生实际范围共用一个展示来源。
- 最小修复：证据与结果绑定本次响应的真实日期、部门、周期、视角及scope说明；对象inputs同样取实际范围；草稿编辑不覆盖已应用范围。
- 必留回归：改筛选未提交、原生默认部门、月/季起点归一化、切资源、取消旧请求；窄屏仍能辨认真实范围。

## 复用时保留的证据边界

- 本次为场景3；业务源码和试点配置留在测试服，未更新生产或 zkkgoa/qifuaiqd 业务仓库。新模块不能据此声称已发布或三方核验。
- 工作台新增动作均为manual_only，回原生页面办理；原有Office写工具及确认链继续存在，不能声称自动写闭环全完成。
- 部门计划非空正向样本来自原生保存后的未提交事务并已回滚，不是持久HTTP/World非空证明。
- 历史列表是当前原生授权合法空，没有现存非空历史样本；不能用结构通过推导历史内容已全面验证。
- 当次测试数量仅是当次证据，不作为未来固定基线；新增模块按当前差异补最小回归，并验证当前真实页面/API与权限。

## 归档依据与查证入口

- 最终报告：`my-cloud:/test/org-ontology/20261118-cjx-mbp/REPORT.md`；验收索引：同目录 `FINAL-ACCEPTANCE.json`，设计阶段状态以最终报告纠正。
- 修复公件：`my-cloud:/test/org-ontology/org-ontology-20261118-cjx-mbp/fix-p2-20261118T0104/fix-public.json`。
- 标签/typed字段公件：`my-cloud:/test/org-ontology/org-ontology-20261118-cjx-mbp/labels-20261118T0113/labels-public.json`；上述重复org前缀为实际归档路径。
- 关系授权证据：`my-cloud:/test/org-ontology/20261118-cjx-mbp/native-acceptance/final-related-world-public.json`，含直接related、真实模型及仅元数据授权拒绝。
- 远端修改前快照：`my-cloud:/test/backups/org-ontology-20261118-cjx-mbp/`；查实现以以上当前测试服稳定源码路径为准，快照不代替当前源码核验。
