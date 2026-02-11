# LangGraph Multi-Agent Chatbot with online vendor support integration

An intelligent multi-agent chatbot built with LangGraph that routes conversations to specialized agents and integrates with online vendor's API via a custom MCP (Model Context Protocol) server.

## Features

### Multi-Agent Routing
- **Emotional Support Agent**: Provides empathetic responses for emotional queries
- **Logical Agent**: Handles factual and analytical questions
- **Online Sellers Agent**: Access real-time Amazon seller data with tool calling

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Input                                │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│               Message Classifier                             │
│  (Determines: emotional | logical | amazon_query)            │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    Router                                    │
│         Routes to appropriate agent                          │
└─────┬─────────────────┬─────────────────┬──────────────────┘
      │                 │                 │
      ▼                 ▼                 ▼
┌──────────┐    ┌──────────┐    ┌────────────────────────────┐
│Therapist │    │ Logical  │    │      Seller Platform       │   
│  Agent   │    │  Agent   │    │       Support Agent        │
└──────────┘    └──────────┘    │ ┌────────────────────────┐ │
                                │ │ Python MCP Client      │ │
                                │ └───────────┬────────────┘ │
                                │             │ stdio        │
                                │             ▼              │
                                │ ┌────────────────────────┐ │
                                │ │ Node.js MCP Server     │ │
                                │ │ ( Client)              │ │
                                │ └───────────┬────────────┘ │
                                │             │ HTTPS        │
                                │             ▼              │
                                │ ┌────────────────────────┐ │
                                │ │ Client API             │ │
                                │ └────────────────────────┘ │
                                └────────────────────────────┘

## Usage

### Run the Chatbot

```bash
python main.py
```

### Example Interactions

**Emotional Support:**
```
You: I'm feeling stressed about work
Assistant: I hear that you're feeling stressed about work...
```

**Logical Questions:**
```
You: What is the capital of France?
Assistant: The capital of France is Paris.
```

**Client Support Queries:**
```
You: What are my orders from today?
Assistant: You have 3 orders today:
1. Order #113-5128268-1592224
   - Status: Pending
   - Total: $49.99
   - Items: 1
...
```

```
You: Show me my inventory levels
Assistant: Your current inventory summary:
- Total SKUs: 45
- Available units: 1,247
- Inbound units: 350
...
```

```
You: What are my sales metrics for this week?
Assistant: Sales metrics for this week:
- Total orders: 87
- Total revenue: $4,523.45
- Units sold: 156
...
```

## Project Structure

```
langgraph-project/
├── main.py                    # Main LangGraph agent with routing
├── seller_agent.py            # Tool factory & LangChain integration
├── mcp_client.py              # Python MCP client
├── test_mcp_client.py         # Integration test script
├── check_credentials.sh       # Credential validation script
├── pyproject.toml             # Python dependencies
├── .env                       # OpenAI API key
├── .venv/                     # Python virtual environment
│
├── amazon-mcp-server/         # Node.js MCP Server
│   ├── index.js               # MCP server entry point
│   ├── sp-api-client.js       # wrapper
│   ├── tools.js               # MCP tool definitions
│   ├── test-connection.js     # SP-API connection test
│   ├── package.json           # Node.js dependencies
│   ├── .env                   # credentials
│   └── node_modules/          # Node.js packages
│
└── README.md                  # This file
```

## How It Works

### Message Classification
The chatbot uses GPT-4o-mini to classify incoming messages:

```python
Literal["emotional", "logical", "amazon_query"]
```

### Agent Routing
Based on classification, the router directs to the appropriate agent:

| Classification | Agent | Capabilities |
|---------------|-------|--------------|
| `emotional` | Therapist Agent | Empathy, emotional support |
| `logical` | Logical Agent | Facts, analysis, information |
| `amazon_query` | Amazon Seller Agent | SP-API data via tools |

### Tool Calling Flow

1. **User asks**: "What are my orders?"
2. **Classifier**: Detects `amazon_query`
3. **Router**: Sends to Amazon Seller Agent
4. **LLM**: Decides to call `get_orders` tool
5. **Python**: Executes tool → connects to MCP server
6. **Node.js MCP Server**: Makes SP-API request
7. **Amazon SP-API**: Returns order data
8. **Response flows back** through the chain
9. **LLM**: Formats data into natural language
10. **User sees**: "You have 3 orders today..."
