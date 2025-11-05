# Payanam_AI Travel Agent

An intelligent travel booking assistant powered by GPT-3.5-turbo and LangGraph that helps users search for flights, hotels, and complete travel packages through both a form-based interface and an AI chat assistant.

## System Architecture

```mermaid
graph TB
    User[User] -->|Access| WebUI[Web Interface]
    WebUI --> Form[Quick Search Form]
    WebUI --> Chat[AI Chat Assistant]
    
    Form -->|POST /search| Flask[Flask Server]
    Chat -->|POST /chat| Flask
    
    Flask -->|Invoke State| Agent[LangGraph Agent]
    
    Agent --> Node1[Classify Intent]
    Node1 --> Node2[Extract Parameters]
    Node2 --> Node3[Plan Tools]
    Node3 --> Node4[Execute Tools]
    Node4 --> Node5[Consolidate Results]
    
    Node4 --> Tools[Tool Execution]
    Tools --> FlightTools[Flight Tools]
    Tools --> HotelTools[Hotel Tools]
    Tools --> PackageTools[Package Tools]
    
    Node5 -->|Response| Flask
    Flask -->|HTML/JSON| WebUI
    
    classDef user fill:#e1f5fe
    classDef frontend fill:#f3e5f5
    classDef backend fill:#e8f5e8
    classDef agent fill:#fff3e0
    classDef tools fill:#fce4ec
    
    class User user
    class WebUI,Form,Chat frontend
    class Flask backend
    class Agent,Node1,Node2,Node3,Node4,Node5 agent
    class Tools,FlightTools,HotelTools,PackageTools tools
```

## Complete Request Flow

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant B as 🌐 Browser
    participant F as 🔥 Flask
    participant A as 🤖 Agent
    participant L as 🧠 LLM (GPT-3.5)
    participant T as 🛠️ Tools
    participant D as 💾 Mock DB

    U->>B: Enter Query
    B->>F: POST /chat {message}
    F->>A: invoke(state)
    
    Note over A: Step 1: Classify Intent
    A->>L: "Classify user intent"
    L->>A: intent="package_deal"
    
    Note over A: Step 2: Extract Parameters
    A->>L: "Extract travel params"
    L->>A: {origin, destination, dates}
    
    Note over A: Step 3: Plan Tools
    A->>A: Select tools based on intent
    
    Note over A: Step 4: Execute Tools
    A->>T: search_flights()
    T->>D: Query flight data
    D->>T: Return flights
    T->>A: flights[]
    
    A->>T: search_hotels()
    T->>D: Query hotel data
    D->>T: Return hotels
    T->>A: hotels[]
    
    A->>T: get_hotel_package()
    T->>A: package_info{}
    
    Note over A: Step 5: Consolidate
    A->>L: "Generate response"
    L->>A: final_response
    
    A->>F: return result
    F->>B: JSON response
    B->>U: Display results
```

## 🎯 Agent Workflow (LangGraph)

```mermaid
graph LR
    Start([User Query]) --> Classify[1. Classify Intent]
    Classify --> Extract[2. Extract Parameters]
    Extract --> Plan[3. Plan Tools]
    Plan --> Execute[4. Execute Tools]
    Execute --> Consolidate[5. Consolidate Results]
    Consolidate --> End([Final Response])
    
    style Start fill:#e1f5fe
    style End fill:#c8e6c9
    style Classify fill:#fff9c4
    style Extract fill:#ffecb3
    style Plan fill:#ffe0b2
    style Execute fill:#ffccbc
    style Consolidate fill:#d1c4e9
