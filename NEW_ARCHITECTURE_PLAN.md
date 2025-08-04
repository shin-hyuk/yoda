# Enhanced Temporal AI Agent - Architecture Documentation

## 🎯 **Executive Summary**

Document the current MCP server integration system and explore potential enhancements for streamlined tool development workflows while maintaining full Temporal orchestration capabilities.

**🔑 Key Insight:** The system already supports excellent MCP integration with NPM-based distribution and runtime tool discovery.

---

## 📊 **Architecture Overview**

### **MCP Server Integration Architecture**

**Quick Setup:**
1. Add server definition to `shared/mcp_config.py`
2. Reference in goal files 
3. Publish via NPM
4. Tools auto-discovered at runtime

```mermaid
graph TB
    User["<b>User Input</b>"] --> Frontend["<b>React Frontend</b><br/><i>localhost:5173</i>"]
    Frontend --> API["<b>FastAPI Server</b><br/><i>localhost:8000</i>"]
    API --> Temporal["<b>Temporal Server</b><br/><i>localhost:7233</i>"]
    
    subgraph YodaBrain ["🧠 <b>YODA - The Orchestrator Brain</b>"]
        Temporal --> Worker["<b>Temporal Worker</b>"]
        Worker --> AgentWorkflow["<b>AgentGoalWorkflow</b><br/><i>workflows/agent_goal_workflow.py</i>"]
        AgentWorkflow --> LLMActivity["<b>LLM Activities</b><br/><i>activities/tool_activities.py</i>"]
        AgentWorkflow --> MCPClientManager["<b>MCP Client Manager</b><br/><i>shared/mcp_client_manager.py</i>"]
        MCPClientManager --> RuntimeDiscovery["<b>Runtime Tool Discovery</b><br/><i>mcp_list_tools activity</i>"]
    end
    
    subgraph ExternalMCP ["🌐 <b>External MCP Ecosystem</b>"]
        BusinessAPI["<b>Business API</b><br/><i>npx @company/business-mcp</i>"]
        AnalyticsService["<b>Analytics Service</b><br/><i>npx @company/analytics-mcp</i>"]
        IntegrationHub["<b>Integration Hub</b><br/><i>npx @company/integration-mcp</i>"]
    end
    
    subgraph Infrastructure ["🏗️ <b>Infrastructure</b>"]
        Postgres[("<b>PostgreSQL</b><br/><i>Temporal Persistence</i>")]
        NPMRegistry[("<b>NPM Registry</b><br/><i>MCP Package Distribution</i>")]
    end
    
    %% Connections
    MCPClientManager --> BusinessAPI
    MCPClientManager --> AnalyticsService  
    MCPClientManager --> IntegrationHub
    BusinessAPI -.->|Discovery| RuntimeDiscovery
    AnalyticsService -.->|Discovery| RuntimeDiscovery
    IntegrationHub -.->|Discovery| RuntimeDiscovery
    Temporal --> Postgres
    ExternalMCP --> NPMRegistry
    
    classDef brain fill:#e8f5e8,stroke:#2e7d32,stroke-width:3px
    classDef external fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef infra fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class YodaBrain brain
    class ExternalMCP external
    class Infrastructure infra
```

## 👥 **Team Separation Pattern**

### **🛠️ Tool Development Team**
```bash
# 1. Build MCP Server following MCP protocol standard
# Node.js, Python, Go, etc. - any language that can create NPM executable

# 2. Develop tools with correct schema (automatically validated)
{
  "name": "GetCustomerOrders",
  "description": "Retrieve customer order history",
  "inputSchema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string", "description": "Customer identifier"},
      "date_range": {"type": "string", "description": "Date range filter"}
    },
    "required": ["customer_id"]
  }
}

# 3. Publish to NPM
npm publish @company/business-mcp

# 4. Notify goal team
"Hey, we added @company/business-mcp with order management and customer tools"
```

---

### **🎨 Goal Team** (Agent Design & User Experience Focus)

**Responsibility**: Agent personas, conversation flows, user experience

