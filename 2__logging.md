# Logging

Given the frequently non-deterministic nature of working with LLMs, I've found it useful to log all inputs to and outputs from them. For our purposes here, we'll do simple logging through individual JSON files, for simplicity and reliability.

We name the files primarily by timestamp (allowing easy access to most recent), appending a UUID to prevent overwriting in the unlikely event of two files being written at the same exact time.

```python
def get_filename():
    return f"{time.time_ns()}_{uuid.uuid4()}.json"
```

Example generated filename:

```
1693765445090910000_fb086c5f-583f-416e-a07f-d1168eefaeab.json
```

The full logging apparatus looks like the following:

```python
import json
from pprint import pprint
import openai
import time
import uuid

with open("tokens.json", "r") as f:
    openai.api_key = json.load(f)["OPENAI_API_KEY"]


def get_filename():
    return f"{time.time_ns()}_{uuid.uuid4()}.json"


def write_log_file(subdir, filename, data):
    with open(f"./data/{subdir}/{filename}", "w") as f:
        json.dump(data, f, indent=2)


def make_payload(
    system_prompt,
    user_prompt,
    model="gpt-4",
    temperature=0.0,
):
    return {
        "model": model,
        "messages": [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ],
        "n": 1,
        "temperature": 0.0,
        "top_p": 1.0,
        "frequency_penalty": 0.0,
        "presence_penalty": 0.0,
        "stop": None,
    }


def make_chatcompletion_api_call(params):
    response = openai.ChatCompletion.create(**params)
    response = json.loads(json.dumps(response))  # TODO: hackjob
    return response


def request_chatcompletion(system_prompt, user_prompt):
    filename = get_filename()

    data = {
        "system_prompt": str(system_prompt),
        "user_prompt": str(user_prompt),
        "request": make_payload(system_prompt, user_prompt),
    }

    write_log_file("requests", filename, data)

    response = make_chatcompletion_api_call(data["request"])

    data["response"] = response
    data["response_text"] = response["choices"][0]["message"]["content"]
    data["id"] = data["response"]["id"]

    write_log_file("responses", filename, data)

    return data["response"]


if __name__ == "__main__":
    system_prompt = ""
    user_prompt = 'Say "Hello World!"'
    resp = request_chatcompletion(system_prompt, user_prompt)
    output = resp["choices"][0]["message"]["content"]
    print(output)
```

Note that the request data is logged before making the API call, to ensure that the request prompt is still logged in case of API/network error. It's useful to have a record of earlier versions of prompts. The request log file would look something like the following:

`./data/requests/1693765467279010000_a7bfa298-4bab-41fc-b63d-315f442c8443.json`
```json
{
  "system_prompt": "",
  "user_prompt": "Say \"Hello World!\"",
  "request": {
    "model": "gpt-4",
    "messages": [
      {
        "role": "system",
        "content": ""
      },
      {
        "role": "user",
        "content": "Say \"Hello World!\""
      }
    ],
    "n": 1,
    "temperature": 0.0,
    "top_p": 1.0,
    "frequency_penalty": 0.0,
    "presence_penalty": 0.0,
    "stop": null
  }
}
```

and the response log file would look something like the following:

`./data/responses/1693765467279010000_a7bfa298-4bab-41fc-b63d-315f442c8443.json`
```json
{
  "system_prompt": "",
  "user_prompt": "Say \"Hello World!\"",
  "request": {
    "model": "gpt-4",
    "messages": [
      {
        "role": "system",
        "content": ""
      },
      {
        "role": "user",
        "content": "Say \"Hello World!\""
      }
    ],
    "n": 1,
    "temperature": 0.0,
    "top_p": 1.0,
    "frequency_penalty": 0.0,
    "presence_penalty": 0.0,
    "stop": null
  },
  "response": {
    "id": "chatcmpl-aaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "object": "chat.completion",
    "created": 1693765467,
    "model": "gpt-4-0613",
    "choices": [
      {
        "index": 0,
        "message": {
          "role": "assistant",
          "content": "Hello World!"
        },
        "finish_reason": "stop"
      }
    ],
    "usage": {
      "prompt_tokens": 16,
      "completion_tokens": 3,
      "total_tokens": 19
    }
  },
  "response_text": "Hello World!",
  "id": "chatcmpl-aaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```
