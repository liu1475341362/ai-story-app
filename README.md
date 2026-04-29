2026年4月29日更新：本仓库已经废弃，apk仓库已经迁移到 https://huggingface.co/spaces/liubbninng/ai-story-app/tree/main

# Story - AI故事对话应用

> 一个基于MVVM架构的Android AI聊天应用，支持多终端体验、气泡通知、语音通话和系统级集成。

---

## 📋 项目概览

**Story** 是一款基于AI技术的Android故事对话应用，用户可以与各种虚拟角色（历史人物、虚构角色等）进行沉浸式的聊天互动。应用采用现代化的MVVM架构，实现了从数据层到UI层的完整分层设计。

### 核心能力

- ✅ **动态故事同步**：通过 Retrofit + 协程从服务端拉取最新故事内容
- ✅ **本地持久化**：使用 Room 数据库存储聊天记录，支持离线查看
- ✅ **懒加载聊天会话**：仅在用户进入聊天时才创建会话，优化启动性能
- ✅ **多端聊天体验**：支持主界面、气泡悬浮窗两种模式
- ✅ **AI 回复链路**：实时生成对话，支持文本、图片消息
- ✅ **系统级集成**：Bubble API、通知快捷回复、Deep Link
- ✅ **高性能存储**：MMKV 用于轻量级配置，Protobuf 用于数据传输

---

## 🏗️ 技术架构

### 架构分层

```
┌─────────────────────────────────────────────┐
│                 UI 层                        │
│  ┌──────────────┬──────────────┐           │
│  │ MainActivity │BubbleActivity│           │
│  │  (主界面)     │  (气泡模式)   │           │
│  └──────────────┴──────────────┘           │
├─────────────────────────────────────────────┤
│              ViewModel 层                    │
│     MainViewModel / ChatViewModel          │
├─────────────────────────────────────────────┤
│             Repository 层                    │
│  ┌──────────────────┬──────────────────┐   │
│  │IStoryListRepository│IChatSessionManager│  │
│  │  (故事列表仓库)    │  (聊天会话管理)     │  │
│  └──────────────────┴──────────────────┘   │
├─────────────────────────────────────────────┤
│              DataSource 层                 │
│  ┌──────────────┬──────────────┬─────────┐ │
│  │ StoryDatabase│ RetrofitClient│  MMKV   │ │
│  │  (Room)      │  (网络层)      │(KV存储)  │ │
│  └──────────────┴──────────────┴─────────┘ │
└─────────────────────────────────────────────┘
```

### 架构模式

- **MVVM**：ViewModel 管理 UI 状态，LiveData 实现响应式数据流
- **Repository 模式**：数据访问统一入口，隔离数据源细节
- **懒加载策略**：ChatSessionManagerImpl 按需创建 ChatManager，减少启动负担

---

## 🛠️ 技术栈

### 编程语言
- **Kotlin** (主要) - 现代Android开发首选，协程支持
- **Java** - 部分模块兼容

### 核心依赖

| 类别 | 技术/库 | 说明 |
|------|---------|------|
| **架构组件** | MVVM + Repository | 分层架构设计 |
| **状态管理** | ViewModel + LiveData | 响应式UI更新 |
| **网络请求** | Retrofit 2 + OkHttp | REST API 通信 |
| **数据解析** | Gson + Protobuf | JSON + 高效二进制序列化 |
| **本地数据库** | Room (SQLite) | ORM 数据库，支持协程 |
| **键值存储** | MMKV (Tencent) | 高性能配置存储 |
| **图片加载** | Glide | 图片加载与缓存 |
| **异步编程** | Kotlin Coroutines | 协程异步处理 |
| **系统特性** | Bubble API | Android 11+ 气泡通知 |

---

## 📁 项目结构

```
app/src/main/java/com/xiaoshuohui/story/
│
├── MainActivity.kt                    # 主入口，处理Deep Link
├── BubbleActivity.kt                  # 气泡模式Activity
├── ReplyReceiver.kt                   # 通知快速回复广播接收器
├── StoryApplication.kt                # Application初始化
│
├── data/                              # 数据层
│   ├── IStoryListRepository.kt        # 故事列表仓库接口
│   ├── StoryListRepositoryImpl.kt       # 故事列表仓库实现
│   ├── IChatSessionManager.kt         # 聊天会话管理器接口
│   ├── ChatSessionManagerImpl.kt      # 聊天会话管理器（懒加载实现）
│   ├── StoryRepository.kt             # 远程故事数据源
│   ├── ChatManager.kt                 # 单一会话管理器
│   ├── StoryBean.kt / Message.kt      # 业务数据模型
│   └── repo/                          # 持久化层
│       ├── StoryDatabase.kt           # Room数据库定义
│       ├── ChatRoom.java              # 数据库操作封装
│       ├── dao/                       # DAO接口
│       └── entity/                    # 数据库实体
│
├── ui/                                # UI层
│   ├── chat/                          # 聊天模块
│   │   ├── ChatActivity.kt
│   │   ├── ChatFragment.kt
│   │   ├── ChatViewModel.kt
│   │   └── MessageAdapter.kt
│   ├── home/                          # 首页/联系人列表
│   ├── mine/                          # 个人中心
│   └── settings/                      # 设置页面
│
├── request/                           # 网络层
│   ├── RetrofitClient.kt              # Retrofit配置
│   ├── NetworkConfig.java             # 网络配置
│   └── api/                           # API接口定义
│
├── database/                          # 存储工具
│   └── MMkvUtil.java                  # MMKV封装
│
└── util/                              # 工具类
    └── ImageHelper.kt                 # 图片加载工具
```