```

## 🧠 Intent Classification System

```mermaid
graph TD
    Query[User Query] --> LLM[GPT-3.5 Classifier]
    
    LLM --> Simple{simple_search?}
    LLM --> Price{price_focused?}
    LLM --> Explore{explore_alternatives?}
    LLM --> Deep{deep_analysis?}
    LLM --> Hotel{hotel_search?}
    LLM --> Package{package_deal?}
    
    Simple -->|Yes| T1[Tool: search_flights]
    Price -->|Yes| T2[Tools: search_flights<br/>analyze_prices]
    Explore -->|Yes| T3[Tools: search_flights<br/>analyze_prices<br/>suggest_alternatives]
    Deep -->|Yes| T4[Tools: search_flights<br/>analyze_prices<br/>get_route_info<br/>suggest_alternatives]
    Hotel -->|Yes| T5[Tools: search_hotels<br/>analyze_hotel_prices]
    Package -->|Yes| T6[Tools: search_flights<br/>search_hotels<br/>get_hotel_package]
    
    T1 --> Execute[Execute Selected Tools]
    T2 --> Execute
    T3 --> Execute
    T4 --> Execute
    T5 --> Execute
    T6 --> Execute
    
    classDef query fill:#e1f5fe
    classDef intent fill:#fff3e0
    classDef tools fill:#e8f5e8
    
    class Query query
    class Simple,Price,Explore,Deep,Hotel,Package intent
    class T1,T2,T3,T4,T5,T6,Execute tools
```

## 🛠️ Tool Architecture

```mermaid
graph TB
    subgraph FlightTools[✈️ Flight Tools]
        SF[search_flights<br/>Find available flights]
        AP[analyze_prices<br/>Price analysis]
        GR[get_route_info<br/>Route details]
        SA[suggest_alternatives<br/>Alternative options]
    end
    
    subgraph HotelTools[🏨 Hotel Tools]
        SH[search_hotels<br/>Find hotels]
        AH[analyze_hotel_prices<br/>Best value analysis]
    end
    
    
    Agent[🤖 Agent] --> FlightTools
    Agent --> HotelTools
    
    FlightTools --> Results[📊 Results]
    HotelTools --> Results
    
    classDef agent fill:#fff3e0
    classDef tools fill:#e8f5e8
    classDef results fill:#c8e6c9
    
    class Agent agent
    class SF,AP,GR,SA,SH,AH,GP tools
    class Results results
```

## 💬 Chat Flow Example

```mermaid
graph TD
    A[User: Find cheap flights to Madrid] --> B[Classify Intent]
    B --> C{Intent: price_focused}
    C --> D[Extract Parameters]
    D --> E[origin=Miami<br/>destination=Madrid<br/>date=2025-12-14]
    E --> F[Plan Tools]
    F --> G[Selected Tools:<br/>1. search_flights<br/>2. analyze_prices]
    G --> H[Execute search_flights]
    H --> I[Found 3 flights<br/>$395, $450, $520]
    I --> J[Execute analyze_prices]
    J --> K[Min: $395<br/>Avg: $455<br/>Recommendation: Excellent deal!]
    K --> L[Generate Natural Response]
    L --> M[Agent: I found great options...<br/>Air Europa $395<br/>Best deal: 12% lower than avg]
    
    style A fill:#e1f5fe
    style M fill:#c8e6c9
    style C fill:#fff9c4
    style G fill:#ffecb3
    style I fill:#e8f5e8
    style K fill:#e8f5e8
```

## 🌐 Deployment Architecture

```mermaid
graph TB
    subgraph Colab[☁️ Google Colab Environment]
        Notebook[📓 Jupyter Notebook]
        Flask[🔥 Flask Server :5007]
        Agent[🤖 LangGraph Agent]
        
        Notebook --> Flask
        Flask --> Agent
    end
    
    subgraph ngrok[🚇 ngrok Tunnel]
        Tunnel[Public URL Generator]
    end
    
    subgraph Internet[🌍 Internet]
        Users[👥 Users]
        Browser[🌐 Web Browsers]
    end
    
    Flask --> Tunnel
    Tunnel --> Browser
    Users --> Browser
    
    Agent --> OpenAI[🧠 OpenAI API<br/>GPT-3.5-turbo]
    
    classDef colab fill:#fff3e0
    classDef tunnel fill:#e1f5fe
    classDef internet fill:#f3e5f5
    classDef external fill:#e8f5e8
    
    class Notebook,Flask,Agent colab
    class Tunnel tunnel
    class Users,Browser internet
    class OpenAI external