**Process**: Discover MCP tools and design agents
```python
# Step 1: Add MCP server definition to shared/mcp_config.py
def get_business_mcp_server_definition(included_tools: list[str]) -> MCPServerDefinition:
    return MCPServerDefinition(
        name="business-mcp",
        command="npx",
        args=["-y", "@company/business-mcp"],
        included_tools=included_tools,
    )

# Step 2: Tools are automatically discovered at runtime via mcp_list_tools
# Returns schema like:
{
  "GetCustomerOrders": {
    "name": "GetCustomerOrders",
    "description": "Retrieve customer order history", 
    "inputSchema": {
      "properties": {
        "customer_id": {"type": "string", "description": "Customer identifier"},
        "date_range": {"type": "string", "description": "Date range filter"}
      },
      "required": ["customer_id"]
    }
  }
}

# Step 3: Design goals using MCP server reference
# goals/business.py - Focus on agent behavior and UX  
goal_business_assistant = AgentGoal(
    agent_name="Business Assistant",
    agent_friendly_description="Help with customer orders and business operations",
    starter_prompt="Hi! I can help with your business operations...",
    
    # Reference MCP server (tools auto-discovered at runtime)
    mcp_server_definition=get_business_mcp_server_definition(
        included_tools=["GetCustomerOrders", "UpdateOrderStatus"]
    ),
)
```

---

## **Low-Friction Team Workflow**

The MCP integration enables minimal coordination between teams through a simple 5-step process:

1. **Tool Development Team**: Build + publish MCP server to NPM (any tech stack + MCP protocol)
2. **Goal Team**: Add ~10 lines to `shared/mcp_config.py` (centralized registry)
3. **System Integration**: Goals reference MCP servers - tools automatically discovered at runtime
4. **Distribution**: NPM handles all distribution - no manual coordination needed!
5. **Immediate Availability**: Tools become available instantly when goal activates

**Key Benefits**: Teams work independently, MCP protocol ensures compatibility, runtime discovery eliminates manual registration.

---

## **Goal Switching Architecture**

The system implements a dual-path goal switching mechanism that gives users maximum flexibility:

```mermaid
graph TD
    subgraph AnyGoal ["Any Active Goal (Travel, HR, Finance, etc.)"]
        direction TB
        UserInGoal[User in Middle of Goal<br/>e.g., Booking Flights]
        GoalTools[Goal-Specific Tools<br/>+ ListAgents tool<br/>auto-added in multi-goal mode]
        
        UserInGoal --> GoalTools
    end
    
    subgraph TwoEscapeRoutes ["Two Escape Routes from Any Goal"]
        direction TB
        
        subgraph Route1 ["Route 1: Natural Completion"]
            direction TB
            CompleteAllSteps[Complete ALL Goal Steps<br/>e.g., Search → Book → Invoice]
            SystemTrigger[System Triggers<br/>'pick-new-goal']
            AutoReturn[Auto-Return to<br/>goal_choose_agent_type]
            
            CompleteAllSteps --> SystemTrigger
            SystemTrigger --> AutoReturn
        end
        
        subgraph Route2 ["Route 2: Instant Switch"]
            direction TB
            UserCallsListAgents[User Calls ListAgents<br/>from any goal]
            InstantJump[Instant Jump to<br/>goal_choose_agent_type]
            ChangeGoalAvailable[ChangeGoal Tool<br/>Now Available]
            
            UserCallsListAgents --> InstantJump
            InstantJump --> ChangeGoalAvailable
        end
    end
    
    subgraph AgentSelection ["Agent Selection Hub (goal_choose_agent_type)"]
        direction TB
        ListAgentsSilent[ListAgents Runs Silently<br/>Shows All Available Agents]
        ChangeGoalTool[ChangeGoal Tool Available<br/>Can Switch to ANY Goal]
        NewGoalSelected[User Selects New Goal<br/>e.g., HR, Finance, Travel]
        
        ListAgentsSilent --> ChangeGoalTool
        ChangeGoalTool --> NewGoalSelected
    end
    
    %% Connections
    GoalTools -->|Route 1| Route1
    GoalTools -->|Route 2| Route2
    AutoReturn --> AgentSelection
    ChangeGoalAvailable --> AgentSelection
    NewGoalSelected --> AnyGoal
    
    classDef anyGoal fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef route1 fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef route2 fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef agentHub fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class AnyGoal anyGoal
    class Route1 route1
    class Route2 route2
    class AgentSelection agentHub
```

Users never get "stuck" in a goal - they always have two escape routes. The agent selection hub serves as the universal switching point, providing instant gratification without needing to complete the current goal.

The system supports both single-mode (locked to one agent) and multi-mode (flexible goal switching). YODA defaults to multi-mode to enable the agent orchestration layer and provide the best user experience.