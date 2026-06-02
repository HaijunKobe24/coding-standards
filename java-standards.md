# Java Development Standards (haijun.gao)

本文档基于 [model-adapter](https://github.com/HaijunKobe24/model-adapter) 项目的实际代码提炼而成，体现了 haijun.gao 在 Java 后端开发中的编码偏好与规范。

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
├── xxx-client/                           # gRPC 客户端（Proto 定义、DTO、Converter）
├── xxx-consumer/                         # 消息消费者（Kafka Listener）
└── xxx-remote-spring-boot-starter/       # 远程服务 SDK（HTTP 模板、自动配置）
```

### 1.2 包结构

按功能域 + 技术层分包，基础包路径格式：`cn.unipus.{project}.{module}.{layer}`

```
cn.unipus.modeladapter.api
├── grpc
│   ├── interceptor/          # gRPC 拦截器
│   └── service/              # gRPC 服务实现
├── rest
│   └── controller/           # REST 控制器
├── service/                  # 业务服务层
└── config/                   # 配置类

cn.unipus.modeladapter.base
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
    return this.copyCustomBook(bookId, openId);
}
return this.copyOfficialBook(bookId, openId);
```

### 2.2 行宽

- 控制在 **100 字符**以内，超出时换行
- 换行对齐至上一行参数起始位置（缩进 8 空格）

```java
@Override
public void createUnit(CreateUnitRequestPO request,
        StreamObserver<CreateUnitResponsePO> responseObserver) {
    // ...
}
```

### 2.3 空行

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
    private BookUnitRepository bookUnitRepository;
  
    @Resource
    private IPublishTemplate iPublishTemplate;

    /**
     * 获取课程教材节点
     */
    public List<UnitStructDTO> courseBookNodes(String bookId) {
        // ...
    }

    /**
     * 获取自定义教材节点
     */
    private List<CustomContentSimpleDTO> getCustomNodes(Book book) {
        // ...
    }
}
```

### 2.4 Import 规则

- **不使用通配符导入**，逐条显式 import
- **不在逻辑代码中写类的完全限定路径**，必须提前 import
- 分组顺序（组间空行分隔）：
  1. `java.*`
  2. `javax.*`
  3. 第三方库（`cn.hutool`, `com.google`, `io.grpc`, `lombok`, `org.springframework` 等）
  4. 项目内部包（`cn.unipus.*`）

---

## 3. 命名规范

### 3.1 类命名（PascalCase）

| 类型 | 后缀约定 | 示例 |
|------|----------|------|
| 业务服务 | `Service` | `IPublishBookService` |
| REST 控制器 | `Controller` | `IPublishController` |
| gRPC 服务 | `GrpcService` | `CourseGrpcService` |
| 数据访问 | `Repository` | `BookRepository` |
| 远程调用模板 | `Template` | `IPublishTemplate`, `CourseTemplate` |
| 拦截器 | `Interceptor` | `GrpcLogInterceptor` |
| 配置属性 | `Properties` | `IPublishRemoteProperties` |
| 自动配置 | `AutoConfiguration` | `IPublishAutoConfiguration` |
| 数据传输对象 | `DTO` | `BookStructDTO`, `CopyCourseDataDTO` |
| 请求对象 | `Request` | `AccessTokenRequest` |
| 响应对象 | `Response` | `HttpResponse` |
| 转换器 | `Convert` / `Converter` | `CourseProtoObjectConvert`, `ModelConverter` |
| 工具类 | `Utils` | `BookUtils`, `ExceptionUtils`, `HttpUtils` |
| 枚举 | `Enum` | `CodeEnum`, `ContentStatusEnum` |
| 常量接口 | `Constants` | `CourseConstants` |
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

**所有类必须添加 Javadoc**，格式如下：

```java
/**
 * 教材服务
 *
 * @author haijun.gao
 * @date 2025/7/3
 */
@Service
public class IPublishBookService {
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
public <T> List<T> courseBookNodes(String bookId) throws ValidationException, IllegalAccessException {
```

