# API Client 断言语法对照

模式 C（`manual`）编写断言脚本时，**必须**按用户确认的客户端选用对应语法。  
勿把 Postman 脚本交给 Bruno 用户，反之亦然。

---

## 确认项（写入手动测试文档头部）

```markdown
| 项 | 值 |
|----|-----|
| API Client | Postman / Bruno / Apifox / Insomnia / REST Client / 其他 |
| 断言语法 | 见本文「{Client}」节 |
```

用户未指定时 **先询问**，不可默认 Postman。

---

## Postman

Tests 标签页；Chai 风格 `pm.expect`。

```javascript
pm.test("HTTP 200", function () {
    pm.response.to.have.status(200);
});

const json = pm.response.json();

pm.test("业务字段", function () {
    pm.expect(json.data.id).to.be.a("string");
    pm.expect(json.data.name).to.eql("expected");
});

// 失败场景
pm.test("error code", function () {
    pm.expect(json.error.code).to.eql("NotFound");
});
```

常用：`pm.response.code`、`pm.response.headers.get("X-Request-Id")`、`pm.environment.set("token", json.data.token)`。

---

## Bruno

Tests 面板；`test()` + `expect()` + `res` 对象（**非** `pm`）。

```javascript
test("HTTP 200", function () {
    expect(res.getStatus()).to.equal(200);
});

const json = res.getBody();

test("业务字段", function () {
    expect(json.data.id).to.be.a("string");
    expect(json.data.name).to.equal("expected");
});

// 失败场景
test("error code", function () {
    expect(json.error.code).to.equal("NotFound");
});
```

常用：`res.getStatus()`、`res.getHeader("content-type")`、`bru.setVar("token", json.data.token)`（变量名以当前 Bruno 版本为准）。

---

## Apifox

后置操作 → 断言 / 自定义脚本。国内版多数兼容 **Postman `pm` API**：

```javascript
pm.test("HTTP 200", function () {
    pm.response.to.have.status(200);
});
const json = pm.response.json();
pm.test("业务字段", function () {
    pm.expect(json.data.id).to.exist;
});
```

若用户说明为「Apifox 新版断言 UI」而非脚本，则改写成 **断言规则表格**（字段路径、比较符、期望值），并在文档注明「在 Apifox 断言 tab 配置，非脚本」。

---

## Insomnia

Tests 标签；Insomnia 内置 **Chai** + `response` 对象：

```javascript
const body = JSON.parse(response.body);

test("HTTP 200", () => {
    expect(response.status).toBe(200);
});

test("业务字段", () => {
    expect(body.data.id).toBeTruthy();
});
```

语法因 Insomnia 版本略有差异；用户指明版本时以其文档为准。

---

## REST Client（.http / VS Code）

**无统一 Test 脚本**。交付改为：

1. **人工核对清单**（Status、关键 JSON 字段）
2. 可选：配套 **Postman/Bruno 集合** 仅当用户另有该客户端

示例：

```markdown
**人工核对**
- [ ] Status: 200
- [ ] Body `data.id` 非空
- [ ] Body `data.name` = "expected"
```

---

## 其他 / 未列出客户端

1. 请用户提供该工具的 **官方断言/Tests 文档链接** 或一段现有脚本样例  
2. 对齐其 API 后再写脚本  
3. 在手动测试 md 顶部注明工具名与参考文档

---

## 脚本块标注规范

手动测试 md 中，每个场景的断言须：

1. 代码块语言标记与工具一致（`javascript` 通用；Bruno 仍用 javascript 但注释首行标明 `// Bruno Tests`）
2. 代码块**上方**一行写明：`断言脚本（{Client 名称}）`
3. 一段脚本只服务**一个场景**，便于复制到对应请求
