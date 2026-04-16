# 数据库设计文档

## 1. 数据库选择

### 1.1 选择理由
- **PostgreSQL**：选择PostgreSQL 15-alpine作为数据库，原因如下：
  - 支持复杂的数据类型和查询
  - 事务支持完善
  - 可靠性高
  - 适合存储结构化数据
  - 支持JSON数据类型，便于存储复杂的消息内容
  - 开源免费

### 1.2 环境配置
- **版本**：PostgreSQL 15-alpine
- **容器化**：使用Docker容器运行
- **配置**：
  - 端口：5432
  - 用户名：postgres
  - 密码：postgres
  - 数据库名：agent_brainstorm

## 2. 表结构设计

### 2.1 表关系图
```
+----------+         +-----------+         +-----------------+
|  users   |         |  sessions |         | session_members |
+----------+         +-----------+         +-----------------+
| id (PK)  |<--------| id (PK)   |<--------| id (PK)         |
| username |         | user_id   |         | session_id      |
| password |         | type      |         | agent_type      |
| created  |         | topic     |         | agent_id        |
+----------+         | status    |         | role            |
                     | created   |         +-----------------+
                     | updated   |
                     +-----------+         +-----------+
                           |              | messages  |
                           |              +-----------+
                           |              | id (PK)   |
                           |              | session_id|
                           |              | sender_type|
                           |              | sender_id |
                           |              | content   |
                           |              | timestamp |
                           |              +-----------+
                           |
                           |              +------------+
                           |              | viewpoints |
                           |              +------------+
                           |              | id (PK)    |
                           +------------->| session_id |
                                          | content    |
                                          | created    |
                                          +------------+
                                                |
                                                |
                                          +--------+
                                          | votes  |
                                          +--------+
                                          | id (PK)|
                                          | session_id|
                                          | viewpoint_id|
                                          | voter_id   |
                                          | created    |
                                          +--------+
```

### 2.2 详细表结构

#### 2.2.1 users表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 用户ID |
| username | VARCHAR(255) | UNIQUE NOT NULL | 用户名 |
| password_hash | VARCHAR(255) | NOT NULL | 密码哈希 |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

#### 2.2.2 sessions表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 会话ID |
| user_id | INTEGER | REFERENCES users(id) | 创建用户ID |
| type | VARCHAR(50) | NOT NULL | 会话类型（brainstorm, debate, chat） |
| topic | VARCHAR(255) | NOT NULL | 会话主题 |
| status | VARCHAR(50) | NOT NULL | 会话状态（created, running, paused, completed） |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新时间 |

#### 2.2.3 session_members表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 成员ID |
| session_id | INTEGER | REFERENCES sessions(id) | 会话ID |
| agent_type | VARCHAR(50) | NOT NULL | 智能体类型（user, host, mbti） |
| agent_id | VARCHAR(255) | NOT NULL | 智能体ID（用户ID或MBTI类型） |
| role | VARCHAR(50) | | 角色（辩论中使用：pro, con） |

#### 2.2.4 messages表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 消息ID |
| session_id | INTEGER | REFERENCES sessions(id) | 会话ID |
| sender_type | VARCHAR(50) | NOT NULL | 发送者类型（user, host, mbti） |
| sender_id | VARCHAR(255) | NOT NULL | 发送者ID（用户ID或MBTI类型） |
| content | TEXT | NOT NULL | 消息内容 |
| timestamp | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 发送时间 |

#### 2.2.5 viewpoints表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 观点ID |
| session_id | INTEGER | REFERENCES sessions(id) | 会话ID |
| content | TEXT | NOT NULL | 观点内容 |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

#### 2.2.6 votes表
| 字段名 | 数据类型 | 约束 | 描述 |
|-------|---------|------|------|
| id | SERIAL | PRIMARY KEY | 投票ID |
| session_id | INTEGER | REFERENCES sessions(id) | 会话ID |
| viewpoint_id | INTEGER | REFERENCES viewpoints(id) | 观点ID |
| voter_id | VARCHAR(255) | NOT NULL | 投票者ID（MBTI类型） |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 投票时间 |

## 3. 索引设计

### 3.1 主键索引
- 所有表的id字段都有主键索引

### 3.2 外键索引
- sessions表的user_id字段
- session_members表的session_id字段
- messages表的session_id字段
- viewpoints表的session_id字段
- votes表的session_id和viewpoint_id字段

### 3.3 其他索引
- users表的username字段（唯一索引）
- sessions表的status字段
- messages表的timestamp字段
- session_members表的agent_type和agent_id字段组合索引

## 4. 数据操作

### 4.1 插入操作
1. **用户注册**
   ```sql
   INSERT INTO users (username, password_hash) VALUES ('user1', 'hashed_password');
   ```

2. **创建会话**
   ```sql
   INSERT INTO sessions (user_id, type, topic, status) VALUES (1, 'brainstorm', 'AI未来发展', 'created');
   ```

3. **添加会话成员**
   ```sql
   INSERT INTO session_members (session_id, agent_type, agent_id, role) VALUES (1, 'mbti', 'INTJ', NULL);
   ```

4. **发送消息**
   ```sql
   INSERT INTO messages (session_id, sender_type, sender_id, content) VALUES (1, 'host', 'host', '欢迎大家参加头脑风暴！');
   ```

