---
layout: post
title: "Identifying Pony Alpha"
date: 2026-02-08
---

There's a new stealth model on Openrouter! Pony Alpha! 

What Openrouter say about it:

> Pony Alpha
> openrouter/pony-alpha
>
> Created Feb 6, 2026
> 200,000 context
> Pony is a cutting-edge foundation model with strong performance in coding, agentic workflows, reasoning, and roleplay, making it well suited for hands-on coding and real-world use.

I have seen posts online that GLM-5 would be coming in Feb, so I wanted to confirm if Pony Alpha was GLM-5 or not.

## Likely Models

Given that OpenAI just dropped GPT-5.3-codex and Anthropic dropped Opus 4.6, it's unlikely to be another model from them. GLM-5 has been mentioned to be releasing in February, so that needs to be considered as an option. There's also Qwen and Deepseek who haven't released something "big" in a little while. 

One thing that model providers can't hide easily, and don't tend to change frequently is their tokenizer. So I resolved to quickly check if the tokenizer used by Pony Alpha matched any of the usual suspects.

## Identifying Tokenizers 

How to identify tokenizers? We can use the quirks of them to tell them apart, by counting the number of tokens in our requests and responses.

### Token Counting 

Each tokenizer will have different tokens for common words or parts of words. We can exploit this with the `token count` of request/responses, by using words which different tokenizers will treat differently.

**Example:**

If two tokenizers treat a word like "elephant" differently, we can use the different token counts to identify the tokenizers.

In our hypothetical example, tokenizer 1 has a unique token for "elephant". tokenizer 2 does not, so it will see "elephant" as "ele", "p", "h", "ant".

If we send a simple prompt like "hello", we get a baseline token count.

If we then send the same prompt "hello elephant", we get the same baseline token count _plus the number of tokens that elephant uses_.

if the additional token count is 1, it is using tokenizer 1. 

If the additional token count is 4, the model is using tokenizer 2.

Because requests that we send to the model will have system prompts etc added, we can't use the _direct count_ of the tokens used. We have to instead use the _difference in token count between two different prompts_. 

For example, if we prompt with "Hello" like before, we might get a token count of "2014" - this will include system prompts and other info the model is sent, plus our prompt. When we later send "Hello elephant", and get a token count of "2018", we still get the signal we are looking for -- "elephant" is taking 4 tokens, so it's using tokenizer 2.

## Identifying a Tokenizer

So we can use the token counts to tell tokenizers apart, but how do we work out if the tokenizer we are talking to is "a particular tokenizer"?

We need some additional information, like the characteristics of different known tokenizers.

Luckily, this is available.

### Local Tokenizer Comparison

Huggingface provides implementations of common tokenizers! So we can tokenize some test prompts locally with tokenizers from the likely model providers, get the token count, and then compare with the token counts that the API provides.

This will tell us if the reference implementation(s) match the token count from the API, which could be a strong signal they're using the same tokenizer.

### Special tokens 

Each tokenizer uses special keywords (known as "special tokens") which the control structures or the models learn to use to control their processing. Each tokenizer does this slightly different, for example ChatML-style models will use `<|endoftext|>` or `<|im_start|>`, whereas GLM models will use `[gMASK]<sop>` as special tokens. 

Because the special tokens are unique to a tokenizer, they will count as length `1` where the tokenizer "knows" them, and many more where the tokenizer does not know them.

These are high value for determining which tokenizer you're talking to.

### Other common tokenizer differences

Each tokenizer tends to have "tells", things which will be tokenized differently by different tokenizers.

For example:

- different languages ( English, Cyrillic, CJK.. )
- numbers ( single digit, multiple digit, common number combinations .. )
- repeating characters ( "aaaaaa" vs "aaaaaaaaa" or "\n\n\n\n" .. )
- leading whitespace ( " The" vs "  The" .. )
- URLs ( "https://www.example.com/path?query=value&foo=bar" could be represented differently across tokenizers)
- long words ( "indivisibility" can be split different ways )

may be handled differently by different tokenizers. We can use the differences to fingerprint the tokenizers.

### Limitations 

These techniques will help us identify a "tokenizer family", not a particular _model_. 

