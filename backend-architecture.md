# 后端架构设计文档

## 1. 技术栈

### 1.1 核心技术
- **语言**：Python 3.11.14
- **框架**：LangChain 1.2.10
- **扩展**：LangChain-DeepSeek 1.0.1
- **图框架**：LangGraph
- **数据库**：PostgreSQL 15-alpine（可选）
- **API框架**：FastAPI
- **实时通信**：WebSocket

### 1.2 依赖管理
```bash
# requirements.txt
langchain==1.2.10
langchain-deepseek==1.0.1
langgraph
fastapi
uvicorn
websockets
psycopg2-binary
python-dotenv
```

## 2. 系统架构

### 2.1 整体架构
```
┌─────────────────────┐
│      前端应用        │
└──────────┬──────────┘
           │ WebSocket/HTTP
┌──────────▼──────────┐
│      API服务         │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│    会话管理服务       │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│    LangGraph图执行   │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│    LLM调用服务       │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│     数据库服务        │
└─────────────────────┘
```

### 2.2 核心模块
1. **API服务**：处理前端请求，提供RESTful API和WebSocket连接
2. **会话管理服务**：管理会话生命周期，包括创建、启动、暂停、结束会话
3. **LangGraph图执行**：执行三种模式的图流程
4. **LLM调用服务**：封装LLM调用，支持并发调用
5. **数据库服务**：存储会话信息、消息记录、观点和投票

## 3. Agent设计

### 3.1 Host Agent
- **作用**：主持人智能体，用于推进讨论，总结观点
- **实现**：基于LangChain的Agent，使用系统提示词定义其角色
- **核心功能**：
  - 开场白生成
  - 讨论监听和管理
  - 观点总结
  - 投票结果分析
  - 最终结论生成

### 3.2 MBTI Agent
- **作用**：带有MBTI人格的智能体
- **实现**：基于LangChain的Agent，使用MBTI相关的系统提示词
- **MBTI类型**：16种MBTI人格类型
- **核心功能**：
  - 根据MBTI人格生成观点
  - 参与讨论
  - 对观点进行投票

## 4. 模式实现

### 4.1 头脑风暴Graph

#### 节点设计
1. **开场白节点**
   - 作用：Host Agent根据用户的topic进行开场白
   - 输入：topic
   - 输出：开场白消息
   - 结束标志：Host Agent发出了开场白

2. **第一轮观点表达节点**
   - 作用：多个MBTI Agent对开场白进行观点表达（无记忆）
   - 输入：开场白消息
   - 输出：多个MBTI Agent的观点
   - 结束标志：所有选中的MBTI Agent都表达了观点
   - 并发处理：同时调用多个LLM

3. **第一轮讨论节点**
   - 作用：多个MBTI Agent进行第一轮讨论（有记忆）
   - 输入：所有历史消息
   - 输出：多个MBTI Agent的讨论内容
   - 结束标志：
     - Host Agent总结出的观点 >= 5，或
     - 各MBTI Agent的总发言次数 >= 4 * MBTI Agent数量
   - 并发处理：同时调用多个LLM
   - Host Agent监听：总结已有观点，引导讨论方向

4. **观点投票节点**
   - 作用：Host Agent总结观点，MBTI Agent投票
   - 输入：所有历史消息
   - 输出：
     - Host Agent的观点总结
     - MBTI Agent的投票结果
     - Host Agent的后续讨论引导
   - 结束标志：Host Agent发出了引导后续讨论的开场白

5. **第二轮讨论节点**
   - 作用：多个MBTI Agent针对选定观点进行第二轮讨论（有记忆，落地讨论）
   - 输入：所有历史消息
   - 输出：多个MBTI Agent的讨论内容
   - 结束标志：
     - Host Agent认为方案已经具体可行，或
     - 各MBTI Agent的总发言次数 >= 2 * MBTI Agent数量
   - 并发处理：同时调用多个LLM
   - Host Agent监听：总结已有观点，引导讨论方向

6. **总结节点**
   - 作用：Host Agent根据前面讨论的内容进行总结
   - 输入：所有历史消息
   - 输出：最终结论（包括topic,中间的观点列表以及投票数和最后的总结）
   - 结束标志：Host Agent发出了最终结论

### 4.2 辩论赛Graph

#### 节点设计
1. **开场节点**
   - 作用：Host Agent介绍辩论议题和规则
   - 输入：辩论议题
   - 输出：开场白消息
   - 结束标志：Host Agent发出了开场白

2. **正方立论节点**
   - 作用：正方MBTI Agent依次进行立论
   - 输入：开场白消息
   - 输出：正方辩手的立论内容
   - 结束标志：所有正方辩手都完成立论
   - 并发处理：同时调用多个LLM

3. **反方立论节点**
   - 作用：反方MBTI Agent依次进行立论
   - 输入：正方立论内容
   - 输出：反方辩手的立论内容
   - 结束标志：所有反方辩手都完成立论
   - 并发处理：同时调用多个LLM

4. **自由辩论节点**
   - 作用：正反方MBTI Agent进行自由辩论
   - 输入：所有历史消息
   - 输出：辩手的辩论内容
   - 结束标志：达到预定的辩论时间或回合数
   - 并发处理：同时调用多个LLM
   - Host Agent监听：维护辩论秩序，控制发言时间

5. **总结陈词节点**
   - 作用：正反方MBTI Agent依次进行总结陈词
   - 输入：所有历史消息
   - 输出：辩手的总结陈词
   - 结束标志：所有辩手都完成总结陈词
   - 并发处理：同时调用多个LLM

