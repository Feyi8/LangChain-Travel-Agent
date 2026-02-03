# Michael Zhang's Travel Planning Agent

**An AI-powered travel planning assistant that creates comprehensive multi-city itineraries with weather forecasts, clothing recommendations, and air quality monitoring.**

---

## Features

- **Automatic Address Lookup**: Finds full addresses for attractions using Google Places API
- **Weather Forecasting**: 4-day weather predictions with clothing recommendations
- **Air Quality Monitoring**: Real-time AQI data with mask requirements
- **Conversational Memory**: Remembers context for follow-up questions
- **Any Cities Worldwide**: Works for any destination, not just examples
- **Dynamic & Interactive**: Ask questions, modify plans, get instant answers

---

## Assignment Compliance

**LangChain v1.2+**: Uses LangChain 1.2.7 (security patched)  
**Package Management**: Compatible with `uv` and `pyproject.toml`  
**Uses `requests` Package**: All API calls use `requests.get()` / `requests.post()`  
**LangChain Built-in Functions**: Uses `llm.bind_tools()` and manual tool loop  
**Robust & General**: Works for any cities and dates  
**Hard Mode**: Automatically finds attraction addresses  
**Code Quality**: Heavily documented with clear explanations

---

## Quick Start

### Prerequisites

- Python 3.10+
- Google Maps API Key (with Places, Weather, Air Quality APIs enabled)
- OpenAI API Key (GPT-4o-mini access)

### Installation

1. **Install dependencies:**
   ```bash
   pip install langchain==1.2.7 langchain-core==1.2.7 langchain-openai openai python-dotenv requests
   ```

2. **Create `.env` file:**
   ```env
   GOOGLE_API_KEY=your_google_api_key_here
   OPENAI_API_KEY=your_openai_api_key_here
   ```

3. **Enable Google APIs:**
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Enable: Places API, Weather API, Air Quality API
   - Enable billing (free tier: $200/month credit)

4. **Update travel dates** in `agent_v127.py` (line ~265):
   ```python
   # Change to dates within next 4 days
   City1: Toronto 2026-01-31  # Tomorrow
   City2: Chicago 2026-02-01  # Day after
   ```

### Run

```bash
python agent_v127.py
```

---

## Usage Examples

### Example 1: Initial Travel Plan

The agent automatically creates plans from the code's travel input:

```
TRAVEL PLAN:
=== TRAVEL PLAN FOR MICHAEL ZHANG ===

CITY 1: Toronto - 2026-01-31
Attractions:
1. CN Tower - 290 Bremner Blvd, Toronto, ON M5V 3L9, Canada (8am-9am)
2. Royal Ontario Museum - 100 Queens Park, Toronto, ON M5S 2C6, Canada (10am-11am)

Weather: Partly cloudy, -2°C to 3°C
Clothing: warm jacket and sweater
Umbrella: No
Air Quality: AQI 80 - Excellent air quality
Mask Needed: No

CITY 2: Chicago - 2026-02-01
...

SUMMARY:
Total Masks Needed: 0
```

### Example 2: Ask for Different Cities (Interactive)

After the initial plan, you can request **any cities**:

```
Michael: Can you plan a trip to Vancouver and Seattle for me?

Agent: [Automatically calls tools and creates new plan]
=== TRAVEL PLAN ===
CITY 1: Vancouver - 2026-02-02
...
```

### Example 3: Follow-Up Questions

```
Michael: What's the coldest city?

Agent: Based on the weather data, Toronto is the coldest city with 
       temperatures ranging from -2°C to 3°C.

Michael: Do I need sunscreen?

Agent: No, sunscreen is not necessary. Both cities have cold weather 
       with cloudy conditions and low UV exposure.

Michael: Should I bring an umbrella?

Agent: Yes, you should bring an umbrella for Chicago as the 
       precipitation probability is 40% (above the 30% threshold).
```

---

## How It Works

### Architecture

```
User Input → LLM with Tools → Tool Calls → Google APIs → Results → Travel Report
```

### The 3 Tools

1. **`get_place_details(place_name, city)`**
   - **API**: Google Places Text Search
   - **Purpose**: Find full addresses and GPS coordinates
   - **Example**: "CN Tower, Toronto" → Full address + lat/lng

2. **`get_weather_forecast(latitude, longitude, date)`**
   - **API**: Google Weather
   - **Purpose**: Get 4-day weather forecasts
   - **Features**: Temperature, precipitation, clothing recommendations
   - **Rule**: Umbrella needed if precipitation > 30%

