# 前端存储解决方案（Demo版）

## 1. 需求分析

**用户需求**：
- 快速简单做一个demo
- 不需要A用户看到B用户的数据
- 优先考虑前端存储方案

**核心需求**：
- 数据只存储在用户本地
- 无需服务器和数据库
- 部署简单，成本低
- 功能完整，满足基本使用需求

## 2. 前端存储方案选择

### 2.1 存储方案对比

| 存储方案 | 容量 | 持久化 | 适用场景 | 优缺点 |
|---------|------|--------|----------|--------|
| localStorage | 5-10MB | 持久化 | 小数据存储 | 优点：简单易用；缺点：容量有限，只支持字符串 |
| sessionStorage | 5-10MB | 会话结束后清除 | 临时数据存储 | 优点：简单易用；缺点：容量有限，只支持字符串，会话结束后清除 |
| IndexedDB | 理论上无限制 | 持久化 | 大数据存储 | 优点：容量大，支持复杂查询，支持事务；缺点：API复杂 |

### 2.2 推荐方案

**推荐使用 IndexedDB**，理由：
- 容量大，适合存储会话、消息、观点等数据
- 支持复杂查询，便于检索数据
- 支持事务，确保数据一致性
- 持久化存储，用户刷新页面后数据仍然存在

## 3. 实现方案

### 3.1 数据结构设计

**1. 用户信息**：
- 存储用户配置（如LLM API Key）

**2. 会话信息**：
- 会话ID
- 会话类型（brainstorm、debate、chat）
- 主题
- 状态（created、running、paused、completed）
- 创建时间
- 更新时间

**3. 会话成员**：
- 会话ID
- 智能体类型（user、host、mbti）
- 智能体ID（用户ID或MBTI类型）
- 角色（辩论中使用：pro、con）

**4. 消息记录**：
- 消息ID
- 会话ID
- 发送者类型（user、host、mbti）
- 发送者ID（用户ID或MBTI类型）
- 内容
- 时间戳

**5. 观点记录**：
- 观点ID
- 会话ID
- 内容
- 创建时间

**6. 投票记录**：
- 投票ID
- 会话ID
- 观点ID
- 投票者ID（MBTI类型）
- 投票时间

### 3.2 前端存储实现

**1. IndexedDB封装**：
```javascript
// db.js
class DBService {
  constructor() {
    this.dbName = 'agentBrainstormDB';
    this.dbVersion = 1;
    this.db = null;
  }

  async open() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.dbVersion);

      request.onupgradeneeded = (event) => {
        const db = event.target.result;

        // 创建存储对象
        if (!db.objectStoreNames.contains('users')) {
          db.createObjectStore('users', { keyPath: 'id' });
        }

        if (!db.objectStoreNames.contains('sessions')) {
          const sessionStore = db.createObjectStore('sessions', { keyPath: 'id' });
          sessionStore.createIndex('user_id', 'user_id', { unique: false });
        }

        if (!db.objectStoreNames.contains('session_members')) {
          const memberStore = db.createObjectStore('session_members', { keyPath: 'id' });
          memberStore.createIndex('session_id', 'session_id', { unique: false });
        }

        if (!db.objectStoreNames.contains('messages')) {
          const messageStore = db.createObjectStore('messages', { keyPath: 'id' });
          messageStore.createIndex('session_id', 'session_id', { unique: false });
          messageStore.createIndex('timestamp', 'timestamp', { unique: false });
        }

        if (!db.objectStoreNames.contains('viewpoints')) {
          const viewpointStore = db.createObjectStore('viewpoints', { keyPath: 'id' });
          viewpointStore.createIndex('session_id', 'session_id', { unique: false });
        }

        if (!db.objectStoreNames.contains('votes')) {
          const voteStore = db.createObjectStore('votes', { keyPath: 'id' });
          voteStore.createIndex('session_id', 'session_id', { unique: false });
          voteStore.createIndex('viewpoint_id', 'viewpoint_id', { unique: false });
        }
      };

      request.onsuccess = (event) => {
        this.db = event.target.result;
        resolve(this.db);
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }

  async add(storeName, data) {
    await this.open();
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.add(data);

      request.onsuccess = (event) => {
        resolve(event.target.result);
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }

  async get(storeName, key) {
    await this.open();
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.get(key);

      request.onsuccess = (event) => {
        resolve(event.target.result);
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }

  async getAll(storeName, indexName, value) {
    await this.open();
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], 'readonly');
      const store = transaction.objectStore(storeName);
      let request;

      if (indexName && value) {
        const index = store.index(indexName);
        request = index.getAll(value);
      } else {
        request = store.getAll();
      }

      request.onsuccess = (event) => {
        resolve(event.target.result);
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }

  async update(storeName, data) {
    await this.open();
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.put(data);

      request.onsuccess = (event) => {
        resolve(event.target.result);
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }

  async delete(storeName, key) {
    await this.open();
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([storeName], 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.delete(key);

      request.onsuccess = (event) => {
        resolve();
      };

      request.onerror = (event) => {
        reject(event.target.error);
      };
    });
  }
}

export const dbService = new DBService();
```

