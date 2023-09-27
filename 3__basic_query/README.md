# Basic queries

One of the common tasks that LLMs can be useful for is doing some kind of processing on user-supplied text. There are a variety of operations we might want to do to it, including various types of summarization, but we're going to do sentiment analysis, a basic building block of analytics.

Our inputs are formatted like this: ([full input file](/input.json))

```json
  {
    "review_id_hash": "8877736525a297fa057e5268dfecc7b2",
    "text": "Best place to get eggs cheap at the moment!!!"
  },
```

Our **system prompt** will be

`"You are an assistant that judges the sentiment of customer reviews. Read the review and output 'positive', 'neutral', or 'negative'."`

and our **user prompt** will be just the text of the review.

So a payload for the API call might look something like this:

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are an assistant that judges the sentiment of customer reviews. Read the review and output 'positive', 'neutral', or 'negative'."},
    {"role": "user", "content": "Best place to get eggs cheap at the moment!!!"},
  ],
  "n": 1,
  "temperature": 0.0,
  "top_p": 1.0,
  "frequency_penalty": 0.0,
  "presence_penalty": 0.0,
  "stop": null,
  "max_tokens": 20,
}
```

Note the reduced `max_tokens` value. We're looking for a very short response, and don't want to waste money or time if the model decides to return an (invalid) longer response for some reason.

When we make this API call and insert the result, we have something like this: ([full output file](/output.json))

```json
  {
    "review_id_hash": "8877736525a297fa057e5268dfecc7b2",
    "text": "Best place to get eggs cheap at the moment!!!",
    "sentiment": "Positive"
  },
```

Next we will look at how to evaluate our output quality.