```

## 📡 API Endpoint Flow

```mermaid
graph LR
    subgraph Endpoints[🔌 Flask Endpoints]
        E1[GET /<br/>Home Page]
        E2[POST /search<br/>Form Handler]
        E3[POST /chat<br/>Chat Handler]
        E4[GET /health<br/>Health Check]
    end
    
    subgraph Processing[⚙️ Processing]
        P1[Parse Request]
        P2[Create State]
        P3[Invoke Agent]
        P4[Generate Response]
    end
    
    subgraph Response[📤 Response]
        R1[HTML Page]
        R2[JSON Data]
    end
    
    E1 --> R1
    E2 --> P1
    E3 --> P1
    E4 --> R2
    
    P1 --> P2
    P2 --> P3
    P3 --> P4
    P4 --> R1
    P4 --> R2
    
    classDef endpoints fill:#e1f5fe
    classDef processing fill:#fff3e0
    classDef response fill:#c8e6c9
    
    class E1,E2,E3,E4 endpoints
    class P1,P2,P3,P4 processing
    class R1,R2 response
```

## 🐛 Troubleshooting Flow

```mermaid
graph TD
    Problem[🐛 Issue Detected] --> Type{Issue Type?}
    
    Type -->|API Key| K1[Check Key Valid]
    Type -->|Port| P1[Change Port Number]
    Type -->|ngrok| N1[Check Free Tier Limits]
    Type -->|Agent| A1[Check Flask Status]
    
    K1 --> K2{Valid?}
    K2 -->|No| K3[Revoke & Regenerate]
    K2 -->|Yes| K4[Check Billing Enabled]
    
    P1 --> P2[PORT = 5008]
    
    N1 --> N2{Limit Reached?}
    N2 -->|Yes| N3[Wait or Upgrade]
    N2 -->|No| N4[Restart Tunnel]
    
    A1 --> A2[Verify State Dict]
    A2 --> A3[Check Console Logs]
    
    K3 --> Solved[✅ Resolved]
    K4 --> Solved
    P2 --> Solved
    N3 --> Solved
    N4 --> Solved
    A3 --> Solved
    
    style Problem fill:#ffebee
    style Type fill:#fff9c4
    style Solved fill:#c8e6c9
```

## 🚀 Quick Start Flow

```mermaid
graph LR
    Start([Start]) --> Step1[1. Install Dependencies]
    Step1 --> Step2[2. Set API Keys]
    Step2 --> Step3[3. Upload to Colab]
    Step3 --> Step4[4. Run All Cells]
    Step4 --> Step5[5. Get ngrok URL]
    Step5 --> Step6[6. Access Web UI]
    Step6 --> End([Ready! 🎉])
    
    style Start fill:#e1f5fe
    style End fill:#c8e6c9
    style Step1,Step2,Step3,Step4,Step5,Step6 fill:#fff3e0
```

## ✨ Features

- **Dual Interface**: Quick search form + AI chat assistant
- **Smart Intent Classification**: Automatically understands user queries
- **Multi-Tool Agent**: Uses 7 specialized tools for comprehensive travel planning
- **Package Deals**: Combines flights and hotels for better prices
- **Price Analysis**: Real-time price comparisons and recommendations
- **Alternative Suggestions**: Finds nearby airports and alternative dates to save money
- **Natural Language Processing**: Chat naturally about your travel plans

## 🚀 Installation

### Prerequisites
- Python 3.8+
- OpenAI API Key
- ngrok account (for public URL)

### Step 1: Install Dependencies
```bash
pip install openai langchain-openai langgraph Flask pyngrok
```

### Step 2: Set API Keys
```python
# OpenAI API Key
OPENAI_API_KEY = "your-openai-api-key"
os.environ['OPENAI_API_KEY'] = OPENAI_API_KEY

