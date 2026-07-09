# 输出模板

`{占位符}` 替换为 Phase 0 侦察后的 **项目实际值**。

---

## 项目约定（Phase 0，附在分析摘要开头）

见 [project-discovery.md](project-discovery.md) 第 7 节模板。

---

## 功能分析报告（第一步，**必须落盘**）

**路径**：`docs/test-analysis/{接口名}-analysis.md`  
**时机**：无论用户是否已指定测试方案，**先完成并落盘本文件**，再进入第二步。  
**不含**：`@Test` 代码、API Client 断言脚本（留待第二步）。

```markdown
# {接口名称} 功能分析报告

> 生成日期：{date}  
> 目标：{Controller}.{method}  
> 测试方案：**待第二步确认**

## 功能说明

{2～4 段：接口职责、业务规则、成功/失败语义、与上下游关系}

## 项目约定（Phase 0）

| 项 | 值 |
|----|-----|
| 构建 | Maven / Gradle |
| 启动类 | `{全限定名}` |
| 业务层 | `{Service/Manager/...}` |
| 成功响应 | `{类型}` → JSON 路径示例 |
| 失败响应 | HTTP + `{error 字段路径}` |
| 鉴权头 | `{Header: 说明}` |

## 基本信息

| 项 | 值 |
|----|-----|
| Controller | `{类全名}` |
| 方法 | `{methodName}` |
| HTTP | `{METHOD} {完整路径}` |
| 业务入口 | `{实际业务类}.{method}()` |

## 调用链

1. Controller.{method}
2. …
3. {Repository/Mapper/Client}

## 场景分类汇总（S / F / B）

| 分类 | 数量 | 编号列表 |
|------|------|----------|
| S 成功 | {n} | S1, S2, … |
| F 失败 | {n} | F1, F2, … |
| B 边界 | {n} | B1, … |
| **合计** | **{N}** | 共 {N} 个场景 |

## 分支与边界矩阵（全量）

| 编号 | 分类 | 场景 | 触发条件 | 期望 HTTP | 响应断言路径 | 副作用 | 价值 | HTTP可达 | 建议测试层级 |
|------|------|------|----------|-----------|--------------|--------|------|----------|--------------|
| S1 | S | | | | | | 高 | 是 | integration / manual |
| F1 | F | | | | | | 高 | 是 | |
| B1 | B | | | | | | 中 | 是 | |

## 明确跳过的场景（须说明原因，不得静默省略）

| 编号 | 原因 |
|------|------|

## 不可通过 HTTP 覆盖的分支

| 编号 | 分类 | 场景 | 建议交付 |
|------|------|------|----------|
| F3 | F | … | business `@Test` |

---

## 场景 → 用例覆盖对照表（**第二步完成后回填**）

| 编号 | 分类 | 场景摘要 | 交付形式 | 用例名 / md 节 | 断言 | 状态 |
|------|------|----------|----------|----------------|------|------|
| S1 | S | | _待第二步_ | | | ⬜ |
| F1 | F | | _待第二步_ | | | ⬜ |

---

## 请求示例

### URL

~~~
{METHOD} {baseUrl}{path}
~~~

**Path / Query**（按项目实际填写）

**请求头**

| Header | 示例值 | 必填 | 说明 |
|--------|--------|------|------|
| Content-Type | application/json | POST/PUT/PATCH 时 | |
| {项目鉴权头} | {示例} | | |

### 请求体 — 详细版（全字段）

~~~json
{
  "fieldA": "value"
}
~~~

（字段说明表）

### 请求体 — 精简版

**场景**：{新建成功 / 分页查询 / …}

~~~json
{
  "fieldA": "value"
}
~~~

**省略说明**

| 省略字段 | 原因 |
|----------|------|
```

---

## Phase 3 片段（仅当需单独引用时）

以下为分析报告内「请求示例」章节的简版，完整版见上一节「分析报告」。

---

## 集成测试类骨架

按 Phase 0 择 **A 真实 HTTP** 或 **B MockMvc**（或项目已有第三种）。以下为常见 A；B 见同文件末尾。

### A. TestRestTemplate / WebTestClient + RANDOM_PORT