In practice this won't matter for our purposes, because if we know the most likely model provider, we can guess at the model name. 

It also won't help us tell if the model is "GLM-4.8" or "GLM-5", those two (hypothetical at this time!) models would likely both use the same tokenizer. But it _will_ let us rule out tokenizers which don't match the patterns of already known tokenizers.

Model providers tend to stick to a tokenizer for a long time, and they tend to have their own unique tokenizer.

## Pulling it all together

We can create a simple probe script which will use the API to talk to Pony Alpha, sending test prompts and using the token counts to fingerprint the model, and compare against local tokenizer implementations.

After doing that, here's the results! 


| String | API | qwen2.5 | deepseek-v3 | glm4 | llama3 |
|--------|-----|---------|-------------|------|--------|
| `Hello, world!` | 4 | 4 ✓ | 4 ✓ | 4 ✓ | 4 ✓ |
| `    def hello_world():\n...` | 9 | 9 ✓ | 10 | 9 ✓ | 9 ✓ |
| `🦙🐴🦄` | 7 | 3 | 8 | 9 | 9 |
| `123456789012345` | 9 | 15 | 5 | **9 ✓** | 5 |
| `中文测试文本示例` | 4 | 5 | 4 ✓ | **4 ✓** | 6 |
| `Привет мир` | 3 | 4 | 3 ✓ | **3 ✓** | 4 |
| `indivisibility` | 4 | 4 ✓ | 4 ✓ | 4 ✓ | 4 ✓ |
| `https://www.example.com/...` | 13 | 13 ✓ | 15 | **13 ✓** | 13 ✓ |
| `<&#124;endoftext&#124;>` | 1 | 1 ✓ | 7 | **1 ✓** | 7 |
| `<&#124;im_start&#124;>system` | 7 | 2 | 7 ✓ | **7 ✓** | 7 ✓ |
| `\n\n\n\n\n` | 1 | 1 ✓ | 1 ✓ | 1 ✓ | 1 ✓ |
| `aaaaaaaaaa` | 2 | 2 ✓ | 2 ✓ | 2 ✓ | 2 ✓ |
| `大语言模型是人工智能的重要突破` | 7 | 7 ✓ | 7 ✓ | **7 ✓** | 11 |
| `def fibonacci(n):...` | 14 | 14 ✓ | 14 ✓ | 14 ✓ | 14 ✓ |
| `The quick brown fox...` | 9 | 9 ✓ | 9 ✓ | 9 ✓ | 9 ✓ |
| `<｜begin▁of▁sentence｜>` | 11 | 11 ✓ | 1 | 11 ✓ | 11 ✓ |
| `[gMASK]<sop>` | 2 | 6 | 8 | **2 ✓** | 6 |

### Match Scores

| Candidate | Exact Matches |
|-----------|---------------|
| **glm4** | **16/17** |
| qwen2.5 | 11/17 |
| deepseek-v3 | 10/17 |
| llama3 | 10/17 |

The key findings there are some special tokens:


- **`[gMASK]<sop>`** — 2 tokens for Pony Alpha (and GLM-4 tokenizer)
- **`<|endoftext|>`** — 1 token for Pony Alpha (and GLM/Qwen tokenizers)
- **`<|im_start|>system`** — 7 tokens for Pony Alpha. (same as GLM tokenizer, and which Qwen would count as 2)

The "sequence of numbers" test also reveals some useful detail:

- **`123456789012345`** — 9 tokens for Pony Alpha (matching only GLM-4) 

The one test that doesn't match with the GLM-4 tokenizer is an emoji string (`🦙🐴🦄`) which Pony Alpha uses 7 tokens to represent, but GLM-4 would take 9 tokens -- this could mean that there has been a small update to the tokenizer that is being used. This actually supports the idea that this model is a _newer model_ rather than just a _more trained_ version of GLM-4.

## Summary

With a high degree of confidence we can say that "Pony Alpha uses a tokenizer very similar to GLM-4".

This means it's very likely that the model is a _new model from Zhipu AI_, probably a new GLM model.

Given the pre-announcements that GLM-5 is coming, and the reports from people online that the Pony Alpha outputs have been high quality, it's likely Pony Alpha is the upcoming GLM-5.
