DataForge Multi-Agent 大数据洞察助手

一个基于 Spring Boot 3 + DuckDB + 多 Agent 协作 的企业级大数据智能分析助手。
支持 CSV 数据上传、自动数据画像、自然语言提问、SQL 自动生成、安全查询执行、业务洞察分析和 Markdown 报告生成。

项目简介

DataForge Multi-Agent 大数据洞察助手 是一个面向结构化数据分析场景的智能助手系统。
它的目标是让用户在不熟悉 SQL、不熟悉 BI 工具、不具备专业数据分析能力的情况下，也可以通过自然语言完成数据分析。

用户只需要上传 CSV 数据集，然后用中文提出问题，例如：

按地区统计销售额前5名，并给出业务洞察

系统会自动完成：

数据集理解 → 意图识别 → SQL 生成 → SQL 安全校验 → DuckDB 查询 → 业务洞察 → 报告生成

项目采用多 Agent 协作架构，每个 Agent 负责一个明确的任务节点，最终形成一条完整的数据分析链路。

项目解决的核心痛点

在实际业务或学习场景中，很多人拿到订单数据、销售数据、日志数据、运营数据后，会遇到以下问题：

不会写 SQL
普通用户无法快速从数据中提取关键指标。
会写 SQL，但不会分析业务含义
查询结果只是数字，缺少趋势、异常、原因和建议。
传统 BI 工具使用成本高
配置数据源、建模、拖拽报表都需要一定学习成本。
直接把数据交给大模型成本高且不安全
全量数据上传会带来 token 消耗、隐私泄露和结果不可控的问题。
AI 生成 SQL 存在风险
如果没有 SQL 安全护栏，可能生成删除、修改、读取本地文件等危险操作。

DataForge 的设计目标是：
让用户通过一句中文问题，安全、可审计、低成本地完成数据分析。

核心功能
1. CSV 数据上传

支持用户上传 CSV 文件，系统会自动保存数据集并生成元数据。

功能包括：

文件上传
文件类型校验
本地文件存储
数据集记录落库
数据集列表查询
2. 自动数据画像

上传 CSV 后，系统会自动分析字段信息。

包括：

字段名称
字段类型识别
缺失值数量
缺失率
distinct 数量
数值字段的最大值、最小值、平均值
样例数据

这些画像信息会作为后续 Agent 分析的上下文。

3. 自然语言数据问答

用户可以直接输入中文问题，例如：

平均利润是多少？
按渠道统计销售额
哪些记录可能存在销售额异常？
按月份查看销售额趋势

系统会根据用户问题自动判断分析意图，并生成对应查询逻辑。

4. 多 Agent 协作分析

系统内部采用多 Agent 流程编排。

核心 Agent 包括：

Agent 名称	作用
DatasetContextAgent	读取数据画像，构造分析上下文
IntentAgent	识别用户意图，例如统计、排名、趋势、异常检测
SqlPlannerAgent	根据用户问题生成只读 SELECT SQL
QueryExecuteAgent	执行 SQL 安全检查并调用 DuckDB 查询
InsightAgent	根据查询结果生成业务洞察
ReportAgent	生成 Markdown 分析报告
核心逻辑流

整体执行链路如下：

用户上传 CSV
    ↓
DatasetService 保存文件
    ↓
CsvProfiler 自动生成数据画像
    ↓
用户输入自然语言问题
    ↓
AgentOrchestrator 开始编排任务
    ↓
DatasetContextAgent 读取数据集上下文
    ↓
IntentAgent 判断用户分析意图
    ↓
SqlPlannerAgent 生成 SELECT 查询语句
    ↓
QueryExecuteAgent 做 SQL 安全校验
    ↓
DuckDB 对 CSV 执行本地 OLAP 查询
    ↓
InsightAgent 生成业务洞察
    ↓
ReportAgent 生成 Markdown 报告
    ↓
AgentRunEntity 保存执行轨迹
    ↓
前端展示 SQL、结果、洞察、报告

这个流程体现了较完整的长链推理 + 多 Agent 协作能力。

技术栈
后端技术
技术	用途
Java 17	后端开发语言
Spring Boot 3	项目基础框架
Spring MVC	REST API 接口
Spring Security	登录认证与接口保护
JWT	无状态登录令牌
Spring Data JPA	元数据持久化
H2 Database	本地元数据库
DuckDB JDBC	本地 OLAP 分析引擎
Apache Commons CSV	CSV 文件解析
SpringDoc OpenAPI	Swagger 接口文档
JUnit 5	单元测试
前端技术

当前 MVP 使用原生 HTML 页面完成基础交互，包括：

登录
上传数据集
查看数据画像
输入分析问题
展示 Agent 分析结果

后续可以替换为 Vue 3 / React 前端。

企业级项目结构

项目采用较清晰的企业级分层结构：

dataforge-agent-enterprise-mvp
├── src
│   ├── main
│   │   ├── java/com/example/dataforge
│   │   │   ├── api                 # Controller 和 DTO
│   │   │   ├── application         # 应用服务、Agent 编排
│   │   │   ├── application/agents  # 各类 Agent 实现
│   │   │   ├── domain              # 领域对象
│   │   │   ├── infrastructure      # CSV、DuckDB、JPA、文件存储
│   │   │   ├── security            # JWT 和 Spring Security
│   │   │   └── common              # 统一响应、全局异常
│   │   └── resources
│   │       ├── application.yml
│   │       └── static/index.html
│   └── test
│       └── java                    # 单元测试
├── sample_data
│   └── orders.csv                  # 示例数据
├── docs
│   └── application-material.md     # 项目申报材料
├── Dockerfile
├── docker-compose.yml
├── pom.xml
└── README.md
快速启动
环境要求

请确保本机已经安装：