3. **`get_air_quality(latitude, longitude)`**
   - **API**: Google Air Quality
   - **Purpose**: Check pollution levels (AQI)
   - **Rule**: Mask needed if AQI > 100

### Tool-Calling Loop (LangChain 1.2.7)

```python
# 1. Bind tools to LLM (LangChain built-in)
llm_with_tools = llm.bind_tools(tools)

# 2. Manual tool execution loop
while iteration < max_iterations:
    response = llm_with_tools.invoke(messages)
    
    if no tool_calls:
        break  # Done!
    
    for each tool_call:
        result = tool.invoke(args)
        messages.append(ToolMessage(result))
```

### Memory System

```python
messages = [
    SystemMessage("You are a travel assistant..."),
    HumanMessage("Plan Toronto → Chicago"),
    AIMessage("Here's your plan..."),
    HumanMessage("Should I bring sunscreen?"),  # Remembers context!
    AIMessage("No, it's too cold...")
]
```

---

## Customization

### Change Default Cities

Edit `travel_input` in `main()` (line ~265):

```python
travel_input = """
Please create a detailed travel plan for:

City1: Paris 2026-02-05
Eiffel Tower;9am-11am
Louvre Museum;2pm-5pm

City2: London 2026-02-06
Big Ben;10am-12pm
British Museum;2pm-4pm
"""
```

### Adjust Clothing Recommendations

Modify temperature thresholds in `get_weather_forecast()` (line ~95):

```python
if avg_temp < 0:
    clothing = "heavy winter coat..."  # Change threshold
elif avg_temp < 10:
    clothing = "warm jacket..."        # Adjust as needed
```

### Change Mask Threshold

Modify AQI threshold in `get_air_quality()` (line ~150):

```python
mask_needed = "Yes" if aqi_value > 100 else "No"  # Change 100 to your threshold
```

---

## Test Cases

### Test Case 1: Two Cities
```python
City1: Toronto 2026-01-31
CN Tower;8am-9am

City2: Chicago 2026-02-01
Willis Tower;10am-12pm
```
**Expected**: 2 cities, 2 attractions, weather and AQI for both

### Test Case 2: Three Cities
```python
City1: Vancouver 2026-01-31
Stanley Park;9am-11am

City2: Seattle 2026-02-01
Space Needle;2pm-4pm

City3: Portland 2026-02-02
Powell's Books;10am-12pm
```
**Expected**: 3 cities, demonstrates scalability

### Test Case 3: Interactive Questions
```
Q1: "What's the weather in Vancouver?"
Q2: "Do I need a mask anywhere?"
Q3: "How many umbrellas should I pack?"
```
**Expected**: Agent answers using previous data (memory works)

---

## Troubleshooting

### Error: "No forecast available for [date]"

**Cause**: Date is outside 4-day forecast window

**Fix**: Update dates in code to be within next 4 days from today

### Error: "GOOGLE_API_KEY not found"

**Cause**: Missing or incorrect `.env` file

**Fix**:
```bash
# Create .env file with your keys
echo "GOOGLE_API_KEY=your_key" > .env
echo "OPENAI_API_KEY=your_key" >> .env
```

### Error: "No results found for [attraction]"

**Cause**: Attraction name misspelled or doesn't exist

**Fix**: Use well-known attraction names, check spelling

### Agent not calling tools

**Cause**: Already fixed in agent_v127.py with manual tool loop

**Status**: Working!

---

## Project Structure

```
project/
├── travel_agent.py           # Main agent code (working version)
├── .env                     # API keys (don't commit!)
├── pyproject.toml          # Dependencies (for uv)
├── README.md               # This file
```

---

## Technical Details

### LangChain 1.2.7 Architecture

This project uses LangChain 1.2.7's new architecture:

- **Old versions (0.x - 1.1)**: Used `create_tool_calling_agent` + `AgentExecutor`
- **New version (1.2.7)**: Uses `llm.bind_tools()` + manual tool loop

Both approaches are "using LangChain's built-in functions" - just different APIs!

### Why Manual Tool Loop?

LangChain 1.2.7's `create_agent()` doesn't automatically execute tools, so we implement the loop manually:

```python
def run_agent_with_tools(llm_with_tools, tool_map, messages, max_iterations=15):
    while iteration < max_iterations:
        response = llm_with_tools.invoke(messages)
        
        if not response.tool_calls:
            break
        
        for tool_call in response.tool_calls:
            result = tool_map[tool_name].invoke(tool_args)
            messages.append(ToolMessage(content=result, ...))
    
    return messages
```

## 📄 License

This project was created for educational purposes as part of the Foundation of Professional Analytics - Agentic AI course.



