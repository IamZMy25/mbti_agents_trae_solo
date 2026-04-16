# 技术实现文档

## 1. 技术栈选择

### 1.1 前端技术栈
- **框架**：React
- **状态管理**：Redux Toolkit
- **UI库**：Ant Design
- **通信**：WebSocket + Axios
- **样式**：CSS-in-JS (styled-components)
- **构建工具**：Vite

### 1.2 后端技术栈
- **语言**：Python 3.11.14
- **Web框架**：FastAPI
- **LLM框架**：LangChain 1.2.10 + LangGraph
- **数据库**：PostgreSQL 15-alpine
- **实时通信**：WebSocket
- **部署**：Docker + Docker Compose

## 2. 前端实现

### 2.1 项目结构
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
│   │   └── websocket.js
│   ├── store/
│   │   ├── slices/
│   │   └── index.js
│   ├── utils/
│   ├── App.js
│   └── main.js
├── package.json
└── vite.config.js
```

### 2.2 核心组件

#### 2.2.1 ChatRoom组件
- **功能**：显示聊天消息，发送消息，控制讨论
- **实现**：
  ```jsx
  import React, { useState, useEffect, useRef } from 'react';
  import { useSelector, useDispatch } from 'react-redux';
  import { sendMessage, startDiscussion, pauseDiscussion } from '../../store/slices/chatSlice';
  import { WebSocketService } from '../../services/websocket';

  const ChatRoom = ({ sessionId }) => {
    const [message, setMessage] = useState('');
    const messagesEndRef = useRef(null);
    const { messages, session } = useSelector(state => state.chat);
    const dispatch = useDispatch();

    useEffect(() => {
      WebSocketService.connect(sessionId);
      WebSocketService.onMessage((data) => {
        // 处理新消息
      });
      WebSocketService.onStatus((data) => {
        // 处理状态变更
      });

      return () => {
        WebSocketService.disconnect();
      };
    }, [sessionId]);

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

#### 2.2.2 CreateSession组件
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

### 2.3 状态管理
- **Redux Toolkit**：管理全局状态
- **切片**：
  - userSlice：用户信息
  - sessionSlice：会话管理
  - chatSlice：聊天消息

### 2.4 WebSocket实现
- **WebSocketService**：处理实时通信
  ```javascript
  class WebSocketService {
    constructor() {
      this.socket = null;
      this.messageHandlers = [];
      this.statusHandlers = [];
    }

    connect(sessionId) {
      this.socket = new WebSocket(`ws://localhost:8000/ws/sessions/${sessionId}`);

      this.socket.onopen = () => {
        console.log('WebSocket connected');
      };

      this.socket.onmessage = (event) => {
        const data = JSON.parse(event.data);
        if (data.type === 'message') {
          this.messageHandlers.forEach(handler => handler(data.payload));
        } else if (data.type === 'status') {
          this.statusHandlers.forEach(handler => handler(data.payload));
        }
      };

      this.socket.onclose = () => {
        console.log('WebSocket disconnected');
      };

      this.socket.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
    }

    disconnect() {
      if (this.socket) {
        this.socket.close();
        this.socket = null;
      }
    }

    onMessage(handler) {
      this.messageHandlers.push(handler);
    }

    onStatus(handler) {
      this.statusHandlers.push(handler);
    }

    sendMessage(message) {
      if (this.socket && this.socket.readyState === WebSocket.OPEN) {
        this.socket.send(JSON.stringify(message));
      }
    }
  }

  export const WebSocketService = new WebSocketService();
  ```

## 3. 后端实现

### 3.1 项目结构
```
backend/
├── app/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── sessions.py
│   │   └── websocket.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── database.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── session.py
│   │   ├── message.py
│   │   ├── viewpoint.py
│   │   └── vote.py
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── host_agent.py
│   │   └── mbti_agent.py
│   ├── graphs/
│   │   ├── __init__.py
│   │   ├── brainstorm_graph.py
│   │   ├── debate_graph.py
│   │   └── chat_graph.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── llm_service.py
│   │   └── session_service.py
│   └── main.py
├── requirements.txt
└── docker-compose.yml
```

### 3.2 核心实现

#### 3.2.1 LangGraph实现
- **头脑风暴Graph**：
  ```python
  from langgraph.graph import Graph
  from agents.host_agent import HostAgent
  from agents.mbti_agent import MBTIAgent

  class BrainstormGraph:
    def __init__(self, session_id, topic, mbti_types):
      self.session_id = session_id
      self.topic = topic
      self.mbti_types = mbti_types
      self.host_agent = HostAgent()
      self.mbti_agents = {mbti: MBTIAgent(mbti) for mbti in mbti_types}
      self.graph = self.build_graph()

    def build_graph(self):
      graph = Graph()

      # 开场白节点
      def opening_node(state):
        opening_message = self.host_agent.generate_opening(self.topic)
        # 保存消息到数据库
        # 发送消息到前端
        return {**state, "messages": [opening_message]}

      # 第一轮观点表达节点
      async def first_round_node(state):
        messages = state["messages"]
        opening_message = messages[0]
        
        # 并发调用MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.mbti_agents.items():
          tasks.append(agent.generate观点(opening_message))
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.mbti_types, results):
          new_messages.append({
            "sender_type": "mbti",
            "sender_id": mbti,
            "content": result
          })
        
        return {**state, "messages": new_messages}

      # 其他节点实现...

      # 添加节点到图
      graph.add_node("opening", opening_node)
      graph.add_node("first_round", first_round_node)
      # 添加其他节点...

      # 设置边
      graph.set_entry_point("opening")
      graph.add_edge("opening", "first_round")
      # 添加其他边...

      return graph

    def run(self):
      initial_state = {"messages": []}
      return self.graph.run(initial_state)
  ```

#### 3.2.2 WebSocket实现
- **WebSocket处理**：
  ```python
  from fastapi import APIRouter, WebSocket, WebSocketDisconnect
  from app.services.session_service import SessionService

  router = APIRouter()
  session_service = SessionService()

  @router.websocket("/ws/sessions/{session_id}")
