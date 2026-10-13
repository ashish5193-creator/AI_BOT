# AI Weather Assistant Using OpenAI Function Calling

This project demonstrates how to use OpenAI Function Calling to connect an AI model with external APIs.

The application allows the model to:

1. Understand a user's weather-related question.
2. Determine when a weather function should be called.
3. Extract the city name from the user request.
4. Fetch real weather information using external APIs.
5. Generate a natural language response based on live weather data.

---

# Architecture

```text
User Question
      |
      V
OpenAI Model
      |
      | Function Call Request
      V
get_current_weather()
      |
      +--> OpenStreetMap API
      |       |
      |       +--> Latitude & Longitude
      |
      +--> Meteosource Weather API
              |
              +--> Current Weather Data
                      |
                      V
               OpenAI Model
                      |
                      V
              Human-Friendly Response
```

---

# Prerequisites

- Python 3.8+
- OpenAI SDK
- Requests
- Python Dotenv

Install dependencies:

```bash
pip install openai requests python-dotenv
```

---

# Environment Setup

Create a `.env` file:

```env
OPENAI_API_KEY=your_openai_api_key
```

Load the environment variable:

```python
from dotenv import load_dotenv
import os
import openai

load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
```

---

# Step 1: Convert City Name to Coordinates

The project uses OpenStreetMap's Nominatim API.

```python
def get_coordinates(city):
```

Example:

```python
lat, lon = get_coordinates("Lucknow")
```

Output:

```text
26.8467
80.9462
```

---

# Step 2: Retrieve Weather Information

The weather data is fetched from Meteosource through RapidAPI.

```python
def get_current_weather(city):
```

This function:

- Finds latitude and longitude
- Calls the weather API
- Returns JSON weather data

---

# Function Definition

The model is provided with a tool definition describing the weather function.

```python
functions = [
    {
        "name": "get_current_weather",
        "description": "Get the current weather in a given location",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city to get weather for"
                }
            },
            "required": ["city"]
        }
    }
]
```

---

# User Query

```python
user_message = "What is the weather in Lucknow?"
```

Messages list:

```python
messages = [
    {
        "role": "user",
        "content": user_message
    }
]
```

---

# First OpenAI Call

The model decides whether a function should be called.

```python
completion = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    functions=functions,
    function_call="auto"
)
```

Example response:

```json
{
  "function_call": {
    "name": "get_current_weather",
    "arguments": {
      "city": "Lucknow"
    }
  }
}
```

---

# Extract Function Arguments

```python
response = completion.choices[0].message

args = json.loads(
    response.to_dict()["function_call"]["arguments"]
)

city = args["city"]
```

Output:

```text
Lucknow
```

---

# Execute the Weather Function

```python
function_result = get_current_weather(city)
```

Example response:

```json
{
   "temperature": 27.4,
   "humidity": 88,
   "summary": "Cloudy"
}
```

---

# Add Function Result To Conversation

```python
messages.append(response)

messages.append({
    "role": "function",
    "name": "get_current_weather",
    "content": json.dumps(function_result)
})
```

At this point the model has access to:

- Original user question
- Requested function call
- Weather API response

---

# Second OpenAI Call

Generate a natural language answer.

```python
final_response = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    functions=functions
)
```

---

# Final Response

Example:

```text
The current weather in Lucknow is:

• Condition: Cloudy
• Temperature: 27.4°C
• Feels Like: 31.8°C
• Humidity: 88%
• Wind Speed: 2.7 m/s
• Pressure: 1009 hPa

The weather is expected to remain predominantly cloudy throughout the day.
```

---

# Understanding Function Calling

Without Function Calling:

```text
User --> Model
```

The model can only use information from its training.

With Function Calling:

```text
User --> Model
          |
          V
      Function
          |
          V
      External API
          |
          V
        Model
          |
          V
       Response
```

Function Calling allows the model to:

- Access real-time data
- Query databases
- Trigger business logic
- Integrate with APIs
- Perform calculations
- Execute workflows

---

# Project Structure

```text
weather-assistant/
│
├── .env
├── weather_assistant.py
├── requirements.txt
└── README.md
```

---

# requirements.txt

```text
openai
python-dotenv
requests
```

---

# Sample Execution

```text
User:
What is the weather in Lucknow?

Model:
Function Requested:
get_current_weather(city="Lucknow")

Weather API:
Returned weather data

Assistant:
The current weather in Lucknow is cloudy with a temperature of 27.4°C and humidity of 88%.
```

---

# Key Concepts Learned

✅ OpenAI Function Calling

✅ External API Integration

✅ Weather Data Retrieval

✅ Conversation Context Management

✅ JSON Argument Parsing

✅ Multi-step AI Workflows

✅ Tool-Augmented LLM Applications

---

# Security Notes

### Never Hardcode API Keys

Avoid:

```python
headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}
```

Instead:

```python
RAPID_API_KEY = os.getenv("RAPID_API_KEY")
```

Store secrets inside:

```env
OPENAI_API_KEY=xxxx
RAPID_API_KEY=xxxx
```

### Do Not Commit Secrets

Add a `.gitignore` file:

```gitignore
.env
__pycache__/
```

---

# Conclusion

This project demonstrates a practical implementation of OpenAI Function Calling. Instead of relying solely on the model's knowledge, the assistant dynamically invokes external weather services, retrieves live weather information, and produces an intelligent, human-readable response.

This pattern forms the foundation of modern AI agents capable of interacting with APIs, databases, enterprise systems, and real-world services.
``
