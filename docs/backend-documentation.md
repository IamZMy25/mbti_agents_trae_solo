# 后端实现文档

## 1. 技术栈

### 1.1 核心技术
- **语言**：Python 3.11.14
- **Web框架**：FastAPI
- **LLM框架**：LangChain 1.2.10 + LangGraph
- **HTTP客户端**：httpx
- **部署**：Vercel Serverless Functions / Netlify Functions / AWS Lambda

### 1.2 依赖管理
```bash
# requirements.txt
fastapi==0.104.1
uvicorn==0.24.0
transformers==4.35.2
langchain==1.2.10
langchain-deepseek==1.0.1
langgraph==0.2.0
httpx==0.25.2
python-dotenv==1.0.0
```

## 2. 项目结构

```
backend/
├── app/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── discussions.py
│   │   └── messages.py
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py
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
│   │   └── llm_service.py
│   └── main.py
├── requirements.txt
└── vercel.json
```

## 3. 核心模块

### 3.1 配置管理
- **core/config.py**：
  ```python
  import os
  from dotenv import load_dotenv

  load_dotenv()

  class Config:
      API_KEY = os.getenv("API_KEY", "")
      MODEL_NAME = os.getenv("MODEL_NAME", "gpt-3.5-turbo")
      PORT = int(os.getenv("PORT", "8000"))
      HOST = os.getenv("HOST", "0.0.0.0")

  config = Config()
  ```

### 3.2 LLM服务
- **services/llm_service.py**：
  ```python
  from langchain.llms import OpenAI
  from langchain.prompts import PromptTemplate

  class LLMService:
      def __init__(self, api_key, model_name="gpt-3.5-turbo"):
          self.llm = OpenAI(api_key=api_key, model_name=model_name, temperature=0.7)

      def generate_response(self, prompt, context=None):
          if context:
              prompt = f"{context}\n{prompt}"
          return self.llm(prompt)
  ```

### 3.3 Agent实现

#### 3.3.1 Host Agent
- **agents/host_agent.py**：
  ```python
  from langchain.agents import AgentType, initialize_agent, Tool
  from langchain.llms import OpenAI

  class HostAgent:
      def __init__(self, api_key):
          self.llm = OpenAI(api_key=api_key, temperature=0.7)
          self.system_prompt = "You are a host agent for brainstorming and debate sessions. Your role is to facilitate discussions, summarize viewpoints, and guide the conversation towards productive outcomes."

      def generate_opening(self, topic):
          prompt = f"{self.system_prompt}\n\nPlease generate an opening statement for a brainstorming session on the topic: {topic}."
          return self.llm(prompt)

      def summarize_viewpoints(self, messages):
          context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
          prompt = f"{self.system_prompt}\n\nPlease summarize the key viewpoints from the following discussion:\n{context}"
          return self.llm(prompt)

      def guide_discussion(self, topic, viewpoints):
          prompt = f"{self.system_prompt}\n\nPlease provide guidance for continuing the discussion on {topic} based on the following viewpoints:\n{viewpoints}"
          return self.llm(prompt)

      def generate_summary(self, topic, messages):
          context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
          prompt = f"{self.system_prompt}\n\nPlease provide a comprehensive summary of the brainstorming session on {topic}, including key viewpoints and potential next steps:\n{context}"
          return self.llm(prompt)
  ```

