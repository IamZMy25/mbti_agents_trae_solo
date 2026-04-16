# 前端实现文档

## 1. 技术栈

### 1.1 核心技术
- **框架**：React
- **状态管理**：Redux Toolkit
- **UI库**：Ant Design
- **样式**：CSS-in-JS (styled-components)
- **构建工具**：Vite
- **数据存储**：IndexedDB
- **HTTP客户端**：Axios

### 1.2 依赖管理
```bash
# package.json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@reduxjs/toolkit": "^1.9.5",
    "react-redux": "^8.1.1",
    "antd": "^5.8.4",
    "styled-components": "^6.0.7",
    "axios": "^1.4.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.3",
    "vite": "^4.4.5"
  }
}
```

## 2. 项目结构

```
frontend/
├── public/
├── src/
│   ├── components/
│   │   ├── ChatRoom/
│   │   ├── SessionList/
│   │   ├── CreateSession/
│   │   ├── Settings/
│   │   └── Login/
│   ├── pages/
│   │   ├── Home/
│   │   ├── Chat/
│   │   ├── Settings/
│   │   └── Login/
│   ├── services/
│   │   ├── api.js
│   │   └── db.js
│   ├── store/
│   │   ├── slices/
│   │   └── index.js
│   ├── utils/
│   ├── App.js
│   └── main.js
├── package.json
└── vite.config.js
```

## 3. 核心组件

### 3.1 ChatRoom组件
- **功能**：显示聊天消息，发送消息，控制讨论
- **实现**：
  ```jsx
  import React, { useState, useEffect, useRef } from 'react';
  import { useSelector, useDispatch } from 'react-redux';
  import { sendMessage, startDiscussion, pauseDiscussion } from '../../store/slices/chatSlice';

  const ChatRoom = ({ sessionId }) => {
    const [message, setMessage] = useState('');
    const messagesEndRef = useRef(null);
    const { messages, session } = useSelector(state => state.chat);
    const dispatch = useDispatch();

    useEffect(() => {
      scrollToBottom();
    }, [messages]);

    const scrollToBottom = () => {
      messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
    };

    const handleSendMessage = () => {
      if (message.trim()) {
        dispatch(sendMessage({ sessionId, content: message }));
        setMessage('');
      }
    };

    const handleStartDiscussion = () => {
      dispatch(startDiscussion(sessionId));
    };

    const handlePauseDiscussion = () => {
      dispatch(pauseDiscussion(sessionId));
    };

    return (
      <div className="chat-room">
        <div className="chat-header">
          <h2>{session?.topic}</h2>
          <div className="chat-controls">
            {session?.status === 'created' && (
              <button onClick={handleStartDiscussion}>开始讨论</button>
            )}
            {session?.status === 'running' && (
              <button onClick={handlePauseDiscussion}>暂停讨论</button>
            )}
          </div>
        </div>
        <div className="chat-messages">
          {messages.map((msg) => (
            <div key={msg.id} className={`message ${msg.sender_type}`}>
              <div className="message-sender">{msg.sender_id}</div>
              <div className="message-content">{msg.content}</div>
              <div className="message-time">{msg.timestamp}</div>
            </div>
          ))}
          <div ref={messagesEndRef} />
        </div>
        <div className="chat-input">
          <input
            type="text"
            value={message}
            onChange={(e) => setMessage(e.target.value)}
            onKeyPress={(e) => e.key === 'Enter' && handleSendMessage()}
            placeholder="输入消息..."
          />
          <button onClick={handleSendMessage}>发送</button>
        </div>
      </div>
    );
  };

  export default ChatRoom;
  ```