JDK 17+
Maven 3.8+
启动项目

进入项目根目录：

cd dataforge-agent-enterprise-mvp

启动 Spring Boot：

mvn spring-boot:run

启动成功后访问：

http://localhost:8080
默认账号
用户名：admin
密码：admin123

登录成功后，前端会保存 JWT Token，后续请求会携带：

Authorization: Bearer <token>
示例数据

项目内置了示例 CSV 文件：

sample_data/orders.csv

你可以在页面上传这个文件，然后测试数据分析功能。

推荐测试问题

上传数据后，可以尝试输入以下问题：

按地区统计销售额前5名，并给出业务洞察
平均利润是多少
哪些记录可能存在销售额异常
按渠道统计销售额
这个数据集有多少条记录
按月份查看销售额趋势
接口说明
登录接口
POST /api/auth/login
Content-Type: application/json

请求示例：

{
  "username": "admin",
  "password": "admin123"
}

返回示例：

{
  "code": 200,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9..."
  }
}
上传数据集
POST /api/datasets
Authorization: Bearer <token>
Content-Type: multipart/form-data

参数：

file: CSV 文件
获取数据集列表
GET /api/datasets
Authorization: Bearer <token>
查看数据集画像
GET /api/datasets/{id}/profile
Authorization: Bearer <token>
Agent 智能问答
POST /api/agent/chat
Authorization: Bearer <token>
Content-Type: application/json

请求示例：

{
  "datasetId": 1,
  "question": "按地区统计销售额前5名，并给出业务洞察"
}

返回内容包括：

字段	说明
runId	本次 Agent 运行记录 ID
answer	自然语言回答
sql	自动生成的 SQL
rows	DuckDB 查询结果
insights	业务洞察列表
reportMarkdown	Markdown 分析报告
steps	每个 Agent 的执行步骤、耗时和输出
SQL 安全设计

为了避免 AI 生成危险 SQL，系统内置了 SQL 安全检查器。

当前只允许：

SELECT ...

禁止以下危险操作：

DROP
DELETE
UPDATE
INSERT
CREATE
ALTER
TRUNCATE
COPY
ATTACH
LOAD
PRAGMA

同时限制：

禁止多语句执行
禁止 SQL 注释注入
禁止访问任意本地文件
只能查询系统内部创建的虚拟表 dataset
数据分析设计思路

本项目不是简单地把 CSV 原文全部交给 AI，而是采用更适合企业场景的分析方式：

1. 后端先解析 CSV
2. 生成字段画像和样例数据
3. Agent 根据画像判断用户意图
4. 生成安全 SQL
5. DuckDB 执行真实查询
6. Agent 根据聚合结果生成洞察

这样做的好处是：

token 消耗更低
不需要上传全量数据给大模型
查询结果更准确
每一步都可以审计
更符合企业数据安全要求
Token 成本控制

默认 MVP 使用规则版 SQL Planner，可以做到：

LLM Token 消耗：0

如果后续接入大模型，可以采用以下策略控制成本：

1. 不上传全量 CSV，只发送字段画像和少量样例行
2. SQL 生成阶段只传表结构、字段解释和用户问题
3. 洞察生成阶段只传聚合结果前 20 - 50 行
4. 报告生成阶段使用模板化输出
5. 相同问题可以基于 AgentRun 记录做缓存

预估单次分析成本：

规则版：0 Token
LLM 增强版：约 1200 - 2500 Token / 次
Docker 运行

如果你想使用 Docker 启动项目，可以执行：

docker compose up --build

启动后访问：

http://localhost:8080
Swagger 接口文档

启动项目后访问：

http://localhost:8080/swagger-ui.html

可以在 Swagger 页面中测试登录、上传数据集、查看画像和 Agent 问答接口。

H2 数据库控制台

本项目默认使用 H2 作为元数据库。

访问地址：

http://localhost:8080/h2-console

连接信息：

JDBC URL: jdbc:h2:file:./data/dataforge-meta
User: sa
Password: 空
项目亮点
多 Agent 协作架构
每个 Agent 职责单一，链路清晰，方便扩展和调试。
企业级分层结构
项目按照 Controller、Application、Domain、Infrastructure、Security 等层次拆分。
SQL 安全护栏
防止生成危险 SQL，保证系统只执行安全的查询语句。
DuckDB 本地 OLAP 查询
不依赖复杂大数据集群，也能完成较高性能的 CSV 分析。
数据画像能力
上传数据后自动分析字段类型、缺失率、distinct 和数值分布。
Agent 执行轨迹审计
每一次分析都会保存执行步骤，便于复盘和问题排查。
可扩展为真实企业数据 Copilot
后续可以对接 MySQL、PostgreSQL、ClickHouse、Hive、Spark、Flink 等数据源。
后续扩展方向

后续可以继续升级为更完整的企业级系统：

接入真实大模型：OpenAI、DeepSeek、Qwen、GLM
支持 MySQL / PostgreSQL / ClickHouse / Hive 数据源
支持多租户和 RBAC 权限控制
支持数据脱敏和字段级权限
支持异步 Agent 任务队列
支持 WebSocket 实时返回 Agent 执行进度
支持报告导出为 PDF / Word
支持图表自动生成
支持 RAG 知识库解释业务指标口径
支持任务调度和日报、周报自动生成
支持 Prometheus + Grafana 监控
项目定位

本项目适合作为：

Java 后端进阶项目
多 Agent 实战项目
大数据分析类创新项目
简历项目
课程设计项目
企业内部数据分析 Copilot 原型

它不是一个简单的 CRUD 项目，而是一个融合了：

Spring Boot 企业级后端
数据分析
DuckDB OLAP
SQL 安全
多 Agent 编排
自然语言问答
报告生成