要素：
- 首行：一句话描述方法功能
- 空行
- `@param`：每个参数一行，简洁描述
- `@param <T>`：范型参数为最后一个参数（若定义了范型）
- `@return`：返回值描述
- 无返回值的 void 方法省略 `@return`
- `@throws`：异常描述（若声明了异常）

### 4.3 字段级 Javadoc

实体类、DTO、枚举的每个字段都配注释：

```java
/**
 * 引用的教材ID，即官方教材ID
 */
private String refId;
```

### 4.4 常量/内部接口 Javadoc

常量接口中的嵌套接口配完整 Javadoc（含 `@author`、`@date`）。

---

## 5. 注解使用规范

### 5.1 类级注解顺序

自上而下：

1. **Lombok 注解**：`@Data`, `@Slf4j`, `@Getter`, `@AllArgsConstructor`, `@NoArgsConstructor`
2. **排序/作用域注解**：`@Order`
3. **Spring 组件注解**：`@Service`, `@Component`, `@Configuration`, `@RestController`
4. **Spring 配置注解**：`@ConditionalOnProperty`, `@EnableConfigurationProperties`
5. **JPA 注解**：`@Entity`, `@Table`
6. **gRPC 注解**：`@GRpcService`, `@GRpcGlobalInterceptor`

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

### 5.2 依赖注入

**统一使用 `@Resource`**，不使用 `@Autowired`：

```java
@Resource
private BookRepository bookRepository;

@Resource
private IPublishTemplate iPublishTemplate;
```

- 注入字段之间有空行
- `@Resource` 独占一行，紧跟字段声明

### 5.3 JPA 注解

```java
@Entity
@Table(name = "tab_book_unit")
public class BookUnit {

    @Id
    private String id;

    // 普通字段无需 @Column（依赖隐式映射）
    private String bookId;
}
```

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
- 调用自身私有方法使用 `this.` 前缀（非必须）

```java
public CopyCourseDataDTO copyBook(String bookId, String openId) {
    log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId);
    if (BookUtils.isCustomBookId(bookId)) {
        return this.copyCustomBook(bookId, openId);
    }
    return this.copyOfficialBook(bookId, openId);
}

private CopyCourseDataDTO copyOfficialBook(String bookId, String openId) {
    // 具体实现
}

private CopyCourseDataDTO copyCustomBook(String bookId, String openId) {
    // 具体实现
}
```

### 6.2 gRPC 服务实现

- 继承生成的 `*ImplBase` 基类
- 业务委托给 Service 层，gRPC 层只做协议转换
- 使用 Convert 类构建响应

```java
@Slf4j
@GRpcService
public class CourseGrpcService extends CourseServiceGrpc.CourseServiceImplBase {

    @Resource
    private IPublishBookService iPublishBookService;

    @Override
    public void createUnit(CreateUnitRequestPO request,
            StreamObserver<CreateUnitResponsePO> responseObserver) {
        String nodeId = iPublishBookService.addBookNode(request.getBookId(), request.getName());
        CreateUnitResponsePO response = CourseProtoObjectConvert.buildCreateUnitResponse(
                request.getBookId(), nodeId);
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

### 6.3 了解 gRPC 接口

当需要了解依赖包中定义的 gRPC 接口时，使用该依赖服务的**服务端反射（Server Reflection）**接口进行查询，而非阅读源码或 Proto 文件。

### 6.4 远程调用（Template 模式）

- 使用 WebClient 进行 HTTP 调用
- 请求/响应均记录日志
- 统一调用 `ExceptionUtils.checkAndThrow()` 校验响应
- Token 等信息通过 Guava Cache 缓存

```java
public BookStructDTO getBookStruct(String bookId, AccessTokenRequest tokenRequest) {
    log.info("getBookStruct request bookId: {}", bookId);
    IPublishResponse<BookStructDTO> response = webClient.get()
            .uri(properties.getUri().getGetBookStructUri(bookId))
            .header(properties.getTokenHeader(), this.loadAccessToken(tokenRequest))
            .retrieve()
            .bodyToMono(new ParameterizedTypeReference<IPublishResponse<BookStructDTO>>() {
            }).block();
    log.info("getBookStruct response: {}", response);
    ExceptionUtils.checkAndThrow(response, true);
    return Objects.requireNonNull(response).getData();
}
```

### 6.5 了解 HTTP 接口

当需要了解依赖服务的 HTTP 接口时，使用其 **Swagger 文档**（通过 `curl` 获取）。服务地址从配置文件中读取，Swagger 版本可能为 v2 或 v3。

### 6.6 消息消费者

- `@KafkaListener` 配置从 YAML 中引用（SpEL 表达式），非必须
- 消息体使用 Hutool `JSONUtil.toBean()` 反序列化，非必须
- 异常情况记录日志后 return，不中断消费，非必须

```java
@KafkaListener(topics = "#{unitUpdate.topic}", groupId = "#{unitUpdate.group}",
        concurrency = "#{unitUpdate.concurrency}",
        properties = "max.poll.records=#{unitUpdate.maxRecords}")