#### 3.3.2 MBTI Agent
- **agents/mbti_agent.py**：
  ```python
  from langchain.llms import OpenAI

  class MBTIAgent:
      def __init__(self, mbti_type, api_key):
          self.mbti_type = mbti_type
          self.llm = OpenAI(api_key=api_key, temperature=0.7)
          self.system_prompt = self._get_mbti_prompt()

      def _get_mbti_prompt(self):
          mbti_prompts = {
              'INTJ': 'You are an INTJ personality type. You are strategic, analytical, and visionary. You approach problems with logic and reason, and you value efficiency and competence.',
              'INTP': 'You are an INTP personality type. You are logical, curious, and innovative. You enjoy exploring ideas and theories, and you value intellectual freedom.',
              'ENTJ': 'You are an ENTJ personality type. You are decisive, strategic, and leadership-oriented. You value efficiency, organization, and achievement.',
              'ENTP': 'You are an ENTP personality type. You are creative, curious, and debate-oriented. You enjoy challenging assumptions and exploring new possibilities.',
              'INFJ': 'You are an INFJ personality type. You are insightful, empathetic, and idealistic. You value deep connections and meaningful purpose.',
              'INFP': 'You are an INFP personality type. You are creative, idealistic, and values-oriented. You value authenticity and personal growth.',
              'ENFJ': 'You are an ENFJ personality type. You are charismatic, empathetic, and leadership-oriented. You value harmony and personal growth in others.',
              'ENFP': 'You are an ENFP personality type. You are enthusiastic, creative, and people-oriented. You value personal growth and meaningful experiences.',
              'ISTJ': 'You are an ISTJ personality type. You are practical, responsible, and detail-oriented. You value tradition and stability.',
              'ISFJ': 'You are an ISFJ personality type. You are compassionate, responsible, and detail-oriented. You value harmony and service to others.',
              'ESTJ': 'You are an ESTJ personality type. You are practical, organized, and leadership-oriented. You value tradition and order.',
              'ESFJ': 'You are an ESFJ personality type. You are compassionate, organized, and people-oriented. You value harmony and service to others.',
              'ISTP': 'You are an ISTP personality type. You are practical, analytical, and hands-on. You value freedom and immediate experience.',
              'ISFP': 'You are an ISFP personality type. You are creative, adaptable, and sensory-oriented. You value personal freedom and aesthetic experiences.',
              'ESTP': 'You are an ESTP personality type. You are energetic, practical, and action-oriented. You value immediate experience and freedom.',
              'ESFP': 'You are an ESFP personality type. You are energetic, sociable, and sensory-oriented. You value immediate experience and harmony.'
          }
          return mbti_prompts.get(self.mbti_type, 'You are a helpful assistant.')

      def generate观点(self, context):
          prompt = f"{self.system_prompt}\n\n{context}\n\nPlease provide your perspective on this topic based on your personality type."
          return self.llm(prompt)

      def participate_in_discussion(self, context):
          prompt = f"{self.system_prompt}\n\n{context}\n\nPlease participate in the discussion based on your personality type."
          return self.llm(prompt)

      def vote(self, viewpoints):
          prompt = f"{self.system_prompt}\n\nPlease vote on the following viewpoints by selecting the one you most agree with:\n{viewpoints}\n\nProvide only the number of the viewpoint you select."
          return self.llm(prompt)
  ```

## 4. LangGraph实现