# ngrok auth token (for public access)
!ngrok authtoken your-ngrok-token
```

### Step 3: Run the Application

**In Google Colab:**
1. Upload the notebook
2. Run all cells sequentially
3. The final cell will generate a public ngrok URL
4. Access the app through that URL

**Local Development:**
```python
# Run Flask locally
python app.py
# Then visit http://localhost:5007
```

## 📋 Usage Examples

### Quick Search Form
- Fill in origin, destination, dates, and passengers
- Click "Search Trips" for comprehensive results

### AI Chat Assistant
**Example queries:**
- "Find cheap flights to Madrid"
- "I need hotels in London for 3 nights"
- "Book me flights and hotel to Paris for 2 people"
- "What are alternative airports near New York?"
- "Compare prices for flights to Madrid"

## ⚙️ Configuration

### Supported Routes
- Miami ↔ Madrid
- New York ↔ London

### Default Values
```python
{
    "origin": "Miami",
    "destination": "Madrid",
    "date": "2025-12-14",
    "passengers": 1,
    "check_in": "2025-12-14",
    "check_out": "2025-12-17",
    "guests": 2
}
```

## 🔧 Technical Details

### State Management
```python
class TravelAgentState(TypedDict):
    query: str                      # User input
    intent: str                     # Classified intent
    extracted_params: Dict          # Travel parameters
    tools_used: List[str]           # Tools executed
    flights: List[Dict]             # Flight results
    hotels: List[Dict]              # Hotel results
    package_info: Dict              # Package details
    final_response: str             # AI response
    messages: List[str]             # Conversation log
```

### LLM Configuration
- **Model**: GPT-3.5-turbo
- **Temperature**: 0 (deterministic responses)
- **Provider**: OpenAI via LangChain

### Flask Endpoints
- `GET /` - Home page with dual interface
- `POST /search` - Process form submissions
- `POST /chat` - Handle chat messages (returns JSON)
- `GET /health` - Health check endpoint

## ⚠️ Important Notes

### Security Warning
**⚠️ CRITICAL**: The notebook contains an exposed OpenAI API key!

**Immediate actions required:**
1. Revoke the exposed API key from OpenAI dashboard
2. Generate a new key
3. Use environment variables: `OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')`

### Limitations
- Mock data only (no real flight/hotel APIs)
- Limited to pre-configured routes
- ngrok free tier has connection limits
- No persistent storage
- No user authentication

## 🛠️ Troubleshooting

**"API Key Invalid" Error**
- Verify key is correctly set
- Check if key hasn't been revoked
- Ensure billing is enabled on OpenAI account

**"Port Already in Use"**
```python
PORT = 5008  # Change to different port
```

**ngrok Connection Issues**
- Free tier has limits (40 connections/minute)
- Tunnel expires after 2 hours
- May need to restart for new URL

## 📝 Sample Test Queries

1. "Find flights from Miami to Madrid"
2. "I need the cheapest hotel in London"
3. "Book everything for a trip to Paris next month"
4. "What are alternatives to flying into Madrid?"
5. "Compare hotel prices in Madrid"
6. "I want a complete package to New York for 2 people"

## 🤝 Contributing

**Ways to extend:**
- Add more tools to `FlightTools` class
- Integrate real travel APIs (Amadeus, Skyscanner)
- Improve UI/UX design
- Add more intent types
- Implement user preferences and history

## 📄 License

This is a demonstration project for educational purposes.

## 🔗 Resources

- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [OpenAI API Docs](https://platform.openai.com/docs)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [ngrok Documentation](https://ngrok.com/docs)

---

**Built with**: OpenAI GPT-3.5, LangGraph, LangChain, Flask, ngrok