async def websocket_endpoint(websocket: WebSocket, session_id: int):
    await websocket.accept()
    session_service.add_websocket(session_id, websocket)
    
    try:
      while True:
        data = await websocket.receive_json()
        # 处理接收到的消息
        await session_service.handle_message(session_id, data)
    except WebSocketDisconnect:
      session_service.remove_websocket(session_id, websocket)
  ```

#### 3.2.3 数据库实现
- **SQLAlchemy模型**：
  ```python
  from sqlalchemy import Column, Integer, String, Text, ForeignKey, TIMESTAMP
  from sqlalchemy.sql import func
  from app.core.database import Base

  class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(255), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    created_at = Column(TIMESTAMP, server_default=func.now())

  class Session(Base):
    __tablename__ = "sessions"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    type = Column(String(50), nullable=False)
    topic = Column(String(255), nullable=False)
    status = Column(String(50), nullable=False)
    created_at = Column(TIMESTAMP, server_default=func.now())
    updated_at = Column(TIMESTAMP, server_default=func.now(), onupdate=func.now())
  ```

### 3.3 API实现
- **FastAPI路由**：
  ```python
  from fastapi import APIRouter, Depends, HTTPException
  from sqlalchemy.orm import Session
  from app.core.database import get_db
  from app.models.session import Session
  from app.schemas.session import SessionCreate, SessionResponse

  router = APIRouter()

  @router.post("/sessions", response_model=SessionResponse)
async def create_session(session: SessionCreate, db: Session = Depends(get_db)):
    # 创建会话逻辑
    pass

  @router.get("/sessions", response_model=list[SessionResponse])
async def get_sessions(db: Session = Depends(get_db)):
    # 获取会话列表逻辑
    pass

  @router.put("/sessions/{session_id}/start")
