

---

# 📚 基于Spring Boot的论坛系统 - 课程设计项目文档

## 一、项目概述

### 1.1 项目简介

这是一个基于 **Spring Boot + MyBatis + MySQL** 的前后端分离论坛系统，实现了用户管理、版块管理、帖子管理、回复功能、站内信等核心功能。

### 1.2 技术栈

| 层次         | 技术选型                |
| ------------ | ----------------------- |
| **后端框架** | Spring Boot 2.7.6       |
| **持久层**   | MyBatis 2.3.0           |
| **数据库**   | MySQL 5.x               |
| **数据源**   | Druid 1.2.16            |
| **JDK版本**  | JDK 1.8                 |
| **前端**     | HTML + CSS + JavaScript |
| **构建工具** | Maven                   |

---

## 二、系统架构设计

### 2.1 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        前端展示层 (View)                          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │
│  │ index   │ │ sign-in │ │ article │ │ profile │ │ message │    │
│  │ .html   │ │ .html   │ │ .html   │ │ .html   │ │  相关   │    │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │ HTTP/AJAX
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      控制层 (Controller)                          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │UserController│ │ArticleController│ │BoardController│           │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
│  ┌──────────────────┐ ┌──────────────────┐                       │
│  │ArticleReplyController│ │MessageController │                    │
│  └──────────────────┘ └──────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       业务逻辑层 (Service)                        │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐                    │
│  │IUserService│ │IArticleService│ │IBoardService│                 │
│  └────────────┘ └────────────┘ └────────────┘                    │
│  ┌──────────────────┐ ┌──────────────────┐                       │
│  │IArticleReplyService│ │IMessageService │                        │
│  └──────────────────┘ └──────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      数据访问层 (DAO/Mapper)                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │UserMapper│ │ArticleMapper│ │BoardMapper│ │MessageMapper│       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘            │
│                    ┌──────────────────┐                          │
│                    │ArticleReplyMapper│                          │
│                    └──────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        数据库层 (MySQL)                           │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────────┐ │
│  │ t_user │ │t_board │ │t_article│ │t_message│ │t_article_reply│ │
│  └────────┘ └────────┘ └────────┘ └────────┘ └────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 项目目录结构

```
src/main/java/com/bitejiuyeke/forum/
├── ForumApplication.java          # 启动类
├── common/                        # 公共组件
│   ├── AppResult.java             # 统一返回结果封装
│   └── ResultCode.java            # 结果状态码枚举
├── config/                        # 配置类
│   ├── AppConfig.java             # 应用配置
│   ├── MybatisConfig.java         # MyBatis配置
│   └── SwaggerConfig.java         # Swagger配置
├── controller/                    # 控制器层
│   ├── UserController.java        # 用户接口
│   ├── ArticleController.java     # 帖子接口
│   ├── ArticleReplyController.java# 回复接口
│   ├── BoardController.java       # 版块接口
│   └── MessageController.java     # 站内信接口
├── dao/                           # 数据访问层
│   ├── UserMapper.java
│   ├── ArticleMapper.java
│   ├── ArticleReplyMapper.java
│   ├── BoardMapper.java
│   └── MessageMapper.java
├── exception/                     # 异常处理
├── interceptor/                   # 拦截器
│   ├── LoginInterceptor.java      # 登录拦截器
│   └── AppInterceptorConfigurer.java
├── model/                         # 实体类
│   ├── User.java
│   ├── Article.java
│   ├── ArticleReply.java
│   ├── Board.java
│   └── Message.java
├── services/                      # 业务层接口
│   ├── IUserService.java
│   ├── IArticleService.java
│   ├── IArticleReplyService.java
│   ├── IBoardService.java
│   ├── IMessageService.java
│   └── impl/                      # 业务层实现
└── utils/                         # 工具类
    ├── MD5Util.java               # MD5加密
    ├── StringUtil.java            # 字符串工具
    └── UUIDUtil.java              # UUID生成
```

---

## 三、数据库设计

### 3.1 ER图关系

```
┌─────────────┐         ┌─────────────┐
│   t_user    │────1:N──│  t_article  │
│  (用户表)    │         │  (帖子表)    │
└─────────────┘         └─────────────┘
       │                       │
       │                       │
      1:N                     1:N
       │                       │
       ▼                       ▼
┌─────────────┐         ┌─────────────────┐
│  t_message  │         │ t_article_reply │
│ (站内信表)   │         │   (回复表)       │
└─────────────┘         └─────────────────┘

┌─────────────┐         ┌─────────────┐
│   t_board   │────1:N──│  t_article  │
│  (版块表)    │         │  (帖子表)    │
└─────────────┘         └─────────────┘
```

### 3.2 数据表说明

