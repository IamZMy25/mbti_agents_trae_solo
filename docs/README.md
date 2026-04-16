# 多Agents头脑风暴/辩论赛项目

## 项目概述

本项目旨在创建一个多Agents进行头脑风暴或辩论赛的系统，每个Agent都具有不同的MBTI人格特征。系统支持三种会话模式：头脑风暴、辩论赛和闲聊，用户可以通过前端界面与这些智能体进行交互。

### 核心功能
- **多MBTI人格智能体**：每个智能体都具有独特的MBTI人格特征
- **三种会话模式**：头脑风暴、辩论赛和闲聊
- **实时交互**：通过前端状态管理实现实时更新
- **数据持久化**：使用IndexedDB存储会话信息、消息记录、观点和投票
- **可配置性**：用户可以配置LLM API和参与的智能体

### 技术栈
- **前端**：React + Redux Toolkit + Ant Design + IndexedDB
- **后端**：Python 3.11.14 + FastAPI + LangChain 1.2.10 + LangGraph
- **部署**：静态网站（Vercel/Netlify/GitHub Pages） + 云端LLM API

## 文档结构

- [技术需求文档](technical-requirements.md)：详细描述项目的技术需求、系统架构和数据存储设计
- [前端实现文档](frontend-documentation.md)：详细描述前端的实现细节、组件设计和部署方案
- [后端实现文档](backend-documentation.md)：详细描述后端的实现细节、API设计和部署方案

## 快速开始

### 前端开发

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
   - 按照前端文档中的项目结构创建相应的文件夹和文件
   - 实现核心组件和服务

6. **启动开发服务器**：
   ```bash
   npm run dev
   ```

7. **访问应用**：
   打开浏览器，访问 http://localhost:5173

### 后端开发

1. **创建后端目录**：
   ```bash
   mkdir backend
   cd backend
   ```

2. **创建requirements.txt**：
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

3. **创建项目结构**：
   - 按照后端文档中的项目结构创建相应的文件夹和文件
   - 实现核心模块和API

4. **配置环境变量**：
   ```bash
   cat > .env << EOF
   API_KEY=your_api_key
   MODEL_NAME=gpt-3.5-turbo
   PORT=8000
   HOST=0.0.0.0
   EOF
   ```

5. **安装依赖**：
   ```bash
   pip install -r requirements.txt
   ```

6. **启动开发服务器**：
   ```bash
   uvicorn app.main:app --reload
   ```

7. **访问API文档**：
   打开浏览器，访问 http://localhost:8000/docs

## 部署方案

### 前端部署

**部署到Vercel**：
1. 登录Vercel账号：https://vercel.com/login
2. 点击"New Project"
3. 选择前端代码仓库
4. 配置构建命令：`npm run build`
5. 配置输出目录：`dist`
6. 点击"Deploy"
7. 部署完成后获得访问URL

**部署到Netlify**：
1. 登录Netlify账号：https://app.netlify.com/
2. 点击"Add new site" -> "Import an existing project"
3. 选择前端代码仓库
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

### 后端部署

**部署到Vercel**：
1. 登录Vercel账号：https://vercel.com/login
2. 点击"New Project"
3. 选择后端代码仓库
4. 配置环境变量：
   - API_KEY: your_api_key
   - MODEL_NAME: gpt-3.5-turbo
5. 点击"Deploy"
6. 部署完成后获得访问URL

**部署到Netlify**：
1. 登录Netlify账号：https://app.netlify.com/
2. 点击"Add new site" -> "Import an existing project"
3. 选择后端代码仓库
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

## 故障排除

### 前端问题

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

### 后端问题

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

## 总结

本项目实现了一个多Agents头脑风暴/辩论赛系统，具有以下特点：

1. **多MBTI人格智能体**：每个智能体都具有独特的MBTI人格特征，能够根据不同的人格生成不同的观点和讨论内容。

2. **三种会话模式**：支持头脑风暴、辩论赛和闲聊三种会话模式，满足不同的使用场景。

3. **实时交互**：通过前端状态管理实现实时更新，确保讨论的流畅性和实时性。

4. **数据持久化**：使用IndexedDB实现数据持久化，确保用户数据的安全性和一致性。

5. **可配置性**：用户可以配置LLM API和参与的智能体，满足不同的需求。

6. **易于部署**：提供了详细的本地运行和云端部署步骤，确保用户能够轻松部署和运行应用。

通过本项目，用户可以体验到多Agents交互的乐趣，进行头脑风暴、辩论赛和闲聊等活动，激发创意和思考。