async def start_session(session_id: int, db: Session = Depends(get_db)):
    # 开始会话逻辑
    pass

  @router.put("/sessions/{session_id}/pause")
async def pause_session(session_id: int, db: Session = Depends(get_db)):
    # 暂停会话逻辑
    pass
  ```

## 4. 部署方案

### 4.1 Docker Compose配置
- **docker-compose.yml**：
  ```yaml
  version: '3.8'
  
  services:
    db:
      image: postgres:15-alpine
      environment:
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: postgres
        POSTGRES_DB: agent_brainstorm
      ports:
        - "5432:5432"
      volumes:
        - postgres_data:/var/lib/postgresql/data
    
    backend:
      build: ./backend
      ports:
        - "8000:8000"
      depends_on:
        - db
      environment:
        DATABASE_URL: postgresql://postgres:postgres@db:5432/agent_brainstorm
        API_KEY: your_api_key
    
    frontend:
      build: ./frontend
      ports:
        - "3000:3000"
      depends_on:
        - backend
      environment:
        REACT_APP_API_URL: http://localhost:8000
  
  volumes:
    postgres_data:
  ```

### 4.2 构建和运行
1. **构建镜像**：
   ```bash
   docker-compose build
   ```

2. **启动服务**：
   ```bash
   docker-compose up -d
   ```

3. **访问应用**：
   - 前端：http://localhost:3000
   - 后端API：http://localhost:8000/docs

## 5. 代码示例

### 5.1 前端API调用
- **创建会话**：
  ```javascript
  import axios from 'axios';

  const createSession = async (sessionData) => {
    try {
      const response = await axios.post('http://localhost:8000/api/sessions', sessionData);
      return response.data;
    } catch (error) {
      console.error('Error creating session:', error);
      throw error;
    }
  };
  ```

### 5.2 后端LLM调用
- **LLM服务**：
  ```python
  from langchain.llms import OpenAI
  from langchain.prompts import PromptTemplate

  class LLMService:
    def __init__(self, api_key):
      self.llm = OpenAI(api_key=api_key, temperature=0.7)

    def generate_response(self, prompt, context=None):
      if context:
        prompt = f"{context}\n{prompt}"
      return self.llm(prompt)
  ```

### 5.3 MBTI Agent实现
- **MBTI Agent**：
  ```python
  from langchain.agents import AgentType, initialize_agent, Tool
  from langchain.llms import OpenAI

  class MBTIAgent:
    def __init__(self, mbti_type):
      self.mbti_type = mbti_type
      self.llm = OpenAI(temperature=0.7)
      self.system_prompt = self._get_mbti_prompt()

    def _get_mbti_prompt(self):
      mbti_prompts = {
        'INTJ': 'You are an INTJ personality type. You are strategic, analytical, and visionary. You approach problems with logic and reason, and you value efficiency and competence.',
        # 其他MBTI类型的提示词...
      }
      return mbti_prompts.get(self.mbti_type, 'You are a helpful assistant.')

    def generate观点(self, context):
      prompt = f"{self.system_prompt}\n\n{context}\n\nPlease provide your perspective on this topic."
      return self.llm(prompt)
  ```

## 6. 总结

本技术实现文档详细描述了多Agents头脑风暴/辩论赛项目的技术栈选择、前端实现、后端实现、部署方案和代码示例。通过使用React、FastAPI、LangChain和PostgreSQL等技术，实现了一个功能完整、性能良好的多Agents交互系统。

该系统支持三种会话模式：头脑风暴、辩论赛和闲聊，每种模式都有独特的交互流程和智能体行为。通过WebSocket实现实时通信，确保了讨论的流畅性和实时性。同时，通过PostgreSQL数据库存储会话信息、消息记录、观点和投票，确保了数据的持久化和可靠性。

部署方案使用Docker Compose，简化了环境配置和部署流程，使系统能够快速部署到生产环境。代码示例提供了关键功能的实现细节，帮助开发人员理解和实现系统的各个组件。