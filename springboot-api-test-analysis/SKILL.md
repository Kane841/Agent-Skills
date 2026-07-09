---
name: springboot-api-test-analysis
description: >-
  分析任意 Java Spring Boot 项目 Controller 及业务层：两步交付——先落盘功能分析报告（理解、
  S/F/B 分支边界、请求示例），再询问并确认测试方案后交付 Spring 测试代码或 API Client 用例 md。
  用户指出 Controller、要求分析接口、梳理分支或编写测试时使用。
disable-model-invocation: false
---

# Spring Boot 接口分析与测试方案

用户会 **指出 Controller 层接口**（路径、类名或粘贴代码）。Agent 必须 **先侦察项目、再读代码、再分析**，禁止套用固定包结构或臆造字段/分支。

## 两步交付（硬性顺序，不可合并）

```
┌─────────────────────────────────────────────────────────┐
│ 第一步：功能分析报告（无论用户是否已说测试方案，都必须先完成）      │
│   侦察 → 读代码 → 理解功能 → S/F/B 场景 → 请求示例 → 落盘 md   │
│   ↓ 暂停：告知报告路径，询问/确认测试方案                      │
├─────────────────────────────────────────────────────────┤
│ 第二步：测试方案交付（用户确认方案后再做）                      │
│   springtest → 集成测/业务层测 Java 代码                      │
│   api client → 确认具体 Client → 测试用例 md + 断言脚本        │
└─────────────────────────────────────────────────────────┘
```

**禁止**：用户一说「写集成测试」就跳过分析报告直接写代码。  
**禁止**：在第一步未完成或未落盘时进入第二步。  
**允许**：用户第一步只要分析、不要测试 — 交付报告后即可结束。

即使用户在首条消息已提到测试形式，也须 **先完成第一步**；第二步开始前 **再次确认** 方案（用户可 reaffirm 或更改）。

---

## 核心原则

1. **S / F / B 分类**：每个场景标 **S**（成功）、**F**（失败）、**B**（边界），编号 `S1`、`F1`、`B1`…
2. **全量列举**：矩阵穷尽有测试价值的分支，禁止只写样例
3. **第一步只做分析**：报告含功能理解、分支边界、请求示例；**不含**可执行测试代码/断言脚本（对照表「测试交付」列留空或标「待第二步」）
4. **第二步 1:1 交付**：矩阵每个编号（除已跳过）对应用例 — `@Test` 或 manual 小节 + **完整断言**
5. **可查缺补漏**：第二步完成后回填 **场景覆盖对照表**

---

## 触发与输入

| 输入 | 第一步 | 第二步 |
|------|--------|--------|
| **目标接口** | ✅ 必读 | ✅ |
| **项目路径** | ✅ | ✅ |
| **测试形式** | ❌ 不前置依赖；用户提了也先只做分析 | ✅ **须确认**：`integration` / `business` / `manual` |
| **API Client** | ❌ | ✅ `manual` 时 **必问** |
| **可选** | 接口文档、SQL、已有测试类 | 自定义 md/测试类路径 |

---

# 第一步：功能分析报告

内部步骤：`Phase 0 侦察 → Phase 1 读代码 → Phase 2 分析 → Phase 3 请求示例 → 落盘`

## Phase 0：项目侦察

每个项目必做。从仓库归纳约定，写入报告「项目约定」小节。细则见 [project-discovery.md](project-discovery.md)。

| 侦察项 | 记录内容 |
|--------|----------|
| 构建与模块、启动类 | 集成测常放哪个模块 |
| 分层命名、DTO 位置 | Service/Manager 等实际用法 |
| 统一响应、异常与错误码 | JSON 路径、HTTP 映射 |
| 鉴权与请求头 | 测试时如何携带 |
| 持久化、已有测试风格 | 造数方式、MockMvc 或 TestRestTemplate 等 |

## Phase 1：阅读 Controller 与业务层

### Controller

路径/方法、入参校验、返回值、委托的业务类与方法。

### 业务层（沿实际调用链）

```
Controller → {业务类} → {Repository|Mapper|Client} → {Entity|外部 API}
```

关注：主流程、分支、异常、事务、状态机、约束、上下文、边界。

**完成标准**：能复述调用链，并识别可归入 **S / F / B** 的分支。

## Phase 2：功能理解与分析

### 2.1 功能说明（报告必备）

用 prose 说明：**接口做什么、入参/out 语义、关键业务规则、与上下游关系**。读者未读代码也应能理解功能意图。

### 2.2 执行流程

有序步骤或 mermaid flowchart。

### 2.3 场景分类（S / F / B）

| 前缀 | 含义 |
|------|------|
| **S** | Success — 合法输入下的预期成功 |
| **F** | Failure — 校验失败、不存在、无权限、冲突等 |
| **B** | Boundary — null/空/极值/状态临界等 |

编号 `{前缀}{序号}`，同接口内连续不跳号。报告含 **分类汇总表**（S/F/B 数量与编号列表）。

### 2.4 分支与边界矩阵（全量）

| 编号 | 分类 | 场景 | 触发条件 | 期望 HTTP | 期望响应要点 | 持久化副作用 | 价值 | HTTP可达 |
|------|------|------|----------|-----------|--------------|--------------|------|----------|

