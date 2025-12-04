# AI Agent系统流程图详解

## 1. 基础对话流程

```mermaid
graph LR
    A[用户] -->|User Prompt| B[AI模型]
    B -->|回复| A
```

## 2. 带人设的对话流程

```mermaid
graph LR
    A[用户] -->|User Prompt| C[系统]
    D[系统配置] -->|System Prompt| C
    C -->|组合请求| B[AI模型]
    B -->|个性化回复| A
```

## 3. Agent工具调用流程（System Prompt方式）

```mermaid
sequenceDiagram
    participant U as 用户
    participant A as Agent
    participant AI as AI模型
    participant T as Tools

    U->>A: 用户请求
    A->>A: 生成System Prompt<br/>(包含工具描述)
    A->>AI: User Prompt + System Prompt
    AI->>A: 返回工具调用请求<br/>(格式可能错误)
    A->>A: 解析请求
    alt 格式正确
        A->>T: 调用工具
        T->>A: 返回结果
        A->>AI: 转发结果
        AI->>A: 生成最终答案
        A->>U: 返回答案
    else 格式错误
        A->>AI: 重试请求
    end
```

## 4. Function Calling流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant A as Agent
    participant AI as AI模型
    participant T as Tools

    U->>A: 用户请求
    A->>AI: User Prompt + Function Calling格式<br/>(工具信息在独立字段)
    AI->>A: 标准化工具调用请求<br/>(格式固定)
    A->>T: 调用工具
    T->>A: 返回结果
    A->>AI: 转发结果
    AI->>A: 生成最终答案
    A->>U: 返回答案
    
    Note over AI: 服务器端可自动<br/>检测和重试错误
```

## 5. MCP架构流程

```mermaid
graph TB
    subgraph "用户层"
        U[用户]
    end
    
    subgraph "Agent层"
        AC[MCP Client<br/>AI Agent]
    end
    
    subgraph "AI模型层"
        AI[AI模型<br/>GPT/Claude/Gemini]
    end
    
    subgraph "工具服务层"
        MS[MCP Server]
        T1[Tool 1]
        T2[Tool 2]
        R1[Resource 1]
        P1[Prompt 1]
    end
    
    U -->|User Prompt| AC
    AC <-->|MCP协议| MS
    AC <-->|Function Calling| AI
    MS --> T1
    MS --> T2
    MS --> R1
    MS --> P1
```

## 6. 完整端到端流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant AC as Agent<br/>(MCP Client)
    participant MS as MCP Server
    participant AI as AI模型

    U->>AC: "我女朋友肚子疼怎么办？"
    
    Note over AC: 步骤1: 获取工具信息
    AC->>MS: 查询可用工具
    MS->>AC: 返回工具列表<br/>(web_browse等)
    
    Note over AC: 步骤2: 准备请求
    AC->>AC: 将工具信息转换为<br/>Function Calling格式
    
    Note over AC: 步骤3: 发送给AI
    AC->>AI: User Prompt +<br/>Function Calling工具列表
    
    Note over AI: 步骤4: AI决策
    AI->>AC: 调用web_browse工具<br/>搜索"肚子疼怎么办"
    
    Note over AC: 步骤5: 执行工具
    AC->>MS: 调用web_browse工具
    MS->>MS: 访问网页
    MS->>AC: 返回网页内容
    
    Note over AC: 步骤6: 转发结果
    AC->>AI: 转发网页内容
    
    Note over AI: 步骤7: 生成答案
    AI->>AC: "多喝热水"
    
    Note over AC: 步骤8: 返回用户
    AC->>U: 显示答案
```

## 7. 技术演进对比

```mermaid
graph TD
    subgraph "阶段1: 基础聊天"
        A1[用户] -->|User Prompt| B1[AI模型]
        B1 -->|回复| A1
    end
    
    subgraph "阶段2: 人设设定"
        A2[用户] -->|User Prompt| C2[系统]
        D2[System Prompt] --> C2
        C2 --> B2[AI模型]
        B2 --> A2
    end
    
    subgraph "阶段3: 工具调用"
        A3[用户] --> E3[Agent]
        E3 -->|System Prompt+工具| B3[AI模型]
        B3 -->|工具调用| E3
        E3 -->|执行| F3[Tools]
        F3 --> E3
        E3 --> A3
    end
    
    subgraph "阶段4: 标准化"
        A4[用户] --> E4[Agent]
        E4 -->|Function Calling| B4[AI模型]
        B4 -->|标准化调用| E4
        E4 -->|执行| F4[Tools]
        F4 --> E4
        E4 --> A4
    end
    
    subgraph "阶段5: 服务化"
        A5[用户] --> E5[Agent]
        E5 <-->|MCP协议| G5[MCP Server]
        E5 <-->|Function Calling| B5[AI模型]
        G5 --> F5[Tools/Resources]
    end
```

## 8. 错误处理流程对比

```mermaid
graph TB
    subgraph "System Prompt方式"
        SP1[AI返回] --> SP2{格式正确?}
        SP2 -->|是| SP3[执行工具]
        SP2 -->|否| SP4[Agent重试]
        SP4 --> SP1
        SP3 --> SP5[返回结果]
    end
    
    subgraph "Function Calling方式"
        FC1[AI返回] --> FC2{格式正确?}
        FC2 -->|是| FC3[执行工具]
        FC2 -->|否| FC4[服务器自动重试]
        FC4 --> FC1
        FC3 --> FC5[返回结果]
        
        Note over FC4: 用户无感知<br/>节省token
    end
```

## 9. MCP Server服务类型

```mermaid
graph LR
    MS[MCP Server] --> T[Tools<br/>函数调用]
    MS --> R[Resources<br/>数据资源]
    MS --> P[Prompts<br/>提示词模板]
    
    T --> T1[web_browse]
    T --> T2[file_operations]
    T --> T3[data_analysis]
    
    R --> R1[文件系统]
    R --> R2[数据库]
    R --> R3[API数据]
    
    P --> P1[代码生成模板]
    P --> P2[数据分析模板]
```

## 10. 多Agent共享MCP Server

```mermaid
graph TB
    subgraph "Agent层"
        A1[Agent 1]
        A2[Agent 2]
        A3[Agent 3]
    end
    
    subgraph "MCP Server"
        MS[统一工具服务]
        T[共享工具]
    end
    
    A1 -->|MCP协议| MS
    A2 -->|MCP协议| MS
    A3 -->|MCP协议| MS
    MS --> T
    
    Note over MS: 避免代码重复<br/>统一管理工具
```

---

## 流程图说明

以上流程图展示了：

1. **基础对话** → **人设设定** → **工具调用** → **标准化** → **服务化** 的演进过程
2. System Prompt 和 Function Calling 的区别
3. MCP 如何连接 Agent 和工具服务
4. 完整的端到端工作流程
5. 错误处理机制的对比
6. MCP Server 提供的三种服务类型
7. 多Agent共享工具的优势

这些流程图可以帮助理解整个AI Agent系统的运作机制。