6. **结果宣布节点**
   - 作用：Host Agent宣布辩论结果
   - 输入：所有历史消息
   - 输出：辩论结果和总结
   - 结束标志：Host Agent发出了辩论结果

### 4.3 闲聊模式

- **实现**：有记忆的MBTI对话
- **核心功能**：
  - 基于MBTI人格的对话生成
  - 对话历史记忆
  - 实时响应

## 5. 并发处理

### 5.1 LLM并发调用
- **实现**：使用Python的`asyncio`和`concurrent.futures`
- **策略**：
  - 为每个MBTI Agent创建独立的LLM调用任务
  - 使用`asyncio.gather()`并发执行多个任务
  - 谁先得到LLM的输出就谁先回答

### 5.2 消息处理
- **实现**：使用WebSocket进行实时消息传递
- **策略**：
  - 消息到达后立即处理
  - 消息排序：根据时间戳排序
  - 消息去重：确保每条消息只处理一次

## 6. 数据库设计

### 6.1 表结构

1. **users表**
   - id: SERIAL PRIMARY KEY
   - username: VARCHAR(255) UNIQUE NOT NULL
   - password_hash: VARCHAR(255) NOT NULL
   - created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

2. **sessions表**
   - id: SERIAL PRIMARY KEY
   - user_id: INTEGER REFERENCES users(id)
   - type: VARCHAR(50) NOT NULL  -- brainstorm, debate, chat
   - topic: VARCHAR(255) NOT NULL
   - status: VARCHAR(50) NOT NULL  -- created, running, paused, completed
   - created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   - updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

3. **session_members表**
   - id: SERIAL PRIMARY KEY
   - session_id: INTEGER REFERENCES sessions(id)
   - agent_type: VARCHAR(50) NOT NULL  -- user, host, mbti
   - agent_id: VARCHAR(255) NOT NULL  -- user id or mbti type
   - role: VARCHAR(50)  -- for debate: pro, con

4. **messages表**
   - id: SERIAL PRIMARY KEY
   - session_id: INTEGER REFERENCES sessions(id)
   - sender_type: VARCHAR(50) NOT NULL  -- user, host, mbti
   - sender_id: VARCHAR(255) NOT NULL  -- user id or mbti type
   - content: TEXT NOT NULL
   - timestamp: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

5. **viewpoints表**
   - id: SERIAL PRIMARY KEY
   - session_id: INTEGER REFERENCES sessions(id)
   - content: TEXT NOT NULL
   - created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

6. **votes表**
   - id: SERIAL PRIMARY KEY
   - session_id: INTEGER REFERENCES sessions(id)
   - viewpoint_id: INTEGER REFERENCES viewpoints(id)
   - voter_id: VARCHAR(255) NOT NULL  -- mbti type
   - created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

### 6.2 数据库操作
- **插入操作**：记录会话、消息、观点和投票
- **查询操作**：获取历史会话、消息记录、观点和投票结果
- **更新操作**：更新会话状态
- **删除操作**：清理过期会话

## 7. API设计

### 7.1 RESTful API

1. **用户相关**
   - POST /api/users/register - 注册新用户
   - POST /api/users/login - 用户登录
   - GET /api/users/profile - 获取用户信息

2. **会话相关**
   - POST /api/sessions - 创建新会话
   - GET /api/sessions - 获取用户的会话列表
   - GET /api/sessions/{id} - 获取会话详情
   - PUT /api/sessions/{id}/start - 开始会话
   - PUT /api/sessions/{id}/pause - 暂停会话
   - PUT /api/sessions/{id}/stop - 结束会话

3. **消息相关**
   - GET /api/sessions/{id}/messages - 获取会话消息
   - POST /api/sessions/{id}/messages - 发送消息

### 7.2 WebSocket API

- **连接**：ws://localhost:8000/ws/sessions/{id}
- **消息类型**：
  - message - 新消息
  - status - 会话状态变更
  - error - 错误信息

## 8. 部署方案

### 8.1 本地部署
- **环境配置**：
  - Python 3.11.14
  - PostgreSQL 15-alpine
  - 安装依赖：`pip install -r requirements.txt`
- **启动服务**：
  - 启动数据库：`docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15-alpine`
  - 启动API服务：`uvicorn app.main:app --reload`

### 8.2 生产部署
- **容器化**：使用Docker容器
- **编排**：使用Docker Compose
- **服务扩展**：使用Nginx作为反向代理
- **监控**：使用Prometheus和Grafana

## 9. 性能优化

### 9.1 LLM调用优化
- **缓存**：缓存LLM响应
- **批处理**：批量处理相似请求
- **超时设置**：合理设置LLM调用超时

### 9.2 数据库优化
- **索引**：为常用查询字段创建索引
- **连接池**：使用数据库连接池
- **读写分离**：针对高并发场景

### 9.3 API优化
- **异步处理**：使用FastAPI的异步特性
- **WebSocket优化**：合理管理WebSocket连接
- **负载均衡**：使用负载均衡器分发请求

## 10. 安全考虑

### 10.1 API安全
- **认证**：使用JWT进行身份认证
- **授权**：基于角色的访问控制
- **HTTPS**：使用HTTPS加密传输

### 10.2 LLM安全
- **输入验证**：验证用户输入，防止prompt注入
- **输出过滤**：过滤LLM输出，防止有害内容
- **速率限制**：限制LLM调用频率，防止滥用

### 10.3 数据安全
- **加密**：加密存储敏感数据
- **备份**：定期备份数据库
- **审计**：记录关键操作日志