### 4.1 头脑风暴Graph
- **graphs/brainstorm_graph.py**：
  ```python
  from langgraph.graph import Graph
  from agents.host_agent import HostAgent
  from agents.mbti_agent import MBTIAgent

  class BrainstormGraph:
    def __init__(self, session_id, topic, mbti_types, api_key):
      self.session_id = session_id
      self.topic = topic
      self.mbti_types = mbti_types
      self.api_key = api_key
      self.host_agent = HostAgent(api_key)
      self.mbti_agents = {mbti: MBTIAgent(mbti, api_key) for mbti in mbti_types}
      self.graph = self.build_graph()

    def build_graph(self):
      graph = Graph()

      # 开场白节点
      def opening_node(state):
        opening_message = self.host_agent.generate_opening(self.topic)
        return {**state, "messages": [{"sender_type": "host", "sender_id": "host", "content": opening_message, "timestamp": "now"}]}

      # 第一轮观点表达节点
      async def first_round_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.mbti_agents.items():
          tasks.append(agent.generate观点(context))
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.mbti_types, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 第一轮讨论节点
      async def discussion_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.mbti_agents.items():
          tasks.append(agent.participate_in_discussion(context))
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.mbti_types, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        # Host Agent总结观点
        summary = self.host_agent.summarize_viewpoints(new_messages)
        new_messages.append({"sender_type": "host", "sender_id": "host", "content": summary, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 观点投票节点
      async def voting_node(state):
        messages = state["messages"]
        # 提取观点
        viewpoints = [msg["content"] for msg in messages if msg["sender_id"] == "host" and "summary" in msg["content"].lower()]
        if not viewpoints:
          viewpoints = [msg["content"] for msg in messages if msg["sender_type"] == "mbti"]
        
        # 格式化观点
        formatted_viewpoints = "\n".join([f"{i+1}. {viewpoint}" for i, viewpoint in enumerate(viewpoints)])
        
        # 并发调用MBTI Agent投票
        import asyncio
        tasks = []
        for mbti, agent in self.mbti_agents.items():
          tasks.append(agent.vote(formatted_viewpoints))
        
        results = await asyncio.gather(*tasks)
        
        # 计算投票结果
        votes = {}
        for mbti, result in zip(self.mbti_types, results):
          try:
            vote = int(result.strip()) - 1
            if 0 <= vote < len(viewpoints):
              votes[mbti] = vote
          except:
            pass
        
        # 选择得票最高的观点
        vote_counts = {i: 0 for i in range(len(viewpoints))}
        for vote in votes.values():
          vote_counts[vote] += 1
        selected_viewpoint = max(vote_counts, key=vote_counts.get)
        
        # Host Agent引导后续讨论
        guidance = self.host_agent.guide_discussion(self.topic, viewpoints[selected_viewpoint])
        new_messages = messages.copy()
        new_messages.append({"sender_type": "host", "sender_id": "host", "content": guidance, "timestamp": "now"})
        
        return {**state, "messages": new_messages, "selected_viewpoint": selected_viewpoint}

      # 第二轮讨论节点
      async def second_discussion_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.mbti_agents.items():
          tasks.append(agent.participate_in_discussion(context))
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.mbti_types, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 总结节点
      def summary_node(state):
        messages = state["messages"]
        summary = self.host_agent.generate_summary(self.topic, messages)
        new_messages = messages.copy()
        new_messages.append({"sender_type": "host", "sender_id": "host", "content": summary, "timestamp": "now"})
        return {**state, "messages": new_messages}

      # 添加节点到图
      graph.add_node("opening", opening_node)
      graph.add_node("first_round", first_round_node)
      graph.add_node("discussion", discussion_node)
      graph.add_node("voting", voting_node)
      graph.add_node("second_discussion", second_discussion_node)
      graph.add_node("summary", summary_node)

      # 设置边
      graph.set_entry_point("opening")
      graph.add_edge("opening", "first_round")
      graph.add_edge("first_round", "discussion")
      graph.add_edge("discussion", "voting")
      graph.add_edge("voting", "second_discussion")
      graph.add_edge("second_discussion", "summary")

      return graph

    async def run(self):
      initial_state = {"messages": []}
      return await self.graph.run(initial_state)
  ```

### 4.2 辩论赛Graph
- **graphs/debate_graph.py**：
  ```python
  from langgraph.graph import Graph
  from agents.host_agent import HostAgent
  from agents.mbti_agent import MBTIAgent

  class DebateGraph:
    def __init__(self, session_id, topic, pro_mbtis, con_mbtis, api_key):
      self.session_id = session_id
      self.topic = topic
      self.pro_mbtis = pro_mbtis
      self.con_mbtis = con_mbtis
      self.api_key = api_key
      self.host_agent = HostAgent(api_key)
      self.pro_agents = {mbti: MBTIAgent(mbti, api_key) for mbti in pro_mbtis}
      self.con_agents = {mbti: MBTIAgent(mbti, api_key) for mbti in con_mbtis}
      self.graph = self.build_graph()

    def build_graph(self):
      graph = Graph()

      # 开场节点
      def opening_node(state):
        opening_message = f"Welcome to the debate on: {self.topic}\n\nPro side: For the topic\nCon side: Against the topic\n\nLet's begin with opening statements."
        return {**state, "messages": [{"sender_type": "host", "sender_id": "host", "content": opening_message, "timestamp": "now"}]}

      # 正方立论节点
      async def pro_opening_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用正方MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.pro_agents.items():
          tasks.append(agent.generate观点(f"You are on the pro side of the debate on {self.topic}. Please provide your opening statement.") )
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.pro_mbtis, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 反方立论节点
      async def con_opening_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用反方MBTI Agent
        import asyncio
        tasks = []
        for mbti, agent in self.con_agents.items():
          tasks.append(agent.generate观点(f"You are on the con side of the debate on {self.topic}. Please provide your opening statement.") )
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(self.con_mbtis, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 自由辩论节点
      async def free_debate_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用所有MBTI Agent
        import asyncio
        tasks = []
        all_agents = {**self.pro_agents, **self.con_agents}
        all_mbtis = self.pro_mbtis + self.con_mbtis
        
        for mbti, agent in all_agents.items():
          tasks.append(agent.participate_in_discussion(context))
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(all_mbtis, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 总结陈词节点
      async def conclusion_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 并发调用所有MBTI Agent
        import asyncio
        tasks = []
        all_agents = {**self.pro_agents, **self.con_agents}
        all_mbtis = self.pro_mbtis + self.con_mbtis
        
        for mbti, agent in all_agents.items():
          tasks.append(agent.generate观点(f"Please provide your closing statement for the debate on {self.topic}.") )
        
        results = await asyncio.gather(*tasks)
        
        # 添加到消息列表
        new_messages = messages.copy()
        for mbti, result in zip(all_mbtis, results):
          new_messages.append({"sender_type": "mbti", "sender_id": mbti, "content": result, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 结果宣布节点
      def result_node(state):
        messages = state["messages"]
        result = "The debate has concluded. Both sides presented strong arguments. Thank you for participating!"
        new_messages = messages.copy()
        new_messages.append({"sender_type": "host", "sender_id": "host", "content": result, "timestamp": "now"})
        return {**state, "messages": new_messages}

      # 添加节点到图
      graph.add_node("opening", opening_node)
      graph.add_node("pro_opening", pro_opening_node)
      graph.add_node("con_opening", con_opening_node)
      graph.add_node("free_debate", free_debate_node)
      graph.add_node("conclusion", conclusion_node)
      graph.add_node("result", result_node)

      # 设置边
      graph.set_entry_point("opening")
      graph.add_edge("opening", "pro_opening")
      graph.add_edge("pro_opening", "con_opening")
      graph.add_edge("con_opening", "free_debate")
      graph.add_edge("free_debate", "conclusion")
      graph.add_edge("conclusion", "result")

      return graph

    async def run(self):
      initial_state = {"messages": []}
      return await self.graph.run(initial_state)
  ```

