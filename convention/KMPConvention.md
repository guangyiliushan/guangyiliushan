# Kotlin Multiplatform Android 项目代码规范

本规范旨在确保 Kotlin Multiplatform Android 项目结构清晰、依赖统一、易于测试和维护，并充分利用 Kotlin 平台特性。

## 通用原则

- **共享逻辑隔离**：仅在 `commonMain` 中编写与平台无关的业务逻辑，避免导入平台特定 API（如 Android SDK）。
- **平台差异处理**：通过 `expect/actual` 机制处理平台差异，`expect` 定义放在 `commonMain`，各平台实现放在相应模块。

## 代码规范

- **命名约定**：
  - 类名：`PascalCase`
  - 函数/变量名：`camelCase`
  - 常量名：`UPPER_SNAKE_CASE`
- **序列化统一**：统一使用 `kotlinx.serialization`，避免使用多个 JSON 序列化库导致逻辑重复和体积膨胀。

## 构建与依赖管理

- **版本统一管理**：使用 Gradle 的版本目录（Version Catalogs）统一依赖版本，防止版本冲突。
- **依赖层级划分**：
  - `commonMain`: 声明通用库（如 Ktor、kotlinx.coroutines、serialization 等）
  - `androidMain`: 声明 Android 特定库（如 Jetpack Lifecycle、DataStore 等）

## 平台特性与模块组织

- **Android 端实现**：
  - 在 `androidMain` 中使用 Jetpack 组件（如 ViewModel、LiveData 等）。
  - 平台逻辑以接口或注入方式桥接到 `commonMain`。
- **协程调度器注入**：
  - 在 Android 侧注入具体调度器（如 `Dispatchers.Main`），避免在 `commonMain` 中硬编码，提高测试灵活性。

## 异步与并发处理

- **使用协程 + Flow**：推荐在 `commonMain` 中使用 `kotlinx.coroutines` 和 `Flow` 管理异步流程。
- **平台调度器隔离**：使用 `expect DispatcherProvider` 抽象调度器，以适配 Android/iOS 平台。
- **Kotlin/Native 注意事项**：在 iOS 等 Native 平台避免复杂的多线程操作，遵守其内存模型和并发约束。

## 测试策略

- **共享逻辑测试**：业务核心测试放在 `commonTest`，统一使用 `kotlin.test` 和 `kotlinx-coroutines-test`。
- **平台测试策略**：
  - Android：UI 与集成测试放在 `androidAndroidTest`，使用 Espresso、Robolectric。
  - iOS：UI 测试放在 `iosTest`，使用 XCTest。
