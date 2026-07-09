# 业务层级测试规范

被测对象：**Controller 实际调用的业务类**（Service、ServiceImpl、Manager、Handler 等）。  
目标：不启动 Web 层，验证 Phase 2 矩阵分支，尤其 HTTP 不可达逻辑。

---

## 技术选型（默认 + 项目对齐）

| 项 | 默认 | 项目有先例时 |
|----|------|--------------|
| 框架 | JUnit 5 + Mockito | 跟现有 `*Test` |
| 扩展 | `@ExtendWith(MockitoExtension.class)` | 或 `@SpringBootTest` + `@MockBean` |
| Mock 对象 | 下游 Repository / Mapper / Client | 只 Mock **直接依赖** |
| 断言 | AssertJ | 项目已用 Hamcrest 则一致 |

**不 Mock** 被测类 internal private 方法（经 public API 覆盖）。

**静态上下文**（SecurityContext、TenantHolder 等）：`mockStatic` 或测试替身 — 按项目现有做法；无先例则优先测不依赖上下文的 branch，或在注释标明需集成测补充。

---

## 放置位置

```
{含被测类的模块}/src/test/java/{mirror 包路径}/{被测类名}Test.java
```

命名：跟项目 — `{ClassName}Test` / `{ClassName}UnitTest`；避免与集成测 `{ClassName}IT` 冲突。

---

## 异常断言（按 Phase 0 项目类型）

| 项目形态 | 断言方式 |
|----------|----------|
| 自定义业务异常 + 错误码枚举 | `assertThrows(XxxException.class, …)` + 断言 code/enum 字段 |
| `ResponseStatusException` | 断言 `getStatusCode()` |
| 返回 `Result` 失败而非抛异常 | 断言返回值 flag/code，不用 assertThrows |

**勿假设** `BizException`、`DasErrorCode` 等名称 — 从代码读取。

---

## 类骨架（泛型示例）

```java
package com.example.order.service;

import com.example.order.dto.OrderCreateRequest;
import com.example.order.entity.Order;
import com.example.order.exception.OrderNotFoundException;
import com.example.order.repository.OrderRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

/**
 * {@link OrderService} 单元测试。
 * 场景：S1 创建成功、F1 重复单号、F2 订单不存在 …
 */
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    /** 场景 S1：合法请求 → 持久化并返回 id */
    @Test
    @DisplayName("[create]成功：返回新订单 id")
    void create_success_returnsOrderId() {
        OrderCreateRequest req = new OrderCreateRequest();
        req.setOrderNo("ORD-001");
        // … 必填字段

        when(orderRepository.existsByOrderNo("ORD-001")).thenReturn(false);
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> {
            Order o = inv.getArgument(0);
            o.setId(100L);
            return o;
        });

        Long id = orderService.create(req);

        assertThat(id).isEqualTo(100L);
        verify(orderRepository).save(any(Order.class));
    }

    /** 场景 F1：单号重复 → 项目定义异常 */
    @Test
    @DisplayName("[create]失败：单号重复")
    void create_failsWhenOrderNoDuplicate() {
        OrderCreateRequest req = new OrderCreateRequest();
        req.setOrderNo("DUP");

        when(orderRepository.existsByOrderNo("DUP")).thenReturn(true);

        assertThrows(DuplicateOrderException.class, () -> orderService.create(req));
        verify(orderRepository, never()).save(any());
    }

    /** 场景 F2：更新时 id 不存在 */
    @Test
    @DisplayName("[update]失败：订单不存在")
    void update_failsWhenNotFound() {
        when(orderRepository.findById(999L)).thenReturn(Optional.empty());

        assertThrows(OrderNotFoundException.class,
                () -> orderService.update(999L, new OrderCreateRequest()));

        verify(orderRepository, never()).save(any());
    }
}
```

将 `OrderService`、`DuplicateOrderException` 等替换为 **目标项目实际类名**。

---

## Mock 策略

| 场景 | Mock 行为 |
|------|-----------|
| 资源不存在 | `findById` → `Optional.empty()` / `getById` → `null` |
| 软删/状态不符 | 返回带禁用/删除标记的 entity，或 exists 查询为 true |
| 唯一性冲突 | `exists*` → `true` |
| 分页 | 填充 `Page`/`IPage` 的 records 与 total |
| 写成功 | `save`/`insert` 回调设置 id；`ArgumentCaptor` 校验字段 |
| 不应写入 | `verify(repo, never()).save(any())` |
| 外部 HTTP/RPC | Mock Feign Client / RestTemplate Bean |

---

## 与集成测试分工

| 类型 | 集成测 | 业务层测 |
|------|--------|----------|
| `@Valid` / 参数绑定 | ✅ | ❌ |
| 业务规则与异常 | ✅ 优先 | ✅ 补 HTTP 不可达 |
| 真实 SQL / 事务 | ✅ | ❌ |
| 复杂 Mock 链 | ❌ | ✅ |

方法命名与 `@DisplayName` 与集成测保持一致：`{动作}_{情景}_{期望}` + **`[S1]`/`[F1]`/`[B1]` 编号前缀** + 中文场景说明。

**全量覆盖**：分析报告矩阵中标注「仅 business」的每个 S/F/B 编号，均须有对应 `@Test` 与断言；类注释附 S/F/B 场景覆盖对照表。

---

## 运行

Maven：`mvn -pl {module} test "-Dtest={ClassName}Test"`  
Gradle：`./gradlew :{module}:test --tests "{ClassName}Test"`
