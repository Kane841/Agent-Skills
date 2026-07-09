# 项目侦察细则

Phase 0 的操作指南。进入 Phase 1 前完成；结论写入分析摘要「项目约定」。

---

## 1. 工程结构

```bash
# Maven 多模块
find . -name "pom.xml" -maxdepth 3
# 或 Gradle
find . -name "build.gradle*" -maxdepth 3
```

记录：

- 根坐标、Java 版本
- **哪个模块含 `@SpringBootApplication`**（集成测通常放此模块或 sibling `-boot` 模块）
- 业务代码模块 vs 启动模块

---

## 2. 分层与命名

在目标 Controller 所在模块搜索：

```
grep -r "class.*Controller" --include="*.java" | head
```

打开 1～2 个 Controller，看注入类型：

| 常见模式 | 示例 |
|----------|------|
| Service 接口 + Impl | `UserService` / `UserServiceImpl` |
| Manager/Handler | `OrderManager`、`PaymentHandler` |
| 领域服务 | `domain.service.XxxDomainService` |
| 直连 Repository | 简单 CRUD 项目 |

**记录本项目实际用法**，后续文档与测试均用此称谓。

---

## 3. 统一响应与错误

搜索 `@ControllerAdvice` / `@RestControllerAdvice`：

- 成功响应包装类及字段（`data`、`result`、`payload`…）
- 失败时 HTTP 状态码与 body 结构（`error.code`、`errCode`、`message`…）
- 业务异常类名（`BusinessException`、`BizException`、`ServiceException`…）

打开一个已有 Controller 的返回语句，确认成功/失败 JSON 样例路径。

---

## 4. 鉴权与安全

检查：

- `spring.security` 依赖与 `SecurityFilterChain`
- 自定义 Filter / Interceptor（搜索 `OncePerRequestFilter`、`HandlerInterceptor`）
- 测试里如何绕过或携带 token（`@WithMockUser`、`@AutoConfigureMockMvc(addFilters = false)`、测试 header）

---

## 5. 持久化与造数

| 技术 | 侦察 |
|------|------|
| JPA | `@Entity`、`JpaRepository`、软删 `@SQLDelete` / `@Where` |
| MyBatis | `*Mapper.xml`、`*Mapper.java` |
| MyBatis-Plus | `BaseMapper`、`ServiceImpl` |
| 迁移 | `db/migration`、`flyway`、`liquibase` |

集成测造数：优先看现有测试用 **JdbcTemplate / @Sql / Repository / Testcontainers** 哪一种。

---

## 6. 已有测试风格（第二步 Spring 测试须对齐）

在 `src/test` 搜索与被测模块相关的测试：

```
grep -r "SpringBootTest\|WebMvcTest\|MockMvc\|TestRestTemplate\|WebTestClient" src/test
```

归纳表格：

| 项 | 本项目 |
|----|--------|
| 集成测基类/注解 | |
| HTTP 客户端 | |
| 是否 Mock 业务 Bean | |
| 造数/清数 | |
| 断言库 | AssertJ / Hamcrest |
| 命名 | `*IT` / `*IntegrationTest` / `*Test` |

**新写的测试应像「同一作者写的」**，而不是强行引入另一种栈。

---

## 7. 项目约定输出模板

写入分析摘要：

```markdown
## 项目约定（Phase 0）

| 项 | 值 |
|----|-----|
| 构建 | Maven / Gradle |
| 启动类 | `{全限定名}` |
| 集成测模块 | `{module}` |
| 业务层 | `{Service/Manager/...}` |
| 成功响应 | `{类型}` → JSON 路径示例 |
| 失败响应 | HTTP + `{error 字段路径}` |
| 鉴权头 | `{Header: 说明}` |
| 持久化 | JPA / MyBatis / … |
| 测试栈 | `{SpringBootTest + ...}` |
| 参考测试类 | `{同类已有测试}` |
```