```java
package {测试包名};  // mirror Controller 或对齐已有集成测

import {项目启动类全名};
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.*;
import org.springframework.jdbc.core.JdbcTemplate;  // 若项目用 JdbcTemplate 造数
import org.springframework.test.context.TestPropertySource;  // 若项目测试需覆盖配置

/**
 * {@code {METHOD} {path}} 集成测试。
 * 场景覆盖清单：S1 → {methodName}；F1 → …
 */
@Tag("integration")  // 若项目使用
@SpringBootTest(
        classes = {ApplicationClass}.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = {
        // 仅填本项目测试类已用的 properties，勿复制其他项目
})
class {Resource}{Action}IntegrationTest {

    private static final String API_PATH = "{path}";
    private static final String FIXTURE_PREFIX = "{模块缩写}-IT-";

    @Autowired private TestRestTemplate restTemplate;
    @Autowired private ObjectMapper objectMapper;
    // @Autowired private JdbcTemplate jdbcTemplate;  // 按需

    @BeforeEach void setUp() { purgeFixtures(); }
    @AfterEach  void tearDown() { purgeFixtures(); }

    /**
     * 场景 S1：{描述}
     * 前置 / 操作 / 期望 / 矩阵编号
     */
    @Test
    @DisplayName("[{action}]成功：{中文}")
    void {action}_success_{detail}() throws Exception {
        // Arrange → Act → Assert（HTTP + 响应 JSON 路径 + 可选落库）
    }

    private HttpHeaders apiHeaders() {
        HttpHeaders h = new HttpHeaders();
        h.setContentType(MediaType.APPLICATION_JSON);
        // {Phase 0 鉴权头}
        return h;
    }

    private void purgeFixtures() { /* 项目惯用清数方式 */ }
}
```

### B. MockMvc（仅当 Phase 0 确认项目如此做集成测）

```java
@SpringBootTest(classes = {ApplicationClass}.class)
@AutoConfigureMockMvc
class {Resource}{Action}IntegrationTest {

    @Autowired private MockMvc mockMvc;
    @Autowired private ObjectMapper objectMapper;

    @Test
    void {action}_success_{detail}() throws Exception {
        mockMvc.perform(post(API_PATH)
                        .headers(apiHeaders())
                        .content("""
                                {"field":"value"}
                                """))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.{项目data路径}").exists());
    }
}
```

---

## 手动测试文档（第二步 · 方案 B · API Client）

**路径**：`docs/manual-test/{接口名}-manual-test.md`  
**前置**：第一步 `docs/test-analysis/{接口名}-analysis.md` 已存在；用户已确认 `manual` 及具体 API Client。

```markdown
# {接口名称} — 手动测试方案

## 文档信息

| 项 | 值 |
|----|-----|
| 分析报告 | [../test-analysis/{接口名}-analysis.md](../test-analysis/{接口名}-analysis.md) |
| API Client | **{Postman / Bruno / Apifox / …}** |
| Base URL | `{baseUrl}` |
| 鉴权 | `{Phase 0 头}` |

## 场景待办（须与矩阵编号 **1:1**，按 S → F → B 分组）

### S — 成功路径
- [ ] **S1** {一句话}
- [ ] **S2** …

### F — 失败路径
- [ ] **F1** …
- [ ] **F2** …

### B — 边界路径
- [ ] **B1** …

---

### S1 — {场景标题}

**分类**：S | **编号**：S1

**请求**

~~~
{METHOD} {{baseUrl}}{path}
{Headers}

{body 精简版}
~~~

**期望**

- HTTP {status}
- `{JSON 路径}` = …

**断言脚本（{API Client 名称}）**

<!-- Postman 示例 -->
~~~javascript
// Postman Tests
pm.test("status", () => pm.response.to.have.status({code}));
const json = pm.response.json();
pm.test("{业务断言名}", () => pm.expect(json.{path}).to.eql("{expected}"));
~~~

<!-- Bruno 时改用下方，勿混用 pm -->
<!--
~~~javascript
// Bruno Tests
test("status", function () {
    expect(res.getStatus()).to.equal({code});
});
const json = res.getBody();
test("{业务断言名}", function () {
    expect(json.{path}).to.equal("{expected}");
});
~~~
-->

---

### F1 — {场景标题}

**分类**：F | **编号**：F1

（结构同 S1：请求 + 期望 + **完整断言脚本**；失败场景须断言 error 路径）

---

## 场景覆盖对照表（文档末尾，便于查缺补漏）

| 编号 | 分类 | 场景摘要 | md 节 | 有请求 | 有断言脚本 | 状态 |
|------|------|----------|-------|--------|------------|------|
| S1 | S | | §S1 | ✅ | ✅ | ⬜ |
| F1 | F | | §F1 | ✅ | ✅ | ⬜ |
| B1 | B | | §B1 | ✅ | ✅ | ⬜ |

**合计**：矩阵 {N} 个场景，本文 {n} 节，待补 {N-n} 个。
```

断言语法对照：[api-client-assertions.md](api-client-assertions.md)。**只保留用户所选客户端的一种脚本**，删除模板中注释掉的其它工具示例。

### REST Client 用户

无脚本 → **人工核对清单**：

```markdown
**人工核对**
- [ ] Status: 200
- [ ] Body `data.id` 非空
```

---

## 场景覆盖清单（测试类 Javadoc / 类注释用）

| 编号 | 分类 | 场景摘要 | 测试方法 | 断言要点 | 状态 |
|------|------|----------|----------|----------|------|
| S1 | S | | `{methodName}` | HTTP 200 + data | ✅ |
| F1 | F | | `{methodName}` | 404 + error.code | ⬜ |
| B1 | B | | `{methodName}` | 空列表 total=0 | ⬜ |
