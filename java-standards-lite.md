---
description: Java 编码规范 — 写任何 Java 代码时必须遵守
alwaysApply: true
enabled: true
inclusion: always
---

# Java 编码规范 (haijun.gao)

写 Java 代码时，以下规则必须严格遵守。每条规则附带正反示例的，示例即契约。

---

## 1. 项目结构

Maven 多模块，包路径格式：`cn.unipus.{project}.{module}.{layer}`

```
xxx-api/          → cn.unipus.xxx.api.{grpc/service | rest/controller | service | config}
xxx-base/         → cn.unipus.xxx.base.{common/constant | common/utils | db/entity | db/repository}
xxx-client/       → HTTP、gRPC 客户端（Proto 定义、DTO、Converter）
xxx-consumer/     → Kafka 消费者
```

### 依赖管理

- 父 POM 用 `<dependencyManagement>` 统一版本
- 子模块**不声明版本号**，全部继承父 POM
- 新增依赖时必须先在父 POM 的 `<dependencyManagement>` 中声明版本

---

## 2. 格式化

- 缩进 4 空格，不用 Tab
- 大括号同行（Java 风格），右括号独占一行
- 方法之间空 1 行，`@Resource` 注入字段之间空 1 行

```java
// ✓ 换行对齐
@Override
public void createUnit(CreateUnitRequestPO request,
        StreamObserver<CreateUnitResponsePO> responseObserver) {
```

## 3. 命名约定

遵循标准 Java 命名规范，以下为项目特有约定：

- 远程调用封装类后缀 `Template`（如 `OrderTemplate`），不用 `Client` 或 `FeignClient`
- 对象转换类后缀 `Convert` / `Converter`，方法名用 `toDTO()` / `toProto()` 表示方向
- 常量用 `interface` 定义，后缀 `Constants`（如 `BookConstants`）
- 工具类后缀 `Utils`，方法全部 `static`
- Service 层**不定义接口**，直接用 `@Service` 类
- 获取数据用 `get` / `load`，校验用 `check` / `valid`
- 变量简洁表意，集合用复数（`units`），Map 体现映射关系（`destIdBySrcId`）
- 常量 `UPPER_SNAKE_CASE`

## 4. Javadoc

### 类级（必须）

```java
/**
 * 教材服务
 *
 * @author haijun.gao
 * @date 2025/7/3
 */
@Service
public class BookService {
```

- `@date` 格式：`yyyy/M/d`

### 方法级（所有 public/private 方法必须）

```java
/**
 * 获取课程教材节点
 *
 * @param bookId 教材ID
 * @param <T>    节点范型
 * @return 教材节点列表
 * @throws ValidationException    校验失败时抛出
 */
public <T> List<T> bookNodes(String bookId) throws ValidationException {
```

- 范型参数 `@param <T>` 放在最后一个 `@param` 之后
- `void` 方法省略 `@return`
- 声明了异常时必须加 `@throws`
- **重写方法（`@Override`）不需要 Javadoc**，除非有补充信息

### 字段级

实体类、DTO、枚举的每个字段必须配注释：

```java
/**
 * 引用的教材ID
 */
private String refId;
```

## 5. 注解顺序

原则：与功能越无关的注解越往前放。

**类级**：Swagger → Lombok → 自定义注解 → Spring 组件 → Spring 配置 → JPA → gRPC

```java
// ✓ 正确顺序
@Tag(name = "教材管理")
@Slf4j
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tab_book")
public class Book {
```

**方法级**：Swagger → 自定义注解 → Spring MVC → 事务 → JPA → `@Override`

## 6. 依赖注入

**统一使用 `@Resource`，禁止使用 `@Autowired`。**

```java
@Resource
private BookRepository bookRepository;

@Resource
private OrderTemplate orderTemplate;
```

## 7. Service 层设计

**不定义接口，直接用 `@Service` 类。** public 方法为入口，private 方法分解逻辑。

**通用、可复用的工具方法禁止写在 Service（或其他业务类）中**，必须提取到对应的 `Utils` 工具类（存放位置遵循项目实际规划）。

## 8. 返回值风格

- **Early Return**：条件分支尽早返回，减少嵌套
- **Optional 链式**：配合 `orElseThrow` / `orElse`

