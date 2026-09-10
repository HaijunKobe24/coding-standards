---
description: Java 编码规范完整版 — 当精简版未覆盖的场景需要查阅详细规则时加载
alwaysApply: false
enabled: true
trigger: model_decision
inclusion: manual
---

# Java Development Standards (haijun.gao)

本文档体现了 haijun.gao 在 Java 后端开发中的编码偏好与规范。

---

## 目录

- [1. 项目结构规范](#1-项目结构规范)
- [2. 代码格式化](#2-代码格式化)
- [3. 命名规范](#3-命名规范)
- [4. Javadoc 规范](#4-javadoc-规范)
- [5. 注解使用规范](#5-注解使用规范)
- [6. 实现风格](#6-实现风格)
- [7. 异常处理](#7-异常处理)
- [8. 日志规范](#8-日志规范)
- [9. 简洁性原则](#9-简洁性原则)
- [10. 复用设计](#10-复用设计)
- [11. 配置管理](#11-配置管理)
- [12. 常用工具库选型](#12-常用工具库选型)

---

## 1. 项目结构规范

### 1.1 多模块划分

采用 Maven 多模块架构，按职责严格分离：

```
project/
├── xxx-api/                              # 主服务（REST + gRPC），承载业务逻辑
├── xxx-base/                             # 数据层（JPA 实体、Repository、常量、工具类）
├── xxx-client/                           # HTTP、gRPC 客户端（Proto 定义、DTO、Converter）
├── xxx-consumer/                         # 消息消费者（Kafka Listener）
```

### 1.2 包结构

按功能域 + 技术层分包，基础包路径格式：`cn.unipus.{project}.{module}.{layer}`

```
cn.unipus.xxx.api
├── grpc
│   ├── interceptor/          # gRPC 拦截器
│   └── service/              # gRPC 服务实现
├── rest
│   └── controller/           # REST 控制器
├── service/                  # 业务服务层
└── config/                   # 配置类

cn.unipus.xxx.base
├── common
│   ├── constant/             # 常量和枚举
│   └── utils/                # 工具类
└── db
    ├── entity/               # JPA 实体
    └── repository/           # 数据访问层
```

### 1.3 依赖管理

- 父 POM 统一使用 `<dependencyManagement>` 管控版本
- 引入 BOM（Spring Boot、gRPC）统一管理传递依赖
- 子模块不声明版本号，全部继承父 POM

---

## 2. 代码格式化

### 2.1 缩进与空格

- **缩进**：4 个空格，不使用 Tab
- **运算符**：前后各一个空格
- **逗号**：后面一个空格
- **大括号**：开放大括号在同一行（Java 风格），右括号独占一行

```java
if (BookUtils.isCustomBookId(bookId)) {
    return copyCustomBook(bookId, openId);
}
return copyOfficialBook(bookId, openId);
```

### 2.2 空行

| 位置 | 空行数 |
|------|--------|
| 类声明与首个字段/方法之间 | 1 |
| `@Resource` 注入字段之间 | 1 |
| 注入字段组与方法之间 | 1 |
| 方法与方法之间 | 1 |
| 方法内逻辑分段 | 0~1（按语义分段） |
| 静态常量与注入字段之间 | 1 |

```java
@Slf4j
@Service
public class BookService {

    @Resource
    private BookRepository bookRepository;

    @Resource
    private OrderTemplate orderTemplate;

    public List<UnitStructDTO> courseBookNodes(String bookId) {
        // ...
    }
}
```

### 2.3 Import 规则

- **不写类的完全限定路径**，必须提前 import——不仅是逻辑代码，Javadoc 标签（`@throws`、`@link`、`@see`）引用类型时同样要用简单类名并配合 import，不能因为"只是注释"而写全限定名
- **删除多余的 import**：未被使用的 import 语句必须及时清理，保持 import 列表精简

---

## 3. 命名规范

### 3.1 类命名（PascalCase）

| 类型 | 后缀约定 | 示例 |
|------|----------|------|
| 业务服务 | `Service` | `BookService` |
| REST 控制器 | `Controller` | `BookController` |
| gRPC 服务 | `GrpcService` | `BookGrpcService` |
| 数据访问 | `Repository` | `BookRepository` |
| 远程调用模板 | `Template` | `OrderTemplate`, `UserTemplate` |
| 拦截器 | `Interceptor` | `GrpcLogInterceptor` |
| 配置属性 | `Properties` | `OrderRemoteProperties` |
| 自动配置 | `AutoConfiguration` | `OrderAutoConfiguration` |
| 数据传输对象 | `DTO` | `BookStructDTO` |
| 请求对象 | `Request` | `AccessTokenRequest` |
| 响应对象 | `Response` | `HttpResponse` |
| 转换器 | `Convert` / `Converter` | `CourseProtoObjectConvert`, `ModelConverter` |
| 工具类 | `Utils` | `BookUtils`, `ExceptionUtils`, `HttpUtils` |
| 枚举 | `Enum` | `CodeEnum`, `ContentStatusEnum` |
| 常量接口 | `Constants` | `BookConstants` |
| JPA 实体 | 无后缀 | `Book`, `BookUnit`, `Course` |
| 消息监听器 | `Listener` | `UnitUpdateListener` |

### 3.2 方法命名（camelCase）

| 语义 | 前缀 | 示例 |
|------|------|------|
| 获取 | `get` / `load` | `getBookStruct()`, `loadAccessToken()` |
| 查询 | `find` / `query` | `findById()`, `queryBookInfo()` |
| 创建 | `create` / `gen` | `createBook()`, `genBookId()` |
| 复制 | `copy` | `copyBook()`, `copyOfficialBook()` |
| 添加 | `add` | `addBookNode()` |
| 更新 | `update` | `updateBookNode()` |
| 删除 | `delete` | `deleteBookNode()` |
| 发布 | `publish` | `publishBook()` |
| 转换 | `to` / `convert` | `toDTO()`, `toProto()` |
| 构建 | `build` | `buildCreateUnitResponse()` |
| 校验 | `check` / `valid` | `checkAndThrow()`, `bookValid()` |
| 布尔判断 | `is` / `has` | `isCustomBookId()`, `isSuccess()` |
| 处理 | `process` | `processOfficalContentList()` |

### 3.3 变量命名（camelCase）

- 普通变量：简洁表意，如 `bookId`, `openId`, `unitName`
- 集合变量：复数形式，如 `units`, `unitIds`
- Map 变量：体现映射关系，如 `destIdBySrcId`
- 常量：`UPPER_SNAKE_CASE`，如 `LOG_PREFIX`, `TOKEN_FMT`, `SOURCE_COURSE`

### 3.4 常量定义

使用接口定义常量组，嵌套内部接口进行二级分类：

```java
public interface CourseConstants {

    String CREATED_TYPE = "customized";
    String QUERY_TYPE = "question-block";
    String BOOK_ID_FMT = "%s_%s";
    String SOURCE_COURSE = "UCOURSE";

    /**
     * 是否官方教材
     *
     * @author haijun.gao
     * @date 2025/7/17
     */
    interface OfficialFlag {
        int YES = 1;
        int NO = 0;
    }
}
```

### 3.5 枚举定义

枚举成员大写，配合 Lombok 和字段 Javadoc：

```java
@Getter
@AllArgsConstructor
public enum ContentStatusEnum {

    /**
     * 草稿
     */
    DRAFT((byte) 0),

    /**
     * 待发布
     */
    WAITING((byte) 1),

    /**
     * 已发布
     */
    PUBLISHED((byte) 2);

    private final Byte status;
}
```

---

## 4. Javadoc 规范

### 4.1 类级 Javadoc

**所有类必须添加 Javadoc**（含内部类、内部接口、内部枚举，见 4.4），格式如下：

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

要素：
- 首行：一句话概述类职责（中文）
- 空行
- `@author`：固定为开发者姓名（`haijun.gao`）
- `@date`：创建日期，格式 **`yyyy/M/d`**（如 `2025/7/3`）

### 4.2 方法级 Javadoc

**所有 public 和 private 方法均配 Javadoc**：

```java
/**
 * 获取课程教材节点
 *
 * @param bookId 教材ID
 * @param <T>    节点范型
 * @return 教材节点列表
 * @throws ValidationException    校验失败时抛出异常
 * @throws IllegalAccessException 访问失败时抛出异常
 */
public <T> List<T> bookNodes(String bookId) throws ValidationException, IllegalAccessException {
```

要素：
- 首行：一句话描述方法功能
- 空行
- `@param`：每个参数一行，简洁描述
- `@param <T>`：范型参数为最后一个参数（若定义了范型）
- `@return`：返回值描述
- 无返回值的 void 方法省略 `@return`
- `@throws`：异常描述（若声明了异常）

**例外：重写方法（含 `@Override` 注解）不需要 Javadoc**，因为继承自父类/接口方法，文档由父类提供。除非需要补充与父类不同的语义或注意事项，否则保持简洁，避免重复。

```java
// 重写方法无需 Javadoc
@Override
public String toString() {
    return "Book[" + id + "]";
}

// 仅在补充信息时才写 Javadoc
/**
 * 重写以追加业务标识到日志上下文
 */
@Override
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
    // ...
}
```

### 4.3 字段级 Javadoc

实体类、DTO、枚举的每个字段都配注释：

```java
/**
 * 引用的教材ID，即官方教材ID
 */
private String refId;
```

**`@Resource` 注入的字段不需要 Javadoc。**

### 4.4 内部类 Javadoc

**内部类、内部接口、内部枚举与顶层类同等要求**，格式完全一致（一句话职责 + `@author` + `@date`），不因嵌套、不因 `private`、不因体积小而省略。常见场景：

| 场景 | 示例 |
|---|---|
| 常量接口的嵌套分类接口 | `CourseConstants.OfficialFlag` |
| 配置类的分组内部类 | `OrderRemoteProperties.Uri` |
| DTO 的嵌套结构类 | `BookStructDTO.Unit` |
| 枚举的嵌套接口/内部枚举 | `ContentStatusEnum.Group` |

```java
@Data
@ConfigurationProperties(prefix = "order.remote")
public class OrderRemoteProperties {

    /**
     * 服务地址
     */
    private String host;

    /**
     * 订单服务接口路径
     *
     * @author haijun.gao
     * @date 2025/7/17
     */
    @Data
    public static class Uri {

        /**
         * 查询订单详情
         */
        private String detail;
    }
}
```

注意区分内部类上的**两类注释**，两者都要写、互不替代：

- 字段位置的内部类**实例**（如 `private Uri uri;`）→ 按 4.3 配字段注释
- 内部类**类型声明**（如 `public static class Uri`）→ 按本节配类级 Javadoc

---

## 5. 注解使用规范

### 5.1 类级注解顺序

原则：与功能越无关的注解越往前放。自上而下：

1. **Swagger 注解**：`@Api`, `@Tag`
2. **Lombok 注解**：`@Slf4j`,`@Data`, `@Getter`, `@AllArgsConstructor`, `@NoArgsConstructor`
3. **自定义注解**：项目自定义的业务注解，如 `@AuditLog`, `@RateLimit`, `@RequirePermission` 等
4. **Spring 组件注解**：`@Service`, `@Component`, `@Configuration`, `@RestController`, `@Order`
5. **Spring 配置注解**：`@ConditionalOnProperty`, `@EnableConfigurationProperties`
6. **JPA 注解**：`@Entity`, `@Table`
7. **gRPC 注解**：`@GRpcService`, `@GRpcGlobalInterceptor`

```java
@Slf4j
@Order(0)
@GRpcGlobalInterceptor
public class GrpcLogInterceptor implements ServerInterceptor {
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "tab_book")
public class Book {
```

方法级注解顺序同理，常见方法级注解归类：Swagger（`@Operation`）、自定义注解（`@AuditLog`）、Spring MVC（`@GetMapping`）、事务（`@Transactional`, `@Modifying`）、JPA（`@Query`）、`@Override`。

### 5.2 依赖注入

**统一使用 `@Resource`**，不使用 `@Autowired`。注入字段之间有空行，`@Resource` 独占一行紧跟字段声明（见 2.3 示例）。

### 5.3 JPA 注解

- 实体类必须 `@Id` 标注主键
- 普通字段依赖隐式列映射，不加多余的 `@Column`

### 5.4 Repository 查询注解

```java
@Transactional
@Modifying
@Query(value = "DELETE FROM tab_book_unit WHERE book_id = :bookId", nativeQuery = true)
void deleteByBookId(@Param("bookId") String bookId);
```

---

## 6. 实现风格

### 6.1 Service 层设计

- **不定义接口，直接用 `@Service` 类**（实用主义，避免无意义的 Interface/Impl 分离）
- public 方法为入口，内部用 private 方法分解逻辑

```java
public CopyCourseDataDTO copyBook(String bookId, String openId) {
    log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId);
    if (BookUtils.isCustomBookId(bookId)) {
        return copyCustomBook(bookId, openId);
    }
    return copyOfficialBook(bookId, openId);
}
```

Service 类**只保留三样东西**：`@Resource` 注入字段、public 业务入口、依赖注入字段的 private 编排方法。除此之外的内容（static 字段、工具方法、配置值、内部类）一律移出，规则见 6.1.1 ~ 6.1.4。

以下规则适用于所有业务类：`Service`、`Controller`、`GrpcService`、`Listener`、`Template`。

#### 6.1.1 业务类中禁止出现 static 字段

业务类中**不允许出现任何 `static` 字段**，`private static final` 也不例外。看到 `static` 就是违规，按下表移走：

| 出现的 static 字段 | 必须移到 |
|---|---|
| 字符串/数字常量、格式模板（`LOG_PREFIX`、`BOOK_ID_FMT`） | `XxxConstants` 接口（见 3.4） |
| 有状态语义的取值（状态码、类型码） | 枚举类（见 3.5） |
| `Pattern`、`DateTimeFormatter` 等线程安全的重用实例 | 对应的 `XxxUtils` 工具类 |
| 静态 `Map` / `List` / `Set` 映射表 | `XxxConstants` 接口，或枚举 + 查表方法 |
| Guava `Cache` / `LoadingCache` | 配置类中 `@Bean` 声明，业务类用 `@Resource` 注入 |
| 可配置值（超时、开关、URL、批量大小） | `@ConfigurationProperties` 配置类（见 11.2） |

```java
// ✗ 错误：Service 里攒静态常量、静态实例、静态映射表
@Service
public class BookService {

    private static final String BOOK_ID_FMT = "%s_%s";
    private static final Pattern CUSTOM_ID = Pattern.compile("^C\\d+$");
    private static final Map<String, String> TYPE_MAP = Maps.newHashMap();
    private static final int BATCH_SIZE = 200;
}

// ✓ 正确：常量进 Constants，正则进 Utils，可配置值进配置类
@Service
public class BookService {

    @Resource
    private BookProperties bookProperties;

    public void copy(String prefix, String suffix) {
        String bookId = String.format(BookConstants.BOOK_ID_FMT, prefix, suffix);
        if (BookUtils.isCustomBookId(bookId)) {
            batchCopy(bookId, bookProperties.getBatchSize());
        }
    }
}
```

注意两个常见的自我辩解，都不成立：

- **"这个常量只有本类用，放 Constants 是过度设计"** —— 归属由内容性质决定，不由使用范围决定。单一使用方的常量同样放 `Constants`，必要时用嵌套接口按类收敛（如 `BookConstants.Copy`）。
- **"`private static final` 已经不是魔法值了"** —— 9.3 禁止魔法值和本条是两条独立规则，满足前者不豁免后者。

#### 6.1.2 业务类中禁止写工具方法

不用"是否通用"这种主观判断，对业务类中**每一个** private 方法逐个执行下面这条机械判据：

> 方法体内是否访问了 `this` 的字段（`@Resource` 注入的依赖、成员变量），或调用了会访问这些字段的实例方法？
> - **否** → 它就是工具方法（等价于"补上 `static` 也能编译通过"），**必须移出业务类**。
> - **是** → 属于业务编排，保留。

移出后的落点：

| 方法性质 | 落点 |
|---|---|
| 无领域语义的通用处理（字符串、集合、时间、编解码） | 通用 `Utils`（如 `DateUtils`、`HttpUtils`） |
| 带领域语义的纯计算、纯判断 | 领域工具类（如 `BookUtils.isCustomBookId()`） |
| 对象结构转换、响应体拼装 | `Converter` / `Convert` 类（见 6.5、10.3） |
| 纯参数校验 | `Utils` 或统一校验层（见 7.2） |

```java
// ✗ 错误：三个方法都不访问注入字段，却留在 Service 里
@Service
public class BookService {

    @Resource
    private BookRepository bookRepository;

    private boolean isCustomBookId(String bookId) {
        return StringUtils.startsWith(bookId, BookConstants.CUSTOM_PREFIX);
    }

    private String genBookId(String prefix, String suffix) {
        return String.format(BookConstants.BOOK_ID_FMT, prefix, suffix);
    }

    private UnitDTO toDTO(BookUnit unit) {
        UnitDTO dto = new UnitDTO();
        dto.setId(unit.getId());
        return dto;
    }
}

// ✓ 正确：判断和拼装归 Utils，转换归 Converter，Service 只留编排
@Service
public class BookService {

    @Resource
    private BookRepository bookRepository;

    public UnitDTO loadUnit(String bookId) {
        if (BookUtils.isCustomBookId(bookId)) {
            return BookConverter.toDTO(bookRepository.findCustom(bookId));
        }
        return BookConverter.toDTO(bookRepository.findOfficial(bookId));
    }
}
```

业务类中**同样不允许出现 `static` 方法**——静态方法天然不依赖实例状态，按上述判据必然属于 `Utils`。

工具类的组织方式见 10.1；存放位置遵循项目实际规划，默认 `base` 模块的 `common.utils` 包。

#### 6.1.3 业务类中禁止散落配置项

**配置项禁止散落在业务类中**（如 `@Value` 直接注入、硬编码可配置值），必须集中到 `@ConfigurationProperties` 配置类统一绑定管理，详见 11.2。

#### 6.1.4 业务类中禁止定义内部类

业务类中**不允许出现任何内部类、内部接口、内部枚举**，`private static class` 也不例外。常见动机与正确落点：

| 内部类的用途 | 必须移到 |
|---|---|
| 多返回值、中间态载体（`CopyContext`、`ParseResult`） | 独立 DTO 类，置于项目的 `dto` 包 |
| 分组常量 | `XxxConstants` 接口的嵌套接口（见 3.4） |
| 状态、类型枚举 | 独立枚举类（见 3.5） |
| 配置分组 | `@ConfigurationProperties` 配置类的内部类（见 11.2） |
| 构建器 | 目标 DTO 上用 Lombok `@Builder` |

```java
// ✗ 错误：Service 里塞私有内部类承载中间态
@Service
public class BookService {

    @Resource
    private BookRepository bookRepository;

    private static class CopyContext {
        private String srcId;
        private String destId;
    }
}

// ✓ 正确：中间态提升为独立 DTO，Service 只留编排
@Service
public class BookService {

    @Resource
    private BookRepository bookRepository;

    public void copy(String bookId) {
        CopyContextDTO context = BookConverter.toCopyContext(bookRepository.findById(bookId));
        this.doCopy(context);
    }
}
```

注意：内部类允许出现在 `Constants` 接口、配置类、DTO、枚举中（见 4.4），本条只约束业务类（`Service` / `Controller` / `GrpcService` / `Listener` / `Template`）。

### 6.2 gRPC 服务实现

- 继承生成的 `*ImplBase` 基类
- 业务委托给 Service 层，gRPC 层只做协议转换
- 使用 Convert 类构建响应

### 6.3 远程调用（Template 模式）

- 使用 WebClient 进行 HTTP 调用
- 请求/响应均记录日志
- 统一调用 `ExceptionUtils.checkAndThrow()` 校验响应

### 6.4 消息消费者

- `@KafkaListener` 配置从 YAML 中引用（SpEL 表达式），非必须
- 异常情况记录日志后 return，不中断消费，非必须

### 6.5 对象转换

- Converter 类使用静态方法
- 方法名以 `toDTO()` / `toProto()` 表示转换方向，方法名称中可包含功能定义
- 开头进行 null 检查
- Protobuf 对象使用 Builder 模式构建

```java
public static CreateUnitRequestPO toProto(CreateUnitRequestDTO dto) {
    if (dto == null) {
        return null;
    }
    return CreateUnitRequestPO.newBuilder()
            .setBookId(dto.getBookId())
            .setName(dto.getName())
            .build();
}
```

### 6.6 返回值风格

- **Early Return**：条件分支尽早返回，减少嵌套（见 6.1 示例）
- **Optional 链式处理**：配合 `orElseThrow` / `orElse` 使用

```java
String nodeId = bookRepository.findById(bookId)
        .map(b -> iPublishNodeService.addBookNode(b.getId(), nodeName, b.getCreator()))
        .orElseThrow(() -> new ValidateException(PARAM_ERROR.getCode(), PARAM_ERROR.getMsg()));
```

---

## 7. 异常处理

- 业务层不进行 `try-catch`，异常统一交给框架层（拦截器/全局异常处理器）处理
- 业务层只负责抛出语义明确的自定义异常

### 7.1 自定义异常

- 继承 `RuntimeException`
- 携带业务错误码 `code`
- 构造器接收 `CodeEnum` 或响应对象

```java
public class HttpException extends RuntimeException {

    @Getter
    private final Integer code;

    public HttpException(CodeEnum codeEnum) {
        super(codeEnum.getMsg());
        this.code = codeEnum.getCode();
    }

    public HttpException(HttpResponse<?> response) {
        super(response.getMessage());
        this.code = response.getCode();
    }
}
```

### 7.2 异常工具类

统一校验远程调用响应，失败则抛出异常（`ExceptionUtils.checkAndThrow()`）：
- null 响应 → 抛出 `REMOTE_SERVER_ERROR`
- 非 success → 抛出携带响应信息的异常
- data 为 null（需校验时）→ 抛出 `REMOTE_SERVER_ERROR`

### 7.3 gRPC 异常拦截器

- 使用异常类型映射表（`Map<Class, Function>`）统一处理，非必须
- 将业务异常转为 gRPC `Status` 返回
- 未知异常返回 `INTERNAL` 状态

### 7.4 业务层异常

使用 `ValidateException` 包装参数校验错误，配合 `CodeEnum` 使用（见 6.6 Optional 链式示例）。

---

## 8. 日志规范

### 8.1 Logger 获取

统一使用 Lombok `@Slf4j` 注解，不手动声明 Logger。

### 8.2 日志级别

| 级别 | 使用场景 | 示例 |
|------|---------|------|
| `info` | 业务关键节点、入参出参 | `log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId)` |
| `debug` | 详细调试信息 | `log.debug("{} [{}] Validation passed", LOG_PREFIX, methodName)` |
| `warn` | 非致命异常 | `log.warn("{} [{}] Failed to serialize request: {}", LOG_PREFIX, methodName, e.getMessage())` |
| `error` | 错误和异常 | `log.error("官方教材章节为空：{}", bookId)` |

### 8.3 日志格式

- 使用 **SLF4J 占位符 `{}`**，不用字符串拼接

### 8.4 远程调用日志

请求和响应成对记录：`log.info("xxx request：{}", request)` / `log.info("xxx response：{}", response)`

---

## 9. 简洁性原则

### 9.1 Lombok 最大化使用

| 注解 | 替代内容 |
|------|---------|
| `@Data` | getter/setter/toString/equals/hashCode |
| `@Slf4j` | Logger 声明 |
| `@AllArgsConstructor` | 全参构造函数 |
| `@NoArgsConstructor` | 无参构造函数 |
| `@Getter` | 仅需 getter 的场景（如枚举） |
| `@Data(staticConstructor = "of")` | 静态工厂方法 |

### 9.2 避免冗余代码

- 不写多余的 `else`（使用 Early Return）
- JPA 实体字段依赖隐式列映射，不加多余的 `@Column`
- 非Controller入参DTO，字段不加多余的校验注解（在 Proto 层或拦截器层统一校验）

### 9.3 禁止魔法值

- 不在逻辑代码中直接使用未经定义的字面量（魔法值）
- 字符串、数字等常量必须在项目约定的常量包中定义为枚举类或常量接口
- 若值具有状态语义，使用枚举类；若为简单配置值或标识，使用常量接口

```java
// ✗ 错误：逻辑中直接写魔法值
if (Objects.equals(type, "question-block")) { ... }

// ✓ 正确：使用常量引用
if (Objects.equals(type, BookConstants.QUERY_TYPE)) { ... }
```

### 9.4 工具方法提取

工具方法禁止留在业务类中，判定标准与落点见 **6.1.2**；业务类中 static 字段的处理见 **6.1.1**。

---

## 10. 复用设计

### 10.1 工具类复用

- 工具类方法全部 `static`
- 工具类不实例化（无构造器或私有构造器）
- 默认抽取到 `base` 模块的 `common.utils` 包中；具体存放位置可按项目实际规划调整

### 10.2 gRPC 拦截器链

通过 `@Order` + `@GRpcGlobalInterceptor` 实现可插拔的拦截器链：

| Order | 拦截器 | 职责 |
|-------|--------|------|
| 0 | `GrpcLogInterceptor` | 日志记录（耗时、入参、出参） |
| 1 | `GrpcValidationInterceptor` | 参数校验（protoc-gen-validate） |
| 2 | `GrpcExceptionInterceptor` | 异常处理（统一转为 gRPC Status） |

### 10.3 Converter 层复用

Proto 和 DTO 的互转逻辑集中在 Converter/Convert 类中，避免在 Service 层散落转换代码。

---

## 11. 配置管理

### 11.1 YAML 风格

- 使用树形结构，逻辑分组清晰
- 通过 `spring.profiles.include` 拆分配置模块（common、mq）
- 使用 `${}` 引用避免重复配置

### 11.2 Properties 类

- 使用 `@ConfigurationProperties(prefix = "xxx")` 绑定
- Lombok `@Data` 自动生成 getter/setter
- URI 等复杂配置使用内部类分组
- **配置项禁止散落在业务类中**（如 `@Value` 直接注入、硬编码可配置值），必须集中到配置类统一绑定管理，见 6.1.3

---

## 12. 常用工具库选型

| 场景 | 工具库 | 用法示例 |
|------|--------|---------|
| 集合创建 | Guava | `Lists.newArrayList()`, `Maps.newLinkedHashMap()` |
| 缓存 | Guava Cache | `CacheBuilder.newBuilder().maximumSize().expireAfterWrite().build()` |
| JSON 解析 | Fastjson/Jackson/Hutool | `JSONUtil.toBean()`, `JSONUtil.toJsonStr()` |
| 字符串工具 | Apache Commons | `StringUtils.startsWith()`, `StringUtils.joinWith()` |
| HTTP 客户端 | Spring WebClient | 响应式非阻塞调用，配合 `block()` 同步使用 |
| 对象映射 | MapStruct | 编译期生成映射代码 |
| 代码生成 | Lombok | 减少样板代码 |
| Proto 序列化 | Protobuf JsonFormat | `JsonFormat.printer().omittingInsignificantWhitespace()` |
| 参数校验 | protoc-gen-validate | Proto 文件中声明校验规则 |
| JWT | Nimbus JOSE JWT | Token 生成和验证 |



---

## 附录：编码风格速查表

```
类（方法、字段）注解顺序    Swagger → Lombok → 自定义注解 → Spring/JPA/gRPC，原则：与功能越无关的注解越往前放
依赖注入                  @Resource（不用 @Autowired）
Javadoc                  所有类（含内部类）和方法必写，@author + @date（yyyy/M/d）
日志                      @Slf4j + 占位符，业务中文/框架英文
异常处理                  自定义 RuntimeException + 错误码枚举
Service 设计              无接口，直接 @Service 类；只留注入字段 + public 入口 + 依赖注入的 private 编排
业务类禁止项              static 字段、static 方法、不访问 this 字段的 private 方法、散落的配置项、内部类
返回值                    Early Return，Optional 链式
集合                      Guava Lists/Maps，Stream API
字符串格式化              String.format()
常量                      interface 定义，嵌套分组
配置                      @ConfigurationProperties 绑定
远程调用                  Template 模式 + WebClient + Guava Cache
```

---

> **维护者**：haijun.gao
> **最后更新**：2026/4/14
