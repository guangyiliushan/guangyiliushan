# Kotlin Multiplatform 工程架构与代码规范

本规范面向“多端一份代码 + 可选服务端”的 Kotlin Multiplatform 工程，目标是让共享业务逻辑、平台实现、UI、测试、构建与 CI 边界保持一致，避免平台细节泄漏到共享层。

## 1. 通用原则

- **R1.1 依赖方向单向**：平台启动器 → UI 层 → 业务层接口 ← 数据实现。UI 只依赖接口与领域模型，不 import 具体实现。
- **R1.2 业务层零平台依赖**：业务层不依赖 Compose、不依赖 Android/iOS 平台 API，所有核心规则都应能在 JVM 上测试。
- **R1.3 分层判定**：换 UI 框架仍需复用的逻辑进业务层；只关注外观、交互、手势的逻辑进 UI/设计系统；描述业务事实的数据进领域模型。
- **R1.4 派生值不落库**：只持久化原始事实；统计、等级、聚合结果等派生数据用纯函数即时计算，避免双真源。

## 2. 源集与平台差异

- **R2.1 源集命名固定**：主源集使用 `commonMain`、`androidMain`、`iosMain`、`jvmMain`、`jsMain`、`wasmJsMain`；测试源集同名加 `Test`。Android 测试源集按 KMP 库插件生成的 host/device 测试源集组织。
- **R2.2 target 只开要用的**：iOS 默认开 `iosArm64` 与 `iosSimulatorArm64`，不再保留 x64 模拟器；Web 优先 `wasmJs`，`js` 仅作为兼容档。
- **R2.3 不轻易自造源集**：优先使用 Kotlin 默认层级模板提供的 `appleMain`、`nativeMain` 等中间源集。只有多平台确实需要共享同一段平台特定代码时，才新增自定义源集。
- **R2.4 expect/actual 三档准则**：先寻找多平台库；简单差异使用 `expect/actual` 函数或属性；平台逻辑复杂时，在 `commonMain` 定义接口，在平台源集实现，并通过依赖注入装配。`expect` 只保留必要的工厂入口，不默认把整个类声明成 `expect class`。
- **R2.5 Android KMP 库模块使用官方插件**：`:core`、`:data` 等非应用 Android target 使用 `com.android.kotlin.multiplatform.library`，并按插件 DSL 配置 host/device 测试。

## 3. 编码规范

- **R3.1 命名统一**：类名 `PascalCase`；函数与变量 `camelCase`；常量 `UPPER_SNAKE_CASE`；项目启用 `kotlin.code.style=official`。
- **R3.2 类型安全值类型**：ID、量纲、计数、外部标识符等优先使用 `value class`，避免裸 `String` 或 `Int` 在层间传递。`commonMain` 中使用 `@JvmInline` 时显式导入 `kotlin.jvm.JvmInline`。
- **R3.3 可见性最小化**：实现类默认 `internal`；模块对外只暴露接口、领域模型和必要的工厂函数。
- **R3.4 单一序列化方案**：统一使用 `kotlinx.serialization`；JSON 配置集中定义，避免多个 JSON 库并存。DTO 不进入 UI，必须先映射为领域模型。
- **R3.5 用户可见文案零硬编码**：用户可见字符串统一走 Compose Resources 或语言包；UI 代码中禁止直接写 `Text("硬编码文案")`，该规则应纳入门禁。

## 4. 构建与依赖管理

- **R4.1 版本目录单一真源**：库版本、插件版本、SDK 版本统一由 `gradle/libs.versions.toml` 管理；模块构建脚本不出现裸版本号。
- **R4.2 依赖分层落位**：多平台 API、Ktor core、coroutines、serialization、datetime、日志门面放 `commonMain`；Ktor 引擎、AndroidX、Darwin 框架、浏览器 API wrapper 放对应平台源集。禁止把平台专属库塞入 `commonMain`。
- **R4.3 `api` 与 `implementation` 有意识使用**：出现在模块公开类型签名中的依赖使用 `api`；内部实现依赖使用 `implementation`，避免实现细节泄漏成事实 API。
- **R4.4 工具链固定**：本机与 CI 使用同一 JDK；`jvmTarget` 按最低支持平台约束，不默认等于构建 JDK；需要自动拉取 JDK 时使用 Gradle toolchain 和 foojay resolver。Gradle 构建缓存与 configuration cache 按项目验证结果启用。
- **R4.5 版本对齐矩阵**：Kotlin、Compose Multiplatform、Android Gradle Plugin、Android SDK、静态分析工具内嵌编译器版本必须作为一组检查。升级时先在分支验证，版本无法对齐时保留低依赖、可稳定运行的门禁方案，不直接关闭门禁。