### 3.2 CreateSession组件
- **功能**：创建新会话（头脑风暴/辩论赛/闲聊）
- **实现**：
  ```jsx
  import React, { useState } from 'react';
  import { useDispatch } from 'react-redux';
  import { createSession } from '../../store/slices/sessionSlice';

  const CreateSession = () => {
    const [sessionType, setSessionType] = useState('brainstorm');
    const [topic, setTopic] = useState('');
    const [selectedMbtis, setSelectedMbtis] = useState([]);
    const [proMbtis, setProMbtis] = useState([]);
    const [conMbtis, setConMbtis] = useState([]);
    const dispatch = useDispatch();

    const mbtiTypes = [
      'INTJ', 'INTP', 'ENTJ', 'ENTP',
      'INFJ', 'INFP', 'ENFJ', 'ENFP',
      'ISTJ', 'ISFJ', 'ESTJ', 'ESFJ',
      'ISTP', 'ISFP', 'ESTP', 'ESFP'
    ];

    const handleCreateSession = () => {
      let members = [];
      
      if (sessionType === 'brainstorm') {
        members = selectedMbtis.map(mbti => ({
          agent_type: 'mbti',
          agent_id: mbti
        }));
      } else if (sessionType === 'debate') {
        members = [
          ...proMbtis.map(mbti => ({
            agent_type: 'mbti',
            agent_id: mbti,
            role: 'pro'
          })),
          ...conMbtis.map(mbti => ({
            agent_type: 'mbti',
            agent_id: mbti,
            role: 'con'
          }))
        ];
      } else if (sessionType === 'chat') {
        members = selectedMbtis.map(mbti => ({
          agent_type: 'mbti',
          agent_id: mbti
        }));
      }

      dispatch(createSession({
        type: sessionType,
        topic,
        members
      }));
    };

    return (
      <div className="create-session">
        <h2>创建新会话</h2>
        <div className="session-type">
          <label>
            <input
              type="radio"
              value="brainstorm"
              checked={sessionType === 'brainstorm'}
              onChange={() => setSessionType('brainstorm')}
            />
            头脑风暴
          </label>
          <label>
            <input
              type="radio"
              value="debate"
              checked={sessionType === 'debate'}
              onChange={() => setSessionType('debate')}
            />
            辩论赛
          </label>
          <label>
            <input
              type="radio"
              value="chat"
              checked={sessionType === 'chat'}
              onChange={() => setSessionType('chat')}
            />
            闲聊
          </label>
        </div>
        <div className="topic-input">
          <label>主题：</label>
          <input
            type="text"
            value={topic}
            onChange={(e) => setTopic(e.target.value)}
            placeholder="输入会话主题"
          />
        </div>
        {sessionType === 'brainstorm' && (
          <div className="mbti-selection">
            <label>选择MBTI人格（最多16个）：</label>
            <div className="mbti-options">
              {mbtiTypes.map(mbti => (
                <label key={mbti}>
                  <input
                    type="checkbox"
                    checked={selectedMbtis.includes(mbti)}
                    onChange={(e) => {
                      if (e.target.checked) {
                        if (selectedMbtis.length < 16) {
                          setSelectedMbtis([...selectedMbtis, mbti]);
                        }
                      } else {
                        setSelectedMbtis(selectedMbtis.filter(m => m !== mbti));
                      }
                    }}
                  />
                  {mbti}
                </label>
              ))}
            </div>
          </div>
        )}
        {sessionType === 'debate' && (
          <>
            <div className="mbti-selection">
              <label>正方辩手（4位）：</label>
              <div className="mbti-options">
                {mbtiTypes.map(mbti => (
                  <label key={mbti}>
                    <input
                      type="checkbox"
                      checked={proMbtis.includes(mbti)}
                      onChange={(e) => {
                        if (e.target.checked) {
                          if (proMbtis.length < 4) {
                            setProMbtis([...proMbtis, mbti]);
                          }
                        } else {
                          setProMbtis(proMbtis.filter(m => m !== mbti));
                        }
                      }}
                    />
                    {mbti}
                  </label>
                ))}
              </div>
            </div>
            <div className="mbti-selection">
              <label>反方辩手（4位）：</label>
              <div className="mbti-options">
                {mbtiTypes.map(mbti => (
                  <label key={mbti}>
                    <input
                      type="checkbox"
                      checked={conMbtis.includes(mbti)}
                      onChange={(e) => {
                        if (e.target.checked) {
                          if (conMbtis.length < 4) {
                            setConMbtis([...conMbtis, mbti]);
                          }
                        } else {
                          setConMbtis(conMbtis.filter(m => m !== mbti));
                        }
                      }}
                    />
                    {mbti}
                  </label>
                ))}
              </div>
            </div>
          </>
        )}
        {sessionType === 'chat' && (
          <div className="mbti-selection">
            <label>选择闲聊对象：</label>
            <div className="mbti-options">
              {mbtiTypes.map(mbti => (
                <label key={mbti}>
                  <input
                    type="radio"
                    value={mbti}
                    checked={selectedMbtis.includes(mbti)}
                    onChange={(e) => setSelectedMbtis([e.target.value])}
                  />
                  {mbti}
                </label>
              ))}
            </div>
          </div>
        )}
        <div className="create-button">
          <button onClick={handleCreateSession}>创建会话</button>
        </div>
      </div>
    );
  };

  export default CreateSession;
  ```