- 穷尽有测试价值场景；跳过项单独表说明原因
- HTTP 不可达分支标注「建议 business 层测」
- **第一步不写** 具体测试方法名；可选列「建议测试层级」：`integration` / `business` / `manual`

### 2.5 入矩阵优先级

主成功 → 不存在/无权限 → 校验/业务失败 → 冲突与无副作用 → 可见边界 → 跳过无价值分支

## Phase 3：请求示例（并入分析报告）

每个接口 **必须** 含：

1. **URL**：Method、path、query、path 变量、**项目实际请求头**
2. **BODY 详细版**：DTO 全字段 + 说明
3. **BODY 精简版**：主成功场景（S1）最少必填字段 + 省略说明

GET/DELETE 无 body 时用参数详细版/精简版替代。

## 第一步落盘与暂停

| 项 | 约定 |
|----|------|
| **路径** | `docs/test-analysis/{接口名}-analysis.md`（用户可指定） |
| **内容** | Phase 0～3 全部内容；**不含** `@Test` 代码与 API Client 断言脚本 |
| **聊天回复** | 功能摘要 + **报告路径** + **询问测试方案** |

**暂停话术示例**：

> 功能分析报告已写入 `{path}`，共 {N} 个场景（S {n} / F {n} / B {n}）。  
> 请选择第二步测试方案：  
> 1. **Spring 测试** — 集成测试（`integration`）或业务层单测（`business`）  
> 2. **API Client 手动测试**（`manual`）— 请一并说明工具（Postman / Bruno / Apifox / …）  
> 3. **暂不编写测试**，仅保留分析报告  

模板：[templates.md](templates.md)「功能分析报告」。

---

# 第二步：测试方案交付

**前置条件**：第一步报告已落盘；用户已确认测试形式（及 manual 时的 API Client）。

## 方案 A：Spring 测试

用户选 `integration` 或 `business`（或统称 springtest）。

### A1 — `integration` 集成测试

HTTP → 业务层 → 持久化全链路。对齐 Phase 0 已有测试风格（TestRestTemplate / WebTestClient / MockMvc 等）。

- 矩阵每个 HTTP 可达 S/F/B 编号 → 至少 1 个 `@Test`
- `@DisplayName` 含编号（如 `[S1][save]成功：…`）
- 注释：编号、分类、前置、操作、期望
- HTTP + 响应断言；写操作加落库校验（若项目惯例）
- 类注释附 **场景覆盖对照表**
- 若有仓库专用集成测 skill，读取并叠加

类骨架：[templates.md](templates.md)。运行：`mvn -pl {module} test "-Dtest={Class}"` 或 Gradle 等价命令。

### A2 — `business` 业务层测试

不经过 HTTP。规范：[business-layer-test.md](business-layer-test.md)。

- 矩阵中「仅 business」及 HTTP 不可达场景 → `@Test`
- Mock 下游依赖；断言项目实际异常/返回值

## 方案 B：API Client 手动测试

用户选 `manual`。

### B1 — 确认 API Client（未说明则必问）

> 你使用哪种 API Client？（Postman / Bruno / Apifox / Insomnia / REST Client / 其他）

语法对照：[api-client-assertions.md](api-client-assertions.md)。**禁止**未确认就写 Postman 脚本。

### B2 — 测试用例 md

**路径**：`docs/manual-test/{接口名}-manual-test.md`

1. 链接第一步分析报告
2. 文档头：API Client、Base URL、鉴权
3. 场景待办：S / F / B 分组，与矩阵 **1:1**
4. **每编号一节**：请求 + 期望 + **该 Client 语法的完整断言脚本**（REST Client 用核对清单）
5. 末尾 **场景覆盖对照表**；完成后 **回填** 分析报告中的对照表

模板：[templates.md](templates.md)「手动测试文档」。

## 第二步完成标准

- 矩阵 ↔ 用例 1:1（除已声明跳过）
- 覆盖对照表已回填至分析报告（状态 ✅/⬜）
- Spring 测试：已运行或说明阻塞原因

---

## 执行清单

```
【第一步 — 必须完成并落盘后再暂停】
- [ ] Phase 0 项目约定已记录
- [ ] Phase 1 已读 Controller → 业务层 → 持久化
- [ ] Phase 2 功能说明 + S/F/B 汇总 + 全量矩阵
- [ ] Phase 3 URL + BODY 详细版 + 精简版
- [ ] docs/test-analysis/{接口名}-analysis.md 已写入
- [ ] 已告知用户报告路径并询问/确认测试方案
- [ ] 第一步未写 @Test 代码、未写 API Client 断言脚本

【第二步 — 用户确认方案后】
- [ ] 测试形式已确认（integration / business / manual）
- [ ] manual：API Client 已确认，断言语法正确
- [ ] 每个 S/F/B 编号均有对应用例 + 断言
- [ ] 覆盖对照表已回填分析报告
- [ ] Spring 测试已运行或说明阻塞原因
```

---

## 附加资源

- 输出模板：[templates.md](templates.md)
- 业务层单测：[business-layer-test.md](business-layer-test.md)
- 项目侦察：[project-discovery.md](project-discovery.md)
- API Client 断言：[api-client-assertions.md](api-client-assertions.md)