## 5. `commonMain` 硬约束

- **R5.1 禁止 `java.*`**：时间用 `kotlinx-datetime`，IO 用 Okio，随机用 `kotlin.random`，UUID 与平台能力通过多平台库或 `expect/actual` 工厂处理。
- **R5.2 日志走门面**：不使用 `println` 或 `android.util.Log`；共享层依赖多平台日志门面，具体输出器在平台侧装配。
- **R5.3 调度器注入**：`commonMain` 不硬编码 `Dispatchers.Main`。`CoroutineContext` 或 `CoroutineDispatcher` 由构造注入，生产默认值在平台装配，测试注入 `TestDispatcher`。
- **R5.4 并发安全显式处理**：协程与 `Flow` 是默认异步模型；共享可变状态必须通过不可变数据、原子类型或 `Mutex` 保护。新 Native 内存模型允许对象跨线程共享，但线程安全仍由代码保证。
- **R5.5 Compose 依赖只用多平台坐标**：共享 UI 使用 `org.jetbrains.compose.*` 与 JetBrains/AndroidX 提供的 KMP 产物，不引入 Android 专属 Compose 依赖。
- **R5.6 ViewModel 放共享层**：生命周期感知 ViewModel 可在 `commonMain` 使用；工厂函数必须显式提供 initializer，不依赖 JVM 反射。使用屏幕级 ViewModel 作用域时，按当前 Navigation API 显式挂接 ViewModelStore decorator。
- **R5.7 语言与区域计算下沉纯函数**：编辑距离、分词、取词、难度匹配、格式化规则等逻辑放在业务层纯函数中，UI 只负责展示与组装。

## 6. 平台侧实现规则

| 源集 | 规则 |
|---|---|
| `androidMain` | 应用入口只做 `Activity`、`setContent` 和系统装配；Jetpack 能力优先选择 KMP 发布线；`Context`、权限、Service、Notification 等 Android 专属能力通过接口暴露给共享层。 |
| `iosMain` | 入口由 `UIViewController` 消费；framework 显式声明 `baseName`、静态/动态形态和需要导出的 API；UI 更新回主线程；Swift 侧只依赖稳定导出面。 |
| `jvmMain` | 桌面入口负责窗口、主题和平台调度器装配；桌面端需要启用 `Dispatchers.Main` 时引入对应 coroutines runtime。 |
| `jsMain` / `wasmJsMain` | 浏览器能力通过 kotlin-wrappers 或平台接口隔离；`wasmJs` 为主目标，`js` 只作为兼容档。 |

## 7. 测试策略

- **R7.1 共享逻辑测试在 `commonTest`**：统一使用 `kotlin.test` 与 `kotlinx-coroutines-test`，通过 `runTest` 和注入调度器控制并发行为；不把 JUnit 直接引入共享源集。
- **R7.2 测试命名描述行为**：测试文件使用 `*Test.kt`；测试名表达业务规则，例如 `merge_tombstoneWins`，而不是方法名复读。
- **R7.3 Compose UI 测试优先进共享层**：跨端 UI 行为使用 Compose Multiplatform UI test 与 `runComposeUiTest`，通过语义树断言行为；必要时按当前 API 显式 opt-in。
- **R7.4 Android 测试分两档**：JVM host 测试覆盖不需要真实设备的 Android 行为；仪器化测试使用 KMP 插件提供的 device test 源集与 runner。Espresso 或 Robolectric 只在验证 Android 专属行为时使用，不作为默认测试路线。
- **R7.5 iOS 测试分两层**：共享 Kotlin 测试跑 `iosSimulatorArm64Test`；XCTest/XCUITest 只用于 Swift 侧端到端验收和系统能力验证。
- **R7.6 非 JVM 编译作为门禁**：修改 `commonMain` 后必须至少跑一个非 JVM 编译目标，例如 wasmJs、js 或 iOS，防止 `java.*` 和 JVM 假设只在 Android/JVM 构建中暴露。
- **R7.7 不把截图当主 oracle**：截图测试只作辅助；优先语义树、状态和用户行为断言。flaky 测试先查共现根因，不通过反复 retry 掩盖问题。

## 8. 静态门禁与架构守护

