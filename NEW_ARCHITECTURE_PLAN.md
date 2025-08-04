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

The system supports both single-mode (locked to one agent) and multi-mode (flexible goal switching). **YODA defaults to multi-mode to enable the agent orchestration layer and provide the best user experience.**

---

## **Response Schema & Examples Integration to MCP Schema**

**Current State**: 
- MCP servers only provide `inputSchema` for tool parameters
- Goal teams manually write example responses in `example_conversation_history`
- No formal output schema validation

**Enhanced MCP Schema Integration:**

```python
# MCP Server: @company/auth-mcp
{
  "name": "ValidateJWT",
  "description": "Validate user JWT token and extract permissions",
  "inputSchema": {
    "type": "object",
    "properties": {
      "jwt_token": {"type": "string", "description": "User JWT token"}
    },
    "required": ["jwt_token"]
  },
  "responseSchema": {
    "type": "object", 
    "properties": {
      "valid": {"type": "boolean"},
      "user_id": {"type": "string"},
      "scopes": {"type": "array", "items": {"type": "string"}},
      "expires_at": {"type": "string", "format": "date-time"}
    }
  },
  "examples": {
    "success": {
      "valid": true,
      "user_id": "user_123", 
      "scopes": ["finance:read", "hr:write"],
      "expires_at": "2024-01-15T10:30:00Z"
    },
    "error": {
      "valid": false,
      "error": "Token expired"
    }
  }
}
```

**Implementation Purpose for Goal Teams:**

This enhancement is specifically designed so **goal teams know exactly what responses are expected to be fed to the LLM**:

1. **Schema + Examples**: Goal teams see precise response structure and concrete examples, eliminating guesswork
2. **MCP-Informed Manual Writing**: Goal teams copy from MCP examples to design accurate agent conversations

**Current example_conversation_history approach:**
```python
example_conversation_history="\n ".join([
    "user_confirmed_tool_run: <user clicks confirm on ValidateJWT tool>",
    "tool_result: { 'valid': true, 'user_id': 'user_123' }",  # Manual writing
    "agent: Your token is valid!"
])
```

---

## **JWT Token Authentication Architecture**

**Use Case**: YODA embedded in existing authenticated platforms, where business MCPs need user-specific authorization.

### **Complete JWT Flow Architecture**

```mermaid
graph TD
    subgraph Frontend ["🌐 Frontend (Embedded in Portal)"]
        PortalSession["Portal Session<br/>sessionId: abc123"]
        YodaUI["YODA Chat UI<br/>React Component"]
        PortalSession --> YodaUI
    end
    
    subgraph YODA ["🧠 YODA System"]
        API["FastAPI<br/>/send-prompt"]
        Workflow["AgentGoalWorkflow<br/>session_context: abc123"]
        JWTActivity["MCP Activity<br/>GetJWTFromSession"]
        
        API --> Workflow
        Workflow --> JWTActivity
    end
    
    subgraph AuthMCP ["🔐 Auth MCP Server"]
        GetJWTTool["GetJWTFromSession<br/>MCP Tool"]
        PortalAPI["Portal API Call<br/>GET /api/session/abc123"]
        
        JWTActivity --> GetJWTTool
        GetJWTTool --> PortalAPI
    end
    
    subgraph BusinessMCP ["🏢 Business MCP Server"]
        CustomerTool["GetCustomerOrders<br/>Requires JWT"]
        BusinessAPI["Business API<br/>Authorization: Bearer jwt_token"]
        
        CustomerTool --> BusinessAPI
    end
    
    %% JWT Flow
    YodaUI -->|"{ prompt, sessionId }"| API
    PortalAPI -->|"{ jwt_token, user_id, scopes }"| GetJWTTool
    GetJWTTool -->|"JWT Response"| JWTActivity
    JWTActivity -->|"Store JWT"| Workflow
    Workflow -->|"Pass JWT to tools"| CustomerTool
    
    classDef frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef yoda fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef auth fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef business fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class Frontend frontend
    class YODA yoda
    class AuthMCP auth
    class BusinessMCP business
```

### **Implementation Changes (Minimal)**

**1. Frontend Changes (~3 lines)**
```javascript
// frontend/src/services/api.js
async sendMessage(message) {
    const sessionId = window.portalSession?.id;  // ← NEW: Get from embedded context
    const res = await fetch(`${API_BASE_URL}/send-prompt`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
            prompt: message,
            session_context: { sessionId }  // ← NEW: Pass session
        })
    });
}
```

**2. Backend Changes (~5 lines)**
```python
# api/main.py
@app.post("/send-prompt")
async def send_prompt(request: dict):
    prompt = request.get("prompt")
    session_context = request.get("session_context")  # ← NEW
    
    combined_input = CombinedInput(
        tool_params=AgentGoalWorkflowParams(None, None),
        agent_goal=get_initial_agent_goal(),
        session_context=session_context  # ← NEW
    )
```

**3. Workflow Changes (~8 lines)**
```python
# workflows/agent_goal_workflow.py
async def run(self, combined_input: CombinedInput):
    self.session_context = combined_input.session_context  # ← NEW
    
    # Get JWT if session provided
    if self.session_context:
        jwt_result = await workflow.execute_activity(
            "GetJWTFromSession",  # ← NEW MCP tool (auth-mcp server)
            self.session_context
        )
        self.jwt_token = jwt_result.get("jwt_token")  # ← NEW
```

**4. Tool Execution Changes (~2 lines)**
```python
# workflows/workflow_helpers.py
mcp_args = tool_data["args"].copy()
mcp_args["server_definition"] = goal.mcp_server_definition
if hasattr(workflow_instance, 'jwt_token'):
    mcp_args["jwt_token"] = workflow_instance.jwt_token  # ← NEW
```

### **New MCP Server: @company/auth-mcp**

```python
# This is a NEW MCP server (external to YODA)
{
  "name": "GetJWTFromSession",
  "description": "Exchange portal session ID for JWT token",
  "inputSchema": {
    "type": "object",
    "properties": {
      "sessionId": {"type": "string", "description": "Portal session identifier"}
    },
    "required": ["sessionId"]
  },
  "responseSchema": {
    "type": "object",
    "properties": {
      "jwt_token": {"type": "string"},
      "user_id": {"type": "string"},
      "scopes": {"type": "array", "items": {"type": "string"}},
      "expires_at": {"type": "string", "format": "date-time"}
    }
  },
  "examples": {
    "success": {
      "jwt_token": "eyJhbGciOiJIUzI1NiIs...",
      "user_id": "user_123",
      "scopes": ["finance:read", "hr:write"],
      "expires_at": "2024-01-15T10:30:00Z"
    }
  }
}
```

**Total YODA Changes: ~18 lines across 4 files**

**Why So Minimal?** YODA's MCP architecture is already designed for this - we're just adding session context to the existing flow and using MCP tools for authentication just like any other business logic.