**2. 数据操作示例**：
```javascript
// 存储会话
const session = {
  id: 1,
  user_id: 'user1',
  type: 'brainstorm',
  topic: 'AI未来发展',
  status: 'created',
  created_at: new Date().toISOString(),
  updated_at: new Date().toISOString()
};
await dbService.add('sessions', session);

// 获取会话列表
const sessions = await dbService.getAll('sessions');

// 存储消息
const message = {
  id: 1,
  session_id: 1,
  sender_type: 'host',
  sender_id: 'host',
  content: '欢迎大家参加头脑风暴！',
  timestamp: new Date().toISOString()
};
await dbService.add('messages', message);

// 获取会话消息
const messages = await dbService.getAll('messages', 'session_id', 1);
```

### 3.3 前端状态管理

**使用Redux Toolkit管理状态**：
```javascript
// store/slices/sessionSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { dbService } from '../../services/db';

// 异步操作
export const createSession = createAsyncThunk(
  'session/create',
  async (sessionData) => {
    const session = {
      id: Date.now(),
      user_id: 'current_user', // 固定用户ID
      ...sessionData,
      status: 'created',
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString()
    };
    await dbService.add('sessions', session);
    
    // 添加会话成员
    for (const member of sessionData.members) {
      await dbService.add('session_members', {
        id: Date.now() + Math.random(),
        session_id: session.id,
        ...member
      });
    }
    
    return session;
  }
);

export const getSessions = createAsyncThunk(
  'session/getAll',
  async () => {
    return await dbService.getAll('sessions');
  }
);

// Slice
const sessionSlice = createSlice({
  name: 'session',
  initialState: {
    sessions: [],
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(createSession.pending, (state) => {
        state.loading = true;
      })
      .addCase(createSession.fulfilled, (state, action) => {
        state.loading = false;
        state.sessions.push(action.payload);
      })
      .addCase(createSession.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(getSessions.pending, (state) => {
        state.loading = true;
      })
      .addCase(getSessions.fulfilled, (state, action) => {
        state.loading = false;
        state.sessions = action.payload;
      })
      .addCase(getSessions.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export default sessionSlice.reducer;
```

## 4. 部署方案

### 4.1 静态网站部署

**1. 构建前端应用**：
- 使用Vite构建前端应用
- 生成静态文件（HTML、CSS、JavaScript）

**2. 部署平台**：
- **GitHub Pages**：免费，适合静态网站
- **Vercel**：免费，部署简单，支持自动构建
- **Netlify**：免费，支持自动构建和部署

**3. 部署步骤**：
- 将前端代码上传到GitHub仓库
- 在Vercel或Netlify上连接GitHub仓库
- 配置构建命令和输出目录
- 部署完成后获得访问URL

### 4.2 本地运行

**1. 安装依赖**：
```bash
npm install
```

**2. 启动开发服务器**：
```bash
npm run dev
```

**3. 构建生产版本**：
```bash
npm run build
```

**4. 本地预览**：
```bash
npm run preview
```

## 5. 优缺点分析

### 5.1 优点

1. **部署简单**：无需服务器和数据库，只需部署静态网站
2. **成本低**：使用免费的静态网站托管服务
3. **开发快速**：前端存储API简单易用，无需后端开发
4. **数据隔离**：每个用户的数据只存储在本地，无需担心数据共享

### 5.2 缺点

1. **数据丢失风险**：用户清除浏览器数据后，数据会丢失
2. **跨设备访问**：数据只存储在当前设备，无法跨设备访问
3. **存储容量**：虽然IndexedDB容量较大，但仍有浏览器限制
4. **安全性**：敏感数据（如LLM API Key）存储在前端，存在安全风险
5. **功能限制**：无法实现实时通信（如WebSocket），需要使用轮询或其他方式

## 6. 适用场景

**适合的场景**：
- 快速原型开发和demo展示
- 个人使用，不需要数据共享
- 功能验证和测试
- 临时项目或一次性活动

**不适合的场景**：
- 多用户协作
- 数据需要长期保存
- 敏感数据处理
- 高并发场景

## 7. 总结

对于快速简单的demo需求，前端存储方案是完全可行的。通过使用IndexedDB存储数据，可以实现完整的功能，同时避免了服务器和数据库的部署成本。虽然存在数据丢失风险和跨设备访问限制，但对于demo来说，这些限制是可以接受的。

建议使用IndexedDB作为存储方案，结合静态网站部署平台（如Vercel或Netlify），可以快速部署一个功能完整的demo，满足用户的需求。