---

## 🔄 核心流程

### 1. 应用启动与故事拉取

```
App启动 → MainActivity.onCreate
    → StoryRepository.loadContacts()
        → RetrofitClient.storyApi
            → 服务端API
                → 返回StoryHomeDto
                    → LiveData更新 → UI列表刷新

注: 此时不创建任何ChatManager，优化启动性能
```

### 2. 消息发送与AI回复

```
UI(ChatFragment) → ChatViewModel.send(text)
    → ChatSessionManagerImpl.sendMessage(id, text)
        → 懒加载获取/创建ChatManager
            → ChatManager.addMessage(UserMessage)
                → ChatRoom.persistMessage() 保存到数据库
            → ChatService.sendMessage() 发送到服务端
                ← 接收AI回复
            → ChatManager.addMessage(AiMessage)
                → ChatRoom.persistMessage() 保存到数据库
                    → LiveData更新UI

如果聊天不在前台:
    → NotificationHelper.showNotification()
        → 系统通知 / Bubble气泡
```

---

## 📦 数据模型

### 业务模型

| 类 | 说明 |
|----|------|
| **StoryBean** | 故事角色/联系人模型 |
| **Message** | 聊天消息模型（用户/AI/错误类型） |
| **ChatManager** | 单个聊天会话的消息管理器 |

### 数据库实体

```kotlin
@Entity(tableName = "chat_message")
data class ChatMessageEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val storyId: Long,
    val sender: Int,        // 0: User, 1: AI, 2: Error
    val content: String,
    val timestamp: Long
)
```

---

## 🎯 核心组件详解

### ChatSessionManagerImpl（懒加载实现）

```kotlin
class ChatSessionManagerImpl : IChatSessionManager {
    // 懒加载缓存：只有被访问过的 ChatManager 才会创建
    private val chatsCache = mutableMapOf<Long, ChatManager>()
    
    private fun getOrCreateChatManager(storyId: Long): ChatManager? {
        // 优先从缓存获取
        chatsCache[storyId]?.let { return it }
        // 懒创建：从 StoryRepository 获取 StoryBean
        val storyBean = storyRepository.findContactById(storyId) ?: return null
        return ChatManager(storyBean, chatRoom).also {
            chatsCache[storyId] = it
        }
    }
}
```

**优势**：
- 启动时不创建任何 ChatManager，减少数据库查询
- 只有用户访问过的聊天才占用内存
- 通知快速回复仍可通过 sendMessage() 触发懒加载

---

## ⚙️ 配置与集成

### 必需权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" /> <!-- Android 13+ -->
```

### 环境配置

| 配置项 | 位置 | 说明 |
|--------|------|------|
| **BASE_URL** | `NetworkConfig.java` | 后端API地址 |
| **ServicePath** | `MMkvUtil` | 动态服务路径配置 |
| **UserID** | `MMkvUtil` | 用户标识存储 |

---

## 📱 功能模块

### 1. 首页角色列表
- 展示可对话的AI故事角色
- 支持角色分类浏览
- 角色详情展示

### 2. 智能对话
- 与AI角色实时聊天
- 消息气泡式展示
- 支持图片消息
- 消息本地持久化

### 3. 气泡悬浮窗 (Bubble)
- 基于Android 11+ Bubble API
- 悬浮窗快捷聊天
- 多任务场景下保持对话

### 4. 语音通话
- 语音交互界面
- 模拟真实通话体验

### 5. 通知系统
- 新消息推送通知
- 通知栏快捷回复 (Direct Reply)
- 支持Direct Share分享

### 6. 深度链接 (Deep Link)
- 通过URL直接打开特定角色对话
- 支持外部应用跳转

---

## ⚙️ 兼容性

| 配置项 | 值 | 说明 |
|--------|----|------|
| **minSdk** | 30 | Android 11+ (支持Bubble API) |
| **targetSdk** | 34 | Android 14 |
| **compileSdk** | 36 | 最新编译工具链 |
| **JVM Target** | 17 | Java 17 |

---

## 🏆 技术亮点

1. **现代化MVVM架构**：清晰的层级分离，易于测试和维护
2. **懒加载优化**：按需创建会话，降低启动耗时
3. **响应式编程**：LiveData + Coroutines 实现数据驱动UI
4. **系统级集成**：深度集成Android Bubble、通知、分享等系统能力
5. **高效序列化**：Protobuf + Gson 双协议支持
6. **高性能存储**：MMKV 替代 SharedPreferences，Room 实现本地持久化

---

## 📝 开发信息

- **模块类型**: Application Module (`:app`)
- **架构模式**: MVVM + Repository + LiveData
- **主要语言**: Kotlin (JVM Target 17)
- **最后更新**: 2025-12-24

---

*本项目用于展示Android原生开发能力、MVVM架构实践和系统级功能集成。*
