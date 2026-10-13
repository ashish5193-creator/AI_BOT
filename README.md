# OpenAI Chat API Conversation Demo

This project demonstrates how to use the OpenAI Chat API in Python and highlights the difference between:

1. Sending independent prompts (stateless requests)
2. Sending multiple messages as conversation history (stateful conversation)

## Prerequisites

- Python 3.8+
- OpenAI Python SDK
- python-dotenv

## Installation

Install the required packages:

```bash
pip install openai python-dotenv pandas prompt_toolkit
```

## Environment Setup

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_api_key_here
```

Load the API key:

```python
from dotenv import load_dotenv
import os
import openai

load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
```

## Verify Setup

```python
print("hello")
```

Output:

```text
hello
```

Check whether the API key is loaded:

```python
print(openai.api_key)
```

## Example 1: Single Prompt

```python
prompt = "hello, how are you?"

response = openai.ChatCompletion.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": prompt}
    ],
    temperature=0.7,
    max_tokens=45
)

print(response["choices"][0]["message"]["content"])
```

### Sample Output

```text
Hello! I'm just a program, but I'm here and ready to help you.
How can I assist you today?
```

---

## Example 2: Multiple Independent Prompts

In this example, each request is sent separately.

```python
import openai

prompts = [
    "hello, how are you?",
    "i am a java developer ?",
    "Tell me about me"
]

for prompt in prompts:
    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
        max_tokens=45
    )

    print("Prompt:", prompt)
    print("Response:", response["choices"][0]["message"]["content"])
    print("-" * 50)
```

### Observation

Each API call is independent.

The model does not remember previous prompts because conversation history is not provided.

Example:

```text
Prompt: Tell me about me

Response:
I don't have any personal information about you...
```

---

## Example 3: Conversation Context

In this example, previous messages are included in the request.

```python
response = openai.ChatCompletion.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": "hello, how are you?"},
        {"role": "user", "content": "I am a Java developer"},
        {"role": "user", "content": "Tell me about me"}
    ]
)

print(response["choices"][0]["message"]["content"])
```

### Sample Output

```text
As a Java developer, you likely have experience with programming in Java,
working with frameworks, APIs, databases, and enterprise applications...
```

### Observation

The model can answer based on previous messages because the conversation history is included in the request.

---

## Key Learning

### Stateless Request

```python
messages=[
    {"role": "user", "content": prompt}
]
```

- No memory of previous requests
- Every API call starts fresh
- Suitable for independent questions

### Stateful Conversation

```python
messages=[
    {"role": "user", "content": "hello"},
    {"role": "user", "content": "I am a Java developer"},
    {"role": "user", "content": "Tell me about me"}
]
```

- Previous messages provide context
- The model can reference earlier information
- Enables chatbot-style conversations

---

## Project Structure

```text
project/
│
├── .env
├── main.py
├── requirements.txt
└── README.md
```

## requirements.txt

```text
openai
python-dotenv
pandas
prompt_toolkit
```

## Conclusion

This project demonstrates an important concept when working with Large Language Models:

- The model does not automatically remember previous API calls.
- Context must be explicitly passed through the `messages` parameter.
- Building conversational applications requires maintaining and sending chat history with every request.