### 4.3 闲聊Graph
- **graphs/chat_graph.py**：
  ```python
  from langgraph.graph import Graph
  from agents.mbti_agent import MBTIAgent

  class ChatGraph:
    def __init__(self, session_id, mbti_type, api_key):
      self.session_id = session_id
      self.mbti_type = mbti_type
      self.api_key = api_key
      self.mbti_agent = MBTIAgent(mbti_type, api_key)
      self.graph = self.build_graph()

    def build_graph(self):
      graph = Graph()

      # 聊天节点
      async def chat_node(state):
        messages = state["messages"]
        context = "\n".join([f"{msg['sender_id']}: {msg['content']}" for msg in messages])
        
        # 调用MBTI Agent
        response = self.mbti_agent.participate_in_discussion(context)
        
        # 添加到消息列表
        new_messages = messages.copy()
        new_messages.append({"sender_type": "mbti", "sender_id": self.mbti_type, "content": response, "timestamp": "now"})
        
        return {**state, "messages": new_messages}

      # 添加节点到图
      graph.add_node("chat", chat_node)

      # 设置边
      graph.set_entry_point("chat")
      graph.add_edge("chat", "chat")  # 循环调用

      return graph

    async def run(self, initial_message):
      initial_state = {"messages": [{"sender_type": "user", "sender_id": "user", "content": initial_message, "timestamp": "now"}]}
      return await self.graph.run(initial_state, {"max_steps": 1})
  ```

## 5. API设计

### 5.1 FastAPI配置
- **app/main.py**：
  ```python
  from fastapi import FastAPI
  from fastapi.middleware.cors import CORSMiddleware
  from app.api import discussions, messages
  from app.core.config import config

  app = FastAPI(
      title="Agent Brainstorm API",
      description="API for Agent Brainstorm and Debate sessions",
      version="1.0.0"
  )

  # 配置CORS
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["*"],
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
  )

  # 注册路由
  app.include_router(discussions.router, prefix="/api/discussions", tags=["discussions"])
  app.include_router(messages.router, prefix="/api/messages", tags=["messages"])

  @app.get("/")
async def root():
    return {"message": "Agent Brainstorm API"}

  if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host=config.HOST, port=config.PORT)
  ```

