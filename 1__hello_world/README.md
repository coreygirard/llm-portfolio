# LLM 'Hello World'

I plan for this set of posts to be fairly light on code, instead focusing on the prompts, the tradeoffs, and the processes. But let's do an initial Hello World to illustrate the basic scaffolding everything will hang onto later.

```python
import json
from pprint import pprint
import openai

with open("tokens.json", "r") as f:
    openai.api_key = json.load(f)["OPENAI_API_KEY"]


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


def request_chatcompletion(params):
    response = openai.ChatCompletion.create(**params)
    response = json.loads(json.dumps(response))  # TODO: hackjob
    return response


if __name__ == "__main__":
    user_prompt = 'Say "Hello World!"'
    payload = make_payload(system_prompt="", user_prompt=user_prompt)
    resp = request_chatcompletion(payload)
    output = resp["choices"][0]["message"]["content"]
    print(output)
```

Format of `tokens.json` (replace with your own OpenAI API key):

```json
{
  "OPENAI_API_KEY": "sk-aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```

Output (probably):

```
Hello World!
```