| 表名                | 说明     | 主要字段                                                     |
| ------------------- | -------- | ------------------------------------------------------------ |
| **t_user**          | 用户表   | id, username, password, nickname, salt, avatarUrl, articleCount, isAdmin, state |
| **t_board**         | 版块表   | id, name, articleCount, sort, state                          |
| **t_article**       | 帖子表   | id, boardId, userId, title, content, visitCount, replyCount, likeCount, state |
| **t_article_reply** | 回复表   | id, articleId, postUserId, replyId, replyUserId, content, likeCount |
| **t_message**       | 站内信表 | id, postUserId, receiveUserId, content, state                |

---

## 四、功能模块划分

### 4.1 功能模块图

```
                    ┌──────────────────────────┐
                    │      论坛系统功能模块       │
                    └──────────────────────────┘
                               │
       ┌───────────┬───────────┼───────────┬───────────┐
       │           │           │           │           │
       ▼           ▼           ▼           ▼           ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ 用户模块  │ │ 版块模块  │ │ 帖子模块  │ │ 回复模块  │ │站内信模块 │
└──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
     │            │            │            │            │
     ▼            ▼            ▼            ▼            ▼
• 用户注册     • 版块列表    • 发布帖子    • 回复帖子    • 发送站内信
• 用户登录     • 版块详情    • 帖子列表    • 回复列表    • 站内信列表
• 退出登录                  • 帖子详情                 • 未读数统计
• 获取用户信息              • 修改帖子                 • 标记已读
• 修改密码                  • 删除帖子                 • 回复站内信
• 个人中心                  • 点赞功能
```

---

## 五、六人团队任务分配

### 📋 任务分配总览

| 成员       | 负责模块       | 主要职责                           | 相关文件                                          |
| ---------- | -------------- | ---------------------------------- | ------------------------------------------------- |
| **余涛**   | 用户模块       | 用户注册、登录、退出、用户信息管理 | UserController, UserMapper, IUserService          |
| **祁政儒** | 帖子模块       | 帖子CRUD、点赞、帖子列表           | ArticleController, ArticleMapper, IArticleService |
| **甘雲龙** | 版块与回复模块 | 版块管理、回复功能                 | BoardController, ArticleReplyController           |
| **张家豪** | 站内信模块     | 站内信发送、接收、未读统计         | MessageController, MessageMapper, IMessageService |
| **朱烁**   | 前端开发       | 页面设计与实现、前后端交互         | 所有HTML/CSS/JS文件                               |
| **高若晗** | 公共组件与部署 | 公共代码、配置、测试、部署         | common/, config/, interceptor/, utils/            |

---

### 👤 组长余涛：用户模块负责人

**负责范围：**

- 用户注册功能
- 用户登录功能
- 退出登录功能
- 获取用户信息
- 修改密码
- 个人中心

**涉及文件：**

```
controller/UserController.java
services/IUserService.java
services/impl/UserServiceImpl.java
dao/UserMapper.java
model/User.java
mapper/UserMapper.xml
```

**核心接口：**

| API               | 方法 | 描述         |
| ----------------- | ---- | ------------ |
| `/user/register`  | POST | 用户注册     |
| `/user/login`     | POST | 用户登录     |
| `/user/logout`    | GET  | 退出登录     |
| `/user/info`      | GET  | 获取用户信息 |
| `/user/modifyPwd` | POST | 修改密码     |

---

### 📝 祁政儒：帖子模块负责人

**负责范围：**

- 发布帖子
- 获取帖子列表
- 获取帖子详情
- 修改帖子
- 删除帖子
- 点赞功能

**涉及文件：**

```
controller/ArticleController.java
services/IArticleService.java
services/impl/ArticleServiceImpl.java
dao/ArticleMapper.java
model/Article.java
mapper/ArticleMapper.xml
```

**核心接口：**

| API                        | 方法 | 描述         |
| -------------------------- | ---- | ------------ |
| `/article/create`          | POST | 发布新帖     |
| `/article/getAllByBoardId` | GET  | 获取帖子列表 |
| `/article/details`         | GET  | 获取帖子详情 |
| `/article/modify`          | POST | 修改帖子     |
| `/article/delete`          | POST | 删除帖子     |
| `/article/thumbsUp`        | POST | 点赞         |

---

### 🗂️ **甘雲龙**：版块与回复模块负责人

**负责范围：**

- 获取首页版块列表
- 获取版块信息
- 回复帖子
- 获取回复列表

**涉及文件：**

```
controller/BoardController.java
controller/ArticleReplyController.java
services/IBoardService.java
services/IArticleReplyService.java
services/impl/BoardServiceImpl.java
services/impl/ArticleReplyServiceImpl.java
dao/BoardMapper.java
dao/ArticleReplyMapper.java
model/Board.java
model/ArticleReply.java
mapper/BoardMapper.xml
mapper/ArticleReplyMapper.xml
```

**核心接口：**

| API                 | 方法 | 描述             |
| ------------------- | ---- | ---------------- |
| `/board/topList`    | GET  | 获取首页版块列表 |
| `/board/getById`    | GET  | 获取版块信息     |
| `/reply/create`     | POST | 回复帖子         |
| `/reply/getReplies` | GET  | 获取回复列表     |

