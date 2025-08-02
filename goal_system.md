graph TB
    User[User Input] --> API[FastAPI MCP Server]
    API --> Temporal[Temporal Workflow Engine]
    Temporal --> Worker[Temporal Worker]
    
    subgraph ServerSide ["🔧 Orchestration Server - Temporal Orchestration"]
        direction TB
        Worker --> Orchestrator[Workflow Orchestrator]
        Orchestrator --> LLM[LLM Activities]
        Orchestrator --> MCPClient[MCP Client/Tool Router]
        
        subgraph DataLayer ["📊 Data Layer"]
            Postgres[(PostgreSQL<br/>- Workflow Events<br/>- Conversation History<br/>- Tool Execution Results<br/>- Agent State)]
            Temporal --> Postgres
            LocalStorage[Browser LocalStorage<br/>- UI State<br/>- Chat History Cache]
        end
        
        subgraph Infrastructure ["🏗️ Infrastructure"]
            Docker[Docker Compose<br/>- Temporal Server<br/>- PostgreSQL<br/>- API Server<br/>- Worker<br/>- UI]
            TemporalUI[Temporal UI<br/>- Workflow Monitoring<br/>- Execution History]
        end
    end
    
    MCPClient -.->|MCP Protocol| ToolAPI[Tool Server API]
    
    subgraph ToolServer ["🛠️ Tool Server (Tool Development Team)"]
        direction TB
        ToolAPI --> ToolRouter[Tool Router]
        ToolRouter --> NativeTools[Native Tools<br/>- Python Functions<br/>- Domain-specific Logic]
        ToolRouter --> MCPAdapter[MCP Adapter]
        MCPAdapter --> MCPTools[MCP Tools]
        MCPTools --> External[External MCP Servers<br/>- Stripe API<br/>- Database Connectors<br/>- Third-party Services]
        
        subgraph ToolRegistry ["📋 Tool Registry"]
            AutoDiscovery[Auto-Discovery Tool Registration]
            Metadata[Tool Metadata & Schemas]
            GoalMapping[Goal-Tool Mapping]
        end
        
        subgraph ToolInfra ["🏗️ Tool Infrastructure"]
            ToolDocker[Docker<br/>- Tool Server<br/>- Tool Database<br/>- Caching Layer]
        end
    end
    
    classDef serverBox fill:#e1f5fe,stroke:#0277bd,stroke-width:3px
    classDef toolBox fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px
    classDef dataBox fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef infraBox fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    
    class ServerSide serverBox
    class ToolServer toolBox
    class DataLayer dataBox
    class Infrastructure,ToolInfra infraBox