### 3.3 Settings组件
- **功能**：配置LLM API
- **实现**：
  ```jsx
  import React, { useState, useEffect } from 'react';
  import { useDispatch, useSelector } from 'react-redux';
  import { updateSettings } from '../../store/slices/settingsSlice';

  const Settings = () => {
    const [apiKey, setApiKey] = useState('');
    const [model, setModel] = useState('gpt-3.5-turbo');
    const { settings } = useSelector(state => state.settings);
    const dispatch = useDispatch();

    useEffect(() => {
      if (settings) {
        setApiKey(settings.apiKey || '');
        setModel(settings.model || 'gpt-3.5-turbo');
      }
    }, [settings]);

    const handleSave = () => {
      dispatch(updateSettings({ apiKey, model }));
    };

    return (
      <div className="settings">
        <h2>设置</h2>
        <div className="setting-item">
          <label>API Key：</label>
          <input
            type="password"
            value={apiKey}
            onChange={(e) => setApiKey(e.target.value)}
            placeholder="输入LLM API Key"
          />
        </div>
        <div className="setting-item">
          <label>模型：</label>
          <select value={model} onChange={(e) => setModel(e.target.value)}>
            <option value="gpt-3.5-turbo">GPT-3.5 Turbo</option>
            <option value="gpt-4">GPT-4</option>
            <option value="deepseek-chat">DeepSeek Chat</option>
          </select>
        </div>
        <div className="save-button">
          <button onClick={handleSave}>保存设置</button>
        </div>
      </div>
    );
  };

  export default Settings;
  ```

## 4. IndexedDB实现

### 4.1 DBService封装
- **功能**：封装IndexedDB操作
- **实现**：
  ```javascript
  // services/db.js
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

          if (!db.objectStoreNames.contains('settings')) {
            db.createObjectStore('settings', { keyPath: 'id' });
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

### 4.2 数据操作示例
- **存储会话**：
  ```javascript
  const session = {
    id: Date.now(),
    user_id: 'current_user',
    type: 'brainstorm',
    topic: 'AI未来发展',
    status: 'created',
    created_at: new Date().toISOString(),
    updated_at: new Date().toISOString()
  };
  await dbService.add('sessions', session);
  ```

- **获取会话列表**：
  ```javascript
  const sessions = await dbService.getAll('sessions');
  ```

- **存储消息**：
  ```javascript
  const message = {
    id: Date.now(),
    session_id: 1,
    sender_type: 'host',
    sender_id: 'host',
    content: '欢迎大家参加头脑风暴！',
    timestamp: new Date().toISOString()
  };
  await dbService.add('messages', message);
  ```

- **获取会话消息**：
  ```javascript
  const messages = await dbService.getAll('messages', 'session_id', 1);
  ```

## 5. 状态管理

### 5.1 Redux Toolkit配置
- **store/index.js**：
  ```javascript
  import { configureStore } from '@reduxjs/toolkit';
  import sessionReducer from './slices/sessionSlice';
  import chatReducer from './slices/chatSlice';
  import settingsReducer from './slices/settingsSlice';

  export const store = configureStore({
    reducer: {
      session: sessionReducer,
      chat: chatReducer,
      settings: settingsReducer
    }
  });
  ```

### 5.2 Session Slice
- **store/slices/sessionSlice.js**：
  ```javascript
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