---

### 💬 张家豪：站内信模块负责人

**负责范围：**

- 发送站内信
- 获取站内信列表
- 获取未读数
- 标记已读
- 回复站内信

**涉及文件：**

```
controller/MessageController.java
services/IMessageService.java
services/impl/MessageServiceImpl.java
dao/MessageMapper.java
model/Message.java
mapper/MessageMapper.xml
```

**核心接口：**

| API                       | 方法 | 描述           |
| ------------------------- | ---- | -------------- |
| `/message/send`           | POST | 发送站内信     |
| `/message/getAll`         | GET  | 获取所有站内信 |
| `/message/getUnreadCount` | GET  | 获取未读数     |
| `/message/markRead`       | POST | 标记已读       |
| `/message/reply`          | POST | 回复站内信     |

---

### 🎨 朱烁：前端开发负责人

**负责范围：**

- 首页界面 (index.html)
- 登录/注册页面 (sign-in.html, sign-up.html)
- 帖子相关页面 (article.html, article_list.html, article_edit.html)
- 用户中心页面 (profile.html, settings.html)
- 详情页面 (details.html)
- 公共JS组件 (common.js)
- 前后端数据交互

**涉及文件：**

```
static/
├── index.html           # 首页
├── sign-in.html         # 登录页
├── sign-up.html         # 注册页
├── article.html         # 帖子详情
├── article_list.html    # 帖子列表
├── article_edit.html    # 帖子编辑
├── profile.html         # 用户资料
├── settings.html        # 设置页
├── details.html         # 详情页
├── js/common.js         # 公共JS
└── image/               # 图片资源
```

**技术要点：**

- 使用AJAX进行前后端交互
- 响应式布局设计
- 用户体验优化

---

### ⚙️ 高若晗：公共组件与部署负责人

**负责范围：**

- 统一返回结果封装 (AppResult)
- 状态码定义 (ResultCode)
- 配置类管理
- 登录拦截器
- 工具类开发
- 数据库设计与初始化
- 单元测试
- 项目部署与文档

**涉及文件：**

```
common/
├── AppResult.java       # 统一返回结果
└── ResultCode.java      # 状态码枚举

config/
├── AppConfig.java       # 应用配置
├── MybatisConfig.java   # MyBatis配置
└── SwaggerConfig.java   # Swagger配置

interceptor/
├── LoginInterceptor.java            # 登录拦截器
└── AppInterceptorConfigurer.java    # 拦截器配置

utils/
├── MD5Util.java         # MD5加密工具
├── StringUtil.java      # 字符串工具
└── UUIDUtil.java        # UUID工具

exception/               # 异常处理

src/
├── forum_db.sql         # 数据库脚本
└── repair.sql           # 修复脚本

test/                    # 测试代码
```

**技术要点：**

- 全局异常处理
- 登录状态拦截
- 密码加密存储
- 项目打包部署
- 编写测试用例

---

## 六、开发流程

### 6.1 开发阶段划分

```
第一阶段（第1-2周）：环境搭建与基础架构
├── 成员F：搭建项目骨架、配置数据库、公共组件
├── 成员A：用户表设计、实体类、Mapper接口
└── 全员：熟悉项目结构和技术栈

第二阶段（第3-4周）：核心功能开发
├── 成员A：用户注册、登录功能
├── 成员B：帖子发布、列表功能
├── 成员C：版块功能、回复功能
├── 成员D：站内信基础功能
├── 成员E：基础页面框架搭建
└── 成员F：登录拦截器、工具类

第三阶段（第5-6周）：功能完善与前端对接
├── 成员A：个人中心、修改密码
├── 成员B：帖子编辑、删除、点赞
├── 成员C：回复列表显示
├── 成员D：未读统计、已读标记
├── 成员E：所有页面开发、AJAX交互
└── 成员F：Swagger文档、单元测试

第四阶段（第7周）：测试与部署
├── 全员：功能测试、Bug修复
├── 成员F：项目打包、部署上线
└── 全员：撰写课程设计报告
```

## 七、项目亮点与扩展建议

### 7.1 现有亮点

1. **前后端分离架构** - RESTful API设计
2. **统一返回结果封装** - 规范的接口响应
3. **登录拦截器** - 安全访问控制
4. **密码加盐加密** - MD5 + Salt安全存储

### 7.2 可扩展功能

1. 增加用户头像上传功能
2. 增加帖子分页功能
3. 增加搜索功能
4. 增加管理员后台
5. 增加Redis缓存
6. 增加邮箱验证/短信验证
7. 增加第三方登录

---

以上就是该论坛系统项目的完整设计思路、系统结构和六人团队分工方案。每位成员可以根据分配的模块独立开发，同时通过Git进行代码协作与合并。