| 档 | 手段 | 防护范围 | 局限 |
|---|---|---|---|
| 物理隔离 | 模块依赖图 | 阻止跨模块错误 import | 同模块内部边界防不住 |
| 官方 lint 规则 | ForbiddenImport、Detekt、Android Lint 等规则 | 同模块内禁用 API、边界与风格 | 配置随工具大版本变化 |
| 自定义门禁 | 项目脚本检查硬编码颜色、魔法尺寸、硬编码文案、组件注册完整性 | 设计系统与项目特有约束 | 需要维护例外清单 |

- **R8.1 门禁进快速 CI job**：本地可执行、带退出码，失败即阻塞合入。
- **R8.2 文档单源生成**：组件目录、设计 token、导航表等由注册表或结构化数据导出，避免多处登记漂移。
- **R8.3 门禁迁移保持判定口径不变**：替换工具时，旧脚本与新规则必须语义同构；只改装配，不改变约束结果。
- **R8.4 配置必须被真实消费**：新增参数、插件、feature flag 或配置文件时，必须有消费点和测试验证，否则视为无效配置。

## 9. CI/CD 纪律

- 快速 JVM 测试先跑；Android、iOS、Desktop、Web 按目标拆分并行 job，总时长取决于最长任务而不是任务之和。
- iOS job 使用 macOS runner，并配置路径门控；纯文档、后端或脚本变更不触发昂贵平台构建。
- 缓存 Gradle 构建缓存与 Konan 缓存；workflow 设置 `timeout-minutes` 和 `concurrency.cancel-in-progress`。
- workflow YAML 在合入前使用静态校验工具检查。
- 不因本机单平台构建通过而跳过 CI；只有四端编译、测试和门禁结果才可作为合入依据。
- 分支模型采用 trunk-based、短命 feature 分支、每 feature 一个 squash commit，里程碑版本显式打 tag。

## 10. 反例清单

1. 在屏幕内复制共享组件副本，导致共享组件被遮蔽或变成死代码。
2. 把业务算法写进 UI 文件，导致无法单测。
3. mock 数据散落在多个屏幕，应收敛到单点数据装配。
4. 用全局单例传递导航参数，应改为类型化参数。
5. 依赖“同一模块内都很熟”来自觉维护边界，缺少 lint 或门禁时边界会失效。
6. 只在 Android/JVM 上验证 `commonMain`，导致非 JVM 目标编译失败。

---

## 官方参考

- Kotlin Multiplatform expected/actual declarations: [https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html)
- Kotlin Multiplatform hierarchical project structure: [https://kotlinlang.org/docs/multiplatform/multiplatform-hierarchy.html](https://kotlinlang.org/docs/multiplatform/multiplatform-hierarchy.html)
- Kotlin Multiplatform project structure basics: [https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html](https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html)
- Kotlin Multiplatform dependencies: [https://kotlinlang.org/docs/multiplatform/multiplatform-add-dependencies.html](https://kotlinlang.org/docs/multiplatform/multiplatform-add-dependencies.html)
- Kotlin Multiplatform ViewModel: [https://kotlinlang.org/docs/multiplatform/compose-viewmodel.html](https://kotlinlang.org/docs/multiplatform/compose-viewmodel.html)
- Compose Multiplatform testing: [https://kotlinlang.org/docs/multiplatform/compose-test.html](https://kotlinlang.org/docs/multiplatform/compose-test.html)
- Kotlin/Native memory management: [https://kotlinlang.org/docs/native-memory-manager.html](https://kotlinlang.org/docs/native-memory-manager.html)
- Kotlin/Native memory manager migration: [https://kotlinlang.org/docs/native-migration-guide.html](https://kotlinlang.org/docs/native-migration-guide.html)
- Kotlin serialization: [https://kotlinlang.org/docs/serialization.html](https://kotlinlang.org/docs/serialization.html)
- Kotlin coroutines guide: [https://kotlinlang.org/docs/coroutines-guide.html](https://kotlinlang.org/docs/coroutines-guide.html)
- Android KMP library plugin: [https://developer.android.com/kotlin/multiplatform/plugin](https://developer.android.com/kotlin/multiplatform/plugin)
- Gradle version catalogs: [https://docs.gradle.org/current/userguide/version_catalogs.html](https://docs.gradle.org/current/userguide/version_catalogs.html)
- Gradle toolchains: [https://docs.gradle.org/current/userguide/toolchains.html](https://docs.gradle.org/current/userguide/toolchains.html)
- Gradle configuration cache: [https://docs.gradle.org/current/userguide/configuration_cache.html](https://docs.gradle.org/current/userguide/configuration_cache.html)
- Gradle build cache: [https://docs.gradle.org/current/userguide/build_cache.html](https://docs.gradle.org/current/userguide/build_cache.html)