public void consume(String message) {
    log.info("msg: {}", message);
    IPublishContentMsg msg = JSONUtil.toBean(message, IPublishContentMsg.class);
    if (Objects.equals(msg.getStatus(), ContentStatusEnum.PUBLISHED.getStatus().intValue())) {
        log.info("msg is publish unit: {}", msg.getBizId());
        return;
    }
    // 继续处理...
}
```

### 6.7 对象转换

- Converter 类使用静态方法
- 方法名以 `toDTO()` / `toProto()` 表示转换方向，方法名称中可包含功能定义
- 开头进行 null 检查
- Protobuf 对象使用 Builder 模式构建

```java
public class ModelConverter {

    public static StatusDTO toDTO(Status proto) {
        if (proto == null) {
            return null;
        }
        StatusDTO dto = new StatusDTO();
        dto.setCode(proto.getCode());
        dto.setMsg(proto.getMsg());
        return dto;
    }

    public static CreateUnitRequestPO toProto(CreateUnitRequestDTO dto) {
        if (dto == null) {
            return null;
        }
        return CreateUnitRequestPO.newBuilder()
                .setBookId(dto.getBookId())
                .setName(dto.getName())
                .build();
    }
}
```

### 6.8 返回值风格

- **Early Return**：条件分支尽早返回，减少嵌套
- **Optional 链式处理**：配合 `orElseThrow` / `orElse` 使用

```java
// Early Return
if (BookUtils.isCustomBookId(bookId)) {
    return this.copyCustomBook(bookId, openId);
}
return this.copyOfficialBook(bookId, openId);

// Optional 链式
String nodeId = bookRepository.findById(bookId)
        .map(b -> iPublishNodeService.addBookNode(b.getId(), nodeName, b.getCreator()))
        .orElseThrow(() -> new ValidateException(PARAM_ERROR.getCode(), PARAM_ERROR.getMsg()));
```

### 6.9 Stream 与函数式

- 链式调用，按照代码格式化规则
- 终止操作（`collect`, `forEach`），按照代码格式化规则

```java
List<String> unitIds = units.stream().map(BookUnit::getId).collect(Collectors.toList());

Map<Boolean, List<BookUnit>> unitsByPub = units.stream()
.collect(Collectors.groupingBy(
                unit -> Objects.equals(unit.getStatus(), PUBLISHED.getStatus())));
```

---

## 7. 异常处理

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

统一校验远程调用响应，失败则抛出异常：

```java
public static void checkAndThrow(HttpResponse<?> response, boolean validData) {
    if (response == null) {
        log.error("HttpResponse is null");
        throw new HttpException(CodeEnum.REMOTE_SERVER_ERROR);
    }
    if (!response.isSuccess()) {
        log.error("HttpResponse is not success, response: {}", response);
        throw new HttpException(response);
    }
    if (validData && response.getData() == null) {
        log.error("HttpResponse data is null, response: {}", response);
        throw new HttpException(CodeEnum.REMOTE_SERVER_ERROR);
    }
}
```

### 7.3 gRPC 异常拦截器

- 使用异常类型映射表（`Map<Class, Function>`）统一处理，非必须
- 将业务异常转为 gRPC `Status` 返回
- 未知异常返回 `INTERNAL` 状态

```java
private static final Map<Class<? extends Exception>, Function<Exception, Pair<Integer, String>>>
        EXP_MAP = Maps.newLinkedHashMap();