```java
// ✓ Early Return
public CopyCourseDataDTO copyBook(String bookId, String openId) {
    if (BookUtils.isCustomBookId(bookId)) {
        return this.copyCustomBook(bookId, openId);
    }
    return this.copyOfficialBook(bookId, openId);
}

// ✓ Optional 链式
String nodeId = bookRepository.findById(bookId)
        .map(b -> iPublishNodeService.addBookNode(b.getId(), nodeName, b.getCreator()))
        .orElseThrow(() -> new ValidateException(PARAM_ERROR.getCode(), PARAM_ERROR.getMsg()));
```

## 9. 异常处理

- 业务层**不 try-catch**，异常统一交给框架层处理
- 业务层只负责**抛出**语义明确的自定义异常
- 自定义异常继承 `RuntimeException`，携带业务错误码 `code`
- 远程调用统一用 `ExceptionUtils.checkAndThrow()` 校验响应

## 10. 日志规范

- 统一 `@Slf4j`，不手动声明 Logger
- 使用 **SLF4J 占位符 `{}`**，不用字符串拼接
- 业务日志用中文，框架日志用英文
- 远程调用：请求和响应成对记录

```java
log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId);
log.info("xxx request：{}", request);
log.info("xxx response：{}", response);
```

## 11. Lombok 使用

| 注解 | 用途 |
|------|------|
| `@Data` | getter/setter/toString/equals/hashCode |
| `@Slf4j` | Logger |
| `@AllArgsConstructor` | 全参构造 |
| `@NoArgsConstructor` | 无参构造 |
| `@Getter` | 仅需 getter（如枚举） |
| `@Data(staticConstructor = "of")` | 静态工厂方法 |

## 12. 禁止魔法值

逻辑代码中**禁止直接使用字面量**，必须用常量或枚举：

```java
// ✗ 错误
if (Objects.equals(type, "question-block")) { ... }

// ✓ 正确
if (Objects.equals(type, BookConstants.QUERY_TYPE)) { ... }
```

- 有状态语义 → 枚举类
- 简单配置值/标识 → 常量接口（`interface` 定义，可嵌套分组）

## 13. 常量定义

用 `interface` 定义常量组，嵌套内部接口进行二级分类：

```java
public interface CourseConstants {
    String CREATED_TYPE = "customized";
    String QUERY_TYPE = "question-block";

    interface OfficialFlag {
        int YES = 1;
        int NO = 0;
    }
}
```

## 14. 枚举定义

```java
@Getter
@AllArgsConstructor
public enum ContentStatusEnum {

    /**
     * 草稿
     */
    DRAFT((byte) 0),

    /**
     * 已发布
     */
    PUBLISHED((byte) 1);

    /**
     * 状态值
     */
    private final Byte status;
}
```

## 15. Import 规则

- 必须提前 `import`，不在逻辑代码中写类的完全限定路径
- 删除未使用的 import

## 16. 工具库选型

| 场景 | 工具库 |
|------|--------|
| 集合创建 | Guava `Lists.newArrayList()`, `Maps.newLinkedHashMap()` |
| 缓存 | Guava Cache |
| JSON 解析 | Fastjson / Jackson / Hutool |
| 字符串工具 | Apache Commons `StringUtils` |
| HTTP 客户端 | Spring WebClient |
| 对象映射 | MapStruct |
| 代码生成 | Lombok |

## 17. 其他

- JPA 实体字段依赖隐式列映射，不加多余的 `@Column`
- 非 Controller 入参 DTO 不加多余的校验注解
- 工具类方法全部 `static`，不实例化
- 对象转换集中在 `Converter`/`Convert` 类中，不在 Service 层散落
- 配置类用 `@ConfigurationProperties` 绑定，Lombok `@Data` 生成 getter/setter
- 字符串格式化用 `String.format()`

---

## 自检

写完 Java 代码后，必须逐条核对以下高频易错项，不符合的立即修正：

- [ ] 依赖注入用 `@Resource`，不是 `@Autowired`
- [ ] 类有 Javadoc（`@author haijun.gao` + `@date yyyy/M/d`）
- [ ] 所有方法有 Javadoc（`@param` + `@return` + `@throws`）
- [ ] 注解顺序正确（Swagger → Lombok → 自定义 → Spring → JPA → gRPC）
- [ ] 无魔法值（字面量必须提取为常量或枚举）
- [ ] 无多余 `@Column`、无多余 `else`
- [ ] 通用工具方法已提取到 `Utils`，未写在 Service 等业务类中