### 5.3 Chat Slice
- **store/slices/chatSlice.js**：
  ```javascript
  import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
  import { dbService } from '../../services/db';
  import { apiService } from '../../services/api';

  // 异步操作
export const sendMessage = createAsyncThunk(
  'chat/sendMessage',
  async ({ sessionId, content }) => {
    const message = {
      id: Date.now(),
      session_id: sessionId,
      sender_type: 'user',
      sender_id: 'current_user',
      content,
      timestamp: new Date().toISOString()
    };
    await dbService.add('messages', message);
    return message;
  }
);

export const startDiscussion = createAsyncThunk(
  'chat/startDiscussion',
  async (sessionId) => {
    // 更新会话状态
    const session = await dbService.get('sessions', sessionId);
    session.status = 'running';
    session.updated_at = new Date().toISOString();
    await dbService.update('sessions', session);
    
    // 调用后端API开始讨论
    await apiService.startDiscussion(sessionId);
    
    return session;
  }
);

export const pauseDiscussion = createAsyncThunk(
  'chat/pauseDiscussion',
  async (sessionId) => {
    // 更新会话状态
    const session = await dbService.get('sessions', sessionId);
    session.status = 'paused';
    session.updated_at = new Date().toISOString();
    await dbService.update('sessions', session);
    
    return session;
  }
);

export const getMessages = createAsyncThunk(
  'chat/getMessages',
  async (sessionId) => {
    return await dbService.getAll('messages', 'session_id', sessionId);
  }
);

// Slice
const chatSlice = createSlice({
  name: 'chat',
  initialState: {
    messages: [],
    session: null,
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(sendMessage.pending, (state) => {
        state.loading = true;
      })
      .addCase(sendMessage.fulfilled, (state, action) => {
        state.loading = false;
        state.messages.push(action.payload);
      })
      .addCase(sendMessage.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(startDiscussion.pending, (state) => {
        state.loading = true;
      })
      .addCase(startDiscussion.fulfilled, (state, action) => {
        state.loading = false;
        state.session = action.payload;
      })
      .addCase(startDiscussion.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(pauseDiscussion.pending, (state) => {
        state.loading = true;
      })
      .addCase(pauseDiscussion.fulfilled, (state, action) => {
        state.loading = false;
        state.session = action.payload;
      })
      .addCase(pauseDiscussion.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(getMessages.pending, (state) => {
        state.loading = true;
      })
      .addCase(getMessages.fulfilled, (state, action) => {
        state.loading = false;
        state.messages = action.payload;
      })
      .addCase(getMessages.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export default chatSlice.reducer;
  ```

### 5.4 Settings Slice
- **store/slices/settingsSlice.js**：
  ```javascript
  import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
  import { dbService } from '../../services/db';

  // 异步操作
export const updateSettings = createAsyncThunk(
  'settings/update',
  async (settingsData) => {
    const settings = {
      id: 'user_settings',
      ...settingsData
    };
    await dbService.update('settings', settings);
    return settings;
  }
);

export const getSettings = createAsyncThunk(
  'settings/get',
  async () => {
    return await dbService.get('settings', 'user_settings');
  }
);

// Slice
const settingsSlice = createSlice({
  name: 'settings',
  initialState: {
    settings: null,
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(updateSettings.pending, (state) => {
        state.loading = true;
      })
      .addCase(updateSettings.fulfilled, (state, action) => {
        state.loading = false;
        state.settings = action.payload;
      })
      .addCase(updateSettings.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(getSettings.pending, (state) => {
        state.loading = true;
      })
      .addCase(getSettings.fulfilled, (state, action) => {
        state.loading = false;
        state.settings = action.payload;
      })
      .addCase(getSettings.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export default settingsSlice.reducer;
  ```

## 6. API服务

### 6.1 Axios配置
- **services/api.js**：
  ```javascript
  import axios from 'axios';
  import { dbService } from './db';

  class ApiService {
    constructor() {
      this.baseURL = 'http://localhost:8000/api'; // 本地开发环境
      // this.baseURL = 'https://your-api-url.com/api'; // 生产环境
      this.client = axios.create({
        baseURL: this.baseURL,
        timeout: 30000,
        headers: {
          'Content-Type': 'application/json'
        }
      });
    }

    async getSettings() {
      return await dbService.get('settings', 'user_settings');
    }

    async startDiscussion(sessionId) {
      const settings = await this.getSettings();
      const response = await this.client.post('/discussions/start', {
        session_id: sessionId,
        api_key: settings?.apiKey
      });
      return response.data;
    }

    async sendMessage(sessionId, message) {
      const settings = await this.getSettings();
      const response = await this.client.post('/messages/send', {
        session_id: sessionId,
        message,
        api_key: settings?.apiKey
      });
      return response.data;
    }
  }

export const apiService = new ApiService();
  ```

## 7. 部署方案

### 7.1 本地运行

**1. 安装依赖**：
```bash
# 进入前端目录
cd frontend

# 安装依赖
npm install
```

**2. 启动开发服务器**：
```bash
npm run dev
```

**3. 访问应用**：
打开浏览器，访问 http://localhost:5173

### 7.2 云端部署

**1. 构建生产版本**：
```bash
# 进入前端目录
cd frontend

# 构建生产版本
npm run build
```

**2. 部署到Vercel**：
- 登录Vercel账号：https://vercel.com/login
- 点击"New Project"
- 选择前端代码仓库
- 配置构建命令：`npm run build`
- 配置输出目录：`dist`
- 点击"Deploy"
- 部署完成后获得访问URL

