# Jupiter

一个个性化活动推荐 Web 应用，帮助用户根据地理位置和偏好发现附近的活动。

[English](README.md) | 中文

## 功能特性

- **活动搜索**：使用 TicketMaster API 搜索附近的活动
- **个性化推荐**：根据用户收藏的类别推荐相关活动
- **收藏管理**：保存和管理喜欢的活动
- **地理定位**：发现 50 英里范围内的活动
- **分类筛选**：按类别浏览活动（音乐、体育、艺术等）

## 技术栈

### 后端
- **Java Servlet** - RESTful API 接口
- **MySQL** - 用户、活动和收藏数据持久化
- **TicketMaster Discovery API** - 实时活动数据源

### 前端
- **HTML5 / CSS3** - 响应式 UI 设计
- **JavaScript** - 动态内容加载和用户交互
- **Font Awesome** - 图标库

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                         前端层                               │
│                   (HTML/CSS/JavaScript)                      │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP/JSON
┌─────────────────────────▼───────────────────────────────────┐
│                     RPC 层 (Servlets)                        │
│         SearchItem | ItemHistory | RecommendItem             │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                       业务逻辑层                              │
│                   地理推荐算法模块                            │
└──────────┬──────────────────────────────────┬───────────────┘
           │                                  │
┌──────────▼──────────┐            ┌──────────▼──────────┐
│     外部 API        │            │      数据访问层      │
│  TicketMaster API   │            │    MySQL 数据库     │
└─────────────────────┘            └─────────────────────┘
```

## API 接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/search` | GET | 根据位置和关键词搜索活动 |
| `/history` | GET/POST/DELETE | 管理用户收藏的活动 |
| `/recommendation` | GET | 获取个性化活动推荐 |

## 数据库设计

```sql
-- 用户表
users (user_id, password, first_name, last_name)

-- 活动表
items (item_id, name, rating, address, image_url, url, distance)

-- 活动分类表
categories (item_id, category)

-- 用户收藏历史表
history (user_id, item_id, last_favor_time)
```

### ER 图

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    users     │       │   history    │       │    items     │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ user_id (PK) │──────<│ user_id (FK) │       │ item_id (PK) │
│ password     │       │ item_id (FK) │>──────│ name         │
│ first_name   │       │ last_favor   │       │ rating       │
│ last_name    │       │    _time     │       │ address      │
└──────────────┘       └──────────────┘       │ image_url    │
                                              │ url          │
                                              │ distance     │
                                              └──────┬───────┘
                                                     │
                                              ┌──────▼───────┐
                                              │  categories  │
                                              ├──────────────┤
                                              │ item_id (FK) │
                                              │ category     │
                                              └──────────────┘
```

## 快速开始

### 环境要求

- JDK 8 或更高版本
- Apache Tomcat 9.x
- MySQL 8.0
- Eclipse IDE（可选，用于开发）

### 数据库配置

1. 创建 MySQL 数据库：
```sql
CREATE DATABASE laiproject;
```

2. 修改数据库配置 `src/main/java/db/mysql/MySQLDBUtil.java`：
```java
private static final String HOSTNAME = "localhost";
private static final String PORT_NUM = "3306";
private static final String USERNAME = "你的用户名";
private static final String PASSWORD = "你的密码";
```

3. 运行 `MySQLTableCreation.java` 初始化数据表：
```bash
java -cp .:mysql-connector-java-8.0.11.jar db.mysql.MySQLTableCreation
```

### 部署步骤

1. 将项目打包为 WAR 文件
2. 部署到 Apache Tomcat
3. 访问 `http://localhost:8080/Jupiter`

### 测试账号

- **用户 ID**：1111
- **密码**：3229c1097c00d497a0fd282d586be050（MD5 加密）

## 推荐算法

系统采用基于内容的协同过滤方法：

```
用户收藏 ──> 提取类别 ──> 类别排序 ──> 按类别搜索 ──> 过滤&排序 ──> 推荐结果
```

### 算法步骤

1. **获取用户收藏** - 查询用户所有收藏的活动
2. **提取类别** - 统计收藏活动的类别出现次数
3. **类别排序** - 按频率降序排列类别
4. **按类别搜索** - 调用 TicketMaster API 搜索每个类别的活动
5. **过滤排序** - 移除已收藏的活动，按距离排序

### 核心代码

```java
// GeoRecommendation.java
public List<Item> recommendItems(String userId, double lat, double lon) {
    // 1. 获取用户收藏
    Set<String> favoriteItemIds = conn.getFavoriteItemIds(userId);

    // 2. 统计类别频率
    Map<String, Integer> allCategories = new HashMap<>();
    for (String itemId : favoriteItemIds) {
        Set<String> categories = conn.getCategories(itemId);
        for (String category : categories) {
            allCategories.put(category,
                allCategories.getOrDefault(category, 0) + 1);
        }
    }

    // 3. 按频率排序，搜索并过滤
    // ...
}
```

## 项目结构

```
Jupiter/
├── src/main/java/
│   ├── algorithm/          # 推荐算法
│   │   └── GeoRecommendation.java
│   ├── db/                 # 数据访问层
│   │   ├── DBConnection.java
│   │   ├── DBConnectionFactory.java
│   │   └── mysql/          # MySQL 实现
│   │       ├── MySQLConnection.java
│   │       ├── MySQLDBUtil.java
│   │       └── MySQLTableCreation.java
│   ├── entity/             # 数据实体
│   │   └── Item.java
│   ├── external/           # 外部 API 集成
│   │   ├── GeoHash.java
│   │   └── TicketMasterAPI.java
│   └── rpc/                # Servlet 接口
│       ├── ItemHistory.java
│       ├── RecommendItem.java
│       ├── RpcHelper.java
│       └── SearchItem.java
├── src/main/webapp/
│   ├── scripts/            # JavaScript 文件
│   │   └── main.js
│   ├── styles/             # CSS 样式文件
│   │   └── main.css
│   ├── WEB-INF/lib/        # 依赖 JAR 包
│   │   ├── java-json.jar
│   │   └── mysql-connector-java-8.0.11.jar
│   └── index.html          # 主页面
├── README.md               # 英文文档
└── README_CN.md            # 中文文档
```

## 许可证

本项目仅供学习交流使用。

## 致谢

- [TicketMaster API](https://developer.ticketmaster.com/) - 活动数据来源
- [Font Awesome](https://fontawesome.com/) - 图标库