### 5.2 讨论API
- **app/api/discussions.py**：
  ```python
  from fastapi import APIRouter, HTTPException
  from pydantic import BaseModel
  from app.graphs.brainstorm_graph import BrainstormGraph
  from app.graphs.debate_graph import DebateGraph

  router = APIRouter()

  class StartDiscussionRequest(BaseModel):
    session_id: int
    api_key: str
    session_type: str = "brainstorm"
    topic: str = ""
    mbti_types: list[str] = []
    pro_mbtis: list[str] = []
    con_mbtis: list[str] = []

  class DiscussionResponse(BaseModel):
    messages: list

  @router.post("/start", response_model=DiscussionResponse)
async def start_discussion(request: StartDiscussionRequest):
    try:
      if request.session_type == "brainstorm":
        graph = BrainstormGraph(
          request.session_id,
          request.topic,
          request.mbti_types,
          request.api_key
        )
      elif request.session_type == "debate":
        graph = DebateGraph(
          request.session_id,
          request.topic,
          request.pro_mbtis,
          request.con_mbtis,
          request.api_key
        )
      else:
        raise HTTPException(status_code=400, detail="Invalid session type")

      result = await graph.run()
      return DiscussionResponse(messages=result["messages"])
    except Exception as e:
      raise HTTPException(status_code=500, detail=str(e))
  ```

### 5.3 消息API
- **app/api/messages.py**：
  ```python
  from fastapi import APIRouter, HTTPException
  from pydantic import BaseModel
  from app.graphs.chat_graph import ChatGraph

  router = APIRouter()

  class SendMessageRequest(BaseModel):
    session_id: int
    message: str
    api_key: str
    mbti_type: str = ""

  class MessageResponse(BaseModel):
    messages: list

  @router.post("/send", response_model=MessageResponse)
async def send_message(request: SendMessageRequest):
    try:
      graph = ChatGraph(
        request.session_id,
        request.mbti_type,
        request.api_key
      )

      result = await graph.run(request.message)
      return MessageResponse(messages=result["messages"])
    except Exception as e:
      raise HTTPException(status_code=500, detail=str(e))
  ```

## 6. 部署方案

### 6.1 本地运行

**1. 安装依赖**：
```bash
# 进入后端目录
cd backend

# 安装依赖
pip install -r requirements.txt
```

**2. 配置环境变量**：
```bash
# 创建.env文件
cat > .env << EOF
API_KEY=your_api_key
MODEL_NAME=gpt-3.5-turbo
PORT=8000
HOST=0.0.0.0
EOF
```

**3. 启动开发服务器**：
```bash
uvicorn app.main:app --reload
```

**4. 访问API文档**：
打开浏览器，访问 http://localhost:8000/docs

### 6.2 云端部署

#### 6.2.1 部署到Vercel

**1. 配置vercel.json**：
```json
{
  "builds": [
    {
      "src": "app/main.py",
      "use": "@vercel/python"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "app/main.py"
    }
  ]
}
```

**2. 部署步骤**：
- 登录Vercel账号：https://vercel.com/login
- 点击"New Project"
- 选择后端代码仓库
- 配置环境变量：
  - API_KEY: your_api_key
  - MODEL_NAME: gpt-3.5-turbo
- 点击"Deploy"
- 部署完成后获得访问URL

#### 6.2.2 部署到Netlify

**1. 配置netlify.toml**：
```toml
[build]
  command = "echo 'Build completed'"
  publish = "."

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/api"
  status = 200
```

**2. 创建Netlify Function**：
```python
# netlify/functions/api.py
from app.main import app
from mangum import Mangum

handler = Mangum(app)
```

**3. 部署步骤**：
- 登录Netlify账号：https://app.netlify.com/
- 点击"Add new site" -> "Import an existing project"
- 选择后端代码仓库
- 配置环境变量：
  - API_KEY: your_api_key
  - MODEL_NAME: gpt-3.5-turbo
- 点击"Deploy site"
- 部署完成后获得访问URL

#### 6.2.3 部署到AWS Lambda

**1. 安装AWS CLI和SAM CLI**：
```bash
# 安装AWS CLI
pip install awscli

# 安装SAM CLI
pip install aws-sam-cli
```

**2. 配置template.yaml**：
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Agent Brainstorm API

Resources:
  AgentBrainstormAPI:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Handler: app.main.app
      Runtime: python3.11
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /{proxy+}
            Method: ANY
      Environment:
        Variables:
          API_KEY: your_api_key
          MODEL_NAME: gpt-3.5-turbo