**3. 部署到Netlify**：
- 登录Netlify账号：https://app.netlify.com/
- 点击"Add new site" -> "Import an existing project"
- 选择前端代码仓库
- 配置构建命令：`npm run build`
- 配置发布目录：`dist`
- 点击"Deploy site"
- 部署完成后获得访问URL

**4. 部署到GitHub Pages**：
- 安装gh-pages包：
  ```bash
  npm install --save-dev gh-pages
  ```
- 在package.json中添加scripts：
  ```json
  "scripts": {
    "deploy": "gh-pages -d dist"
  }
  ```
- 构建并部署：
  ```bash
  npm run build
  npm run deploy
  ```
- 访问URL：https://your-username.github.io/your-repo-name

## 8. 傻瓜式部署步骤

### 8.1 准备工作
1. **安装Node.js**：访问 https://nodejs.org/ 下载并安装最新版本的Node.js
2. **安装Git**：访问 https://git-scm.com/ 下载并安装Git
3. **创建GitHub仓库**：登录GitHub，创建一个新的仓库

### 8.2 本地开发步骤
1. **克隆仓库**：
   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   cd your-repo-name
   ```

2. **创建前端目录**：
   ```bash
   mkdir frontend
   cd frontend
   ```

3. **初始化项目**：
   ```bash
   npm create vite@latest . -- --template react
   ```

4. **安装依赖**：
   ```bash
   npm install @reduxjs/toolkit react-redux antd styled-components axios
   ```

5. **创建文件结构**：
   - 按照项目结构创建相应的文件夹和文件
   - 实现核心组件和服务

6. **启动开发服务器**：
   ```bash
   npm run dev
   ```

7. **访问应用**：
   打开浏览器，访问 http://localhost:5173

### 8.3 云端部署步骤

**部署到Vercel**：
1. 登录Vercel账号：https://vercel.com/login
2. 点击"New Project"
3. 选择你的GitHub仓库
4. 配置构建命令：`npm run build`
5. 配置输出目录：`dist`
6. 点击"Deploy"
7. 部署完成后获得访问URL

**部署到Netlify**：
1. 登录Netlify账号：https://app.netlify.com/
2. 点击"Add new site" -> "Import an existing project"
3. 选择你的GitHub仓库
4. 配置构建命令：`npm run build`
5. 配置发布目录：`dist`
6. 点击"Deploy site"
7. 部署完成后获得访问URL

**部署到GitHub Pages**：
1. 安装gh-pages包：
   ```bash
   npm install --save-dev gh-pages
   ```
2. 在package.json中添加scripts：
   ```json
   "scripts": {
     "deploy": "gh-pages -d dist"
   }
   ```
3. 构建并部署：
   ```bash
   npm run build
   npm run deploy
   ```
4. 访问URL：https://your-username.github.io/your-repo-name

## 9. 故障排除

### 9.1 常见问题

**1. IndexedDB操作失败**：
- 检查浏览器是否支持IndexedDB
- 检查存储空间是否充足
- 检查数据结构是否正确

**2. API调用失败**：
- 检查后端服务是否运行
- 检查API URL是否正确
- 检查API Key是否有效

**3. 部署失败**：
- 检查构建命令是否正确
- 检查输出目录是否存在
- 检查网络连接是否正常

### 9.2 解决方法

**1. IndexedDB操作失败**：
- 清除浏览器缓存和数据
- 重启浏览器
- 检查代码中的IndexedDB操作逻辑

**2. API调用失败**：
- 检查后端服务日志
- 检查前端API配置
- 测试API调用是否正常

**3. 部署失败**：
- 查看部署日志
- 检查构建输出
- 确保所有依赖已正确安装

## 10. 总结

本前端实现文档详细描述了多Agents头脑风暴/辩论赛项目的前端实现细节，包括技术栈、项目结构、核心组件、IndexedDB实现、状态管理、API服务和部署方案。通过使用React、Redux Toolkit、Ant Design和IndexedDB，可以实现一个功能完整、用户友好的前端应用。

部署方案提供了本地运行和云端部署两种方式，用户可以根据自己的需求选择合适的部署方式。傻瓜式部署步骤详细说明了从准备工作到部署完成的整个过程，确保用户能够轻松部署和运行应用。

通过本文档的指导，用户可以快速搭建前端应用，实现多Agents头脑风暴/辩论赛的核心功能，为后续的开发和测试奠定基础。