static {
    EXP_MAP.put(MethodArgumentNotValidException.class, PARAM_ERR_GETTER);
    EXP_MAP.put(BindException.class, PARAM_ERR_GETTER);
    EXP_MAP.put(ConstraintViolationException.class, PARAM_ERR_GETTER);
    EXP_MAP.put(StatefulException.class, e -> {
        StatefulException exp = (StatefulException) e;
        return Pair.of(exp.getStatus(), exp.getMessage());
    });
}
```

### 7.4 业务层异常

使用 `ValidateException` 包装参数校验错误，配合 `CodeEnum` 使用：

```java
bookRepository.findById(id).orElseThrow(
        () -> new ValidateException(PARAM_ERROR.getCode(), PARAM_ERROR.getMsg()));
```

---

## 8. 日志规范

### 8.1 Logger 获取

统一使用 Lombok `@Slf4j` 注解，不手动声明 Logger：

```java
@Slf4j
@Service
public class IPublishBookService {
    // 直接使用 log.info(...)
}
```

### 8.2 日志级别

| 级别 | 使用场景 | 示例 |
|------|---------|------|
| `info` | 业务关键节点、入参出参 | `log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId)` |
| `debug` | 详细调试信息 | `log.debug("{} [{}] Validation passed", LOG_PREFIX, methodName)` |
| `warn` | 非致命异常 | `log.warn("{} [{}] Failed to serialize request: {}", LOG_PREFIX, methodName, e.getMessage())` |
| `error` | 错误和异常 | `log.error("官方教材章节为空：{}", bookId)` |

### 8.3 日志格式

- 使用 **SLF4J 占位符 `{}`**，不用字符串拼接
- 业务日志使用中文描述
- 拦截器/框架日志使用英文 + `LOG_PREFIX` 前缀

```java
// 业务日志（中文）
log.info("开始复制教材，openId: {}, bookId: {}", openId, bookId);
log.info("新增教材节点：bookId：{}，nodeId：{}", bookId, nodeId);
log.error("官方教材不存在：{}", bookId);

// 框架日志（英文 + 前缀）
private static final String LOG_PREFIX = "[gRPC]";
log.info("{} [{}] Request started", LOG_PREFIX, methodName);
log.info("{} [{}] Request completed in {}ms", LOG_PREFIX, methodName, duration);
log.error("{} [{}] Request failed: {}", LOG_PREFIX, methodName, e.getMessage(), e);
```

### 8.4 远程调用日志

请求和响应成对记录：

```java
log.info("accessToken request：{}", request);
log.info("accessToken response：{}", response);
```

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

### 9.2 Service 无接口

- 不创建无意义的 `Interface` + `Impl` 分离
- 直接用 `@Service` 类承载业务逻辑
- 只在确实需要多实现时才抽取接口

### 9.3 字段注入优于构造器注入

- 使用 `@Resource` 字段注入，代码更紧凑
- 不编写构造器注入的大段参数列表

### 9.4 避免冗余代码

- 不写多余的 `else`（使用 Early Return）
- JPA 实体字段依赖隐式列映射，不加多余的 `@Column`
- 非Controller入参DTO，字段不加多余的校验注解（在 Proto 层或拦截器层统一校验）

---

## 10. 复用设计

### 10.1 Spring Boot Starter 封装（非必须）

将远程服务调用封装为独立 Starter，实现跨项目复用：

```
xxx-remote-spring-boot-starter/
├── http/
│   ├── ipublish/
│   │   ├── IPublishTemplate.java            # 调用模板
│   │   ├── config/
│   │   │   ├── IPublishAutoConfiguration.java
│   │   │   └── IPublishRemoteProperties.java
│   │   └── model/                           # 请求/响应模型
│   └── course/
│       ├── CourseTemplate.java
│       └── config/
└── common/
    ├── exception/                           # 统一异常
    ├── model/                               # 通用响应模型
    └── utils/                               # 工具类
```

- 通过 `@ConditionalOnProperty` 控制按需加载
- 通过 `@ConfigurationProperties` 绑定配置
- 通过 `spring.factories` 注册自动配置

### 10.2 工具类复用

- 工具类方法全部 `static`
- 工具类不实例化（无构造器或私有构造器）
- 抽取到 `base` 模块的 `common.utils` 包中

```java
public class BookUtils {

    public static String genBookId() {
        return String.format(CourseConstants.BOOK_ID_FMT,
                CourseConstants.SOURCE_COURSE, UUID.randomUUID());
    }

    public static boolean isCustomBookId(String id) {
        return StringUtils.startsWith(id, CourseConstants.SOURCE_COURSE);
    }
}
```

### 10.3 常量集中管理

- 业务常量集中在 `common.constant` 包
- 使用接口定义常量，嵌套接口分类
- 错误码使用枚举，带 `code` + `msg`

### 10.4 gRPC 拦截器链

通过 `@Order` + `@GRpcGlobalInterceptor` 实现可插拔的拦截器链：

| Order | 拦截器 | 职责 |
|-------|--------|------|
| 0 | `GrpcLogInterceptor` | 日志记录（耗时、入参、出参） |
| 1 | `GrpcValidationInterceptor` | 参数校验（protoc-gen-validate） |
| 2 | `GrpcExceptionInterceptor` | 异常处理（统一转为 gRPC Status） |

### 10.5 Converter 层复用

Proto 和 DTO 的互转逻辑集中在 Converter/Convert 类中，避免在 Service 层散落转换代码。

---

## 11. 配置管理

### 11.1 YAML 风格

- 使用树形结构，逻辑分组清晰
- 通过 `spring.profiles.include` 拆分配置模块（common、mq）
- 使用 `${}` 引用避免重复配置

```yaml
server:
  port: 8081
  servlet:
    context-path: /api/adapter
    application-display-name: model-adapter

spring:
  application:
    name: ${server.servlet.application-display-name}
  profiles:
    include: common,mq
```

### 11.2 Properties 类

- 使用 `@ConfigurationProperties(prefix = "xxx")` 绑定
- Lombok `@Data` 自动生成 getter/setter
- URI 等复杂配置使用内部类分组

```java
@Data
@ConfigurationProperties(prefix = "http.ipublish")
public class IPublishRemoteProperties {

    private boolean enabled;
    private String baseUrl;
    private int timeout;
    private String tokenHeader;
    private Uri uri;

    @Data
    public static class Uri {
        private String accessTokenUri;
        private String getBookStructUri;

        public String getGetBookStructUri(String bookId) {
            return String.format(getBookStructUri, bookId);
        }
    }
}
```

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
类（方法、字段）注解顺序        Swagger → Lombok → JPA/gRPC/Spring/SpringMVC，原则：与功能越无关的注解越往前放
依赖注入                     @Resource（不用 @Autowired）
Javadoc                     所有方法必写，@author + @date（yyyy/M/d）
日志                         @Slf4j + 占位符，业务中文/框架英文
异常处理          					 自定义 RuntimeException + 错误码枚举
Service 设计      				   无接口，直接 @Service 类
返回值            					  Early Return，Optional 链式
集合              				  Guava Lists/Maps，Stream API
字符串格式化      					  String.format()
常量              				  interface 定义，嵌套分组
配置              				  @ConfigurationProperties 绑定
远程调用          				   Template 模式 + WebClient + Guava Cache
```

---

> **维护者**：haijun.gao
> **最后更新**：2026/4/14
> **基准项目**：[model-adapter](https://github.com/HaijunKobe24/model-adapter)