Outputs:
  AgentBrainstormAPI:
    Description: API Gateway endpoint URL
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
```

**3. 部署步骤**：
```bash
# 构建
sam build

# 部署
sam deploy --guided
```

## 7. 傻瓜式部署步骤

### 7.1 准备工作
1. **安装Python**：访问 https://www.python.org/ 下载并安装最新版本的Python
2. **安装Git**：访问 https://git-scm.com/ 下载并安装Git
3. **创建GitHub仓库**：登录GitHub，创建一个新的仓库

### 7.2 本地开发步骤
1. **克隆仓库**：
   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   cd your-repo-name
   ```

2. **创建后端目录**：
   ```bash
   mkdir backend
   cd backend
   ```

3. **创建requirements.txt**：
   ```bash
   cat > requirements.txt << EOF
   fastapi==0.104.1
   uvicorn==0.24.0
   transformers==4.35.2
   langchain==1.2.10
   langchain-deepseek==1.0.1
   langgraph==0.2.0
   httpx==0.25.2
   python-dotenv==1.0.0
   EOF
   ```

4. **创建项目结构**：
   - 按照项目结构创建相应的文件夹和文件
   - 实现核心模块和API

5. **配置环境变量**：
   ```bash
   cat > .env << EOF
   API_KEY=your_api_key
   MODEL_NAME=gpt-3.5-turbo
   PORT=8000
   HOST=0.0.0.0
   EOF
   ```

6. **安装依赖**：
   ```bash
   pip install -r requirements.txt
   ```

7. **启动开发服务器**：
   ```bash
   uvicorn app.main:app --reload
   ```

8. **访问API文档**：
   打开浏览器，访问 http://localhost:8000/docs

### 7.3 云端部署步骤

**部署到Vercel**：
1. 登录Vercel账号：https://vercel.com/login
2. 点击"New Project"
3. 选择你的GitHub仓库
4. 配置环境变量：
   - API_KEY: your_api_key
   - MODEL_NAME: gpt-3.5-turbo
5. 点击"Deploy"
6. 部署完成后获得访问URL

**部署到Netlify**：
1. 登录Netlify账号：https://app.netlify.com/
2. 点击"Add new site" -> "Import an existing project"
3. 选择你的GitHub仓库
4. 配置环境变量：
   - API_KEY: your_api_key
   - MODEL_NAME: gpt-3.5-turbo
5. 点击"Deploy site"
6. 部署完成后获得访问URL

**部署到AWS Lambda**：
1. 安装AWS CLI和SAM CLI
2. 创建template.yaml文件
3. 执行sam build和sam deploy命令
4. 部署完成后获得访问URL

## 8. 故障排除

### 8.1 常见问题

**1. API调用失败**：
- 检查API Key是否有效
- 检查网络连接是否正常
- 检查LLM服务是否可用

**2. 部署失败**：
- 检查依赖是否正确安装
- 检查环境变量是否配置正确
- 检查代码是否有语法错误

**3. 性能问题**：
- 检查LLM调用频率是否过高
- 检查并发请求是否过多
- 优化代码，减少不必要的API调用

### 8.2 解决方法

**1. API调用失败**：
- 验证API Key是否正确
- 检查网络连接，确保能够访问LLM服务
- 查看LLM服务的状态和限制

**2. 部署失败**：
- 查看部署日志，了解具体错误信息
- 确保所有依赖都已正确安装
- 检查环境变量配置是否正确

**3. 性能问题**：
- 实现请求限流，避免过多并发请求
- 缓存LLM响应，减少重复调用
- 优化代码，提高执行效率

## 9. 总结

本后端实现文档详细描述了多Agents头脑风暴/辩论赛项目的后端实现细节，包括技术栈、项目结构、核心模块、API设计、Agent实现、LangGraph实现和部署方案。通过使用FastAPI、LangChain和LangGraph，可以实现一个功能完整、性能良好的后端服务。

部署方案提供了本地运行和云端部署两种方式，用户可以根据自己的需求选择合适的部署方式。傻瓜式部署步骤详细说明了从准备工作到部署完成的整个过程，确保用户能够轻松部署和运行后端服务。

通过本文档的指导，用户可以快速搭建后端服务，实现多Agents头脑风暴/辩论赛的核心功能，为后续的开发和测试奠定基础。