5. **添加观点**
   ```sql
   INSERT INTO viewpoints (session_id, content) VALUES (1, 'AI将改变工作方式');
   ```

6. **添加投票**
   ```sql
   INSERT INTO votes (session_id, viewpoint_id, voter_id) VALUES (1, 1, 'INTJ');
   ```

### 4.2 查询操作
1. **获取用户会话列表**
   ```sql
   SELECT * FROM sessions WHERE user_id = 1 ORDER BY created_at DESC;
   ```

2. **获取会话详情**
   ```sql
   SELECT * FROM sessions WHERE id = 1;
   ```

3. **获取会话成员**
   ```sql
   SELECT * FROM session_members WHERE session_id = 1;
   ```

4. **获取会话消息**
   ```sql
   SELECT * FROM messages WHERE session_id = 1 ORDER BY timestamp ASC;
   ```

5. **获取会话观点**
   ```sql
   SELECT * FROM viewpoints WHERE session_id = 1;
   ```

6. **获取观点投票数**
   ```sql
   SELECT viewpoint_id, COUNT(*) as vote_count FROM votes WHERE session_id = 1 GROUP BY viewpoint_id ORDER BY vote_count DESC;
   ```

### 4.3 更新操作
1. **更新会话状态**
   ```sql
   UPDATE sessions SET status = 'running', updated_at = CURRENT_TIMESTAMP WHERE id = 1;
   ```

2. **更新用户信息**
   ```sql
   UPDATE users SET password_hash = 'new_hashed_password' WHERE id = 1;
   ```

### 4.4 删除操作
1. **删除会话**
   ```sql
   DELETE FROM sessions WHERE id = 1;
   ```

2. **删除过期会话**
   ```sql
   DELETE FROM sessions WHERE status = 'completed' AND created_at < NOW() - INTERVAL '30 days';
   ```

## 5. 数据库优化

### 5.1 性能优化
1. **连接池**：使用数据库连接池减少连接开销
2. **批量操作**：批量插入消息和投票数据
3. **查询优化**：使用索引加速查询
4. **分区表**：对于消息表，可以按时间分区

### 5.2 数据安全
1. **加密**：对用户密码进行哈希加密
2. **备份**：定期备份数据库
3. **权限控制**：设置适当的数据库用户权限
4. **审计**：记录关键操作日志

## 6. 数据库迁移

### 6.1 初始迁移
1. **创建数据库**
   ```sql
   CREATE DATABASE agent_brainstorm;
   ```

2. **创建表**
   ```sql
   CREATE TABLE users (
     id SERIAL PRIMARY KEY,
     username VARCHAR(255) UNIQUE NOT NULL,
     password_hash VARCHAR(255) NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   CREATE TABLE sessions (
     id SERIAL PRIMARY KEY,
     user_id INTEGER REFERENCES users(id),
     type VARCHAR(50) NOT NULL,
     topic VARCHAR(255) NOT NULL,
     status VARCHAR(50) NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   CREATE TABLE session_members (
     id SERIAL PRIMARY KEY,
     session_id INTEGER REFERENCES sessions(id),
     agent_type VARCHAR(50) NOT NULL,
     agent_id VARCHAR(255) NOT NULL,
     role VARCHAR(50)
   );

   CREATE TABLE messages (
     id SERIAL PRIMARY KEY,
     session_id INTEGER REFERENCES sessions(id),
     sender_type VARCHAR(50) NOT NULL,
     sender_id VARCHAR(255) NOT NULL,
     content TEXT NOT NULL,
     timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   CREATE TABLE viewpoints (
     id SERIAL PRIMARY KEY,
     session_id INTEGER REFERENCES sessions(id),
     content TEXT NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   CREATE TABLE votes (
     id SERIAL PRIMARY KEY,
     session_id INTEGER REFERENCES sessions(id),
     viewpoint_id INTEGER REFERENCES viewpoints(id),
     voter_id VARCHAR(255) NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. **创建索引**
   ```sql
   CREATE INDEX idx_sessions_user_id ON sessions(user_id);
   CREATE INDEX idx_sessions_status ON sessions(status);
   CREATE INDEX idx_session_members_session_id ON session_members(session_id);
   CREATE INDEX idx_session_members_agent ON session_members(agent_type, agent_id);
   CREATE INDEX idx_messages_session_id ON messages(session_id);
   CREATE INDEX idx_messages_timestamp ON messages(timestamp);
   CREATE INDEX idx_viewpoints_session_id ON viewpoints(session_id);
   CREATE INDEX idx_votes_session_id ON votes(session_id);
   CREATE INDEX idx_votes_viewpoint_id ON votes(viewpoint_id);
   ```

### 6.2 后续迁移
- 使用数据库迁移工具（如Alembic）管理 schema 变更
- 每次变更都需要记录并测试

## 7. 数据库监控

### 7.1 监控指标
- 连接数
- 查询执行时间
- 索引使用情况
- 表大小
- 慢查询

### 7.2 监控工具
- PostgreSQL内置的pg_stat_statements
- Prometheus + Grafana
- pgAdmin

## 8. 总结

PostgreSQL数据库设计满足了多Agents头脑风暴/辩论赛项目的所有数据存储需求，包括用户管理、会话管理、消息记录、观点管理和投票管理。通过合理的表结构设计和索引优化，确保了系统的性能和可靠性。同时，通过数据库迁移和监控机制，保证了系统的可维护性和稳定性。