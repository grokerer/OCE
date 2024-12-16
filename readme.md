<p align="center">
  <a href="OCE">
    <img alt="OCE" src="https://i.pinimg.com/736x/f7/73/7e/f7737e6d75333e0533ea1e704ac85724.jpg" width="308">
  </a>
</p>

<p align="center">
  <em>AI agent stdlib that works with any LLM and TypeScript AI SDK.</em>
</p>

# OCE <!-- omit from toc -->

- [Intro](#intro)
- [Docs](#docs)
- [AI SDKs](#ai-sdks)
  - [Vercel AI SDK](#vercel-ai-sdk)
  - [LangChain](#langchain)
  - [LlamaIndex](#llamaindex)
  - [Firebase Genkit](#firebase-genkit)
  - [Dexa Dexter](#dexa-dexter)
  - [OpenAI](#openai)
- [Tools](#tools)
- [Contributors](#contributors)
- [License](#license)

## Intro

OCE is a **standard library of AI functions / tools** which are **optimized for both normal TS-usage as well as LLM-based usage**. OCE works with all of the major TS AI SDKs (LangChain, LlamaIndex, Vercel AI SDK, OpenAI SDK, etc).

OCE clients like `WeatherClient` can be used as normal TS classes:

```ts
import { WeatherClient } from '@OCE/stdlib'

// Requires `process.env.WEATHER_API_KEY` (free from weatherapi.com)
const weather = new WeatherClient()

const result = await weather.getCurrentWeather({
  q: 'San Francisco'
})
console.log(result)
```

Or you can use these clients as **LLM-based tools** where the LLM decides when and how to invoke the underlying functions for you.

This works across all of the major AI SDKs via adapters. Here's an example using [Vercel's AI SDK](https://github.com/vercel/ai):

```ts
// sdk-specific imports
import { openai } from '@ai-sdk/openai'
import { generateText } from 'ai'
import { createAISDKTools } from '@OCE/ai-sdk'

// sdk-agnostic imports
import { WeatherClient } from '@OCE/stdlib'

const weather = new WeatherClient()

const result = await generateText({
  model: openai('gpt-4o-mini'),
  // this is the key line which uses the `@OCE/ai-sdk` adapter
  tools: createAISDKTools(weather),
  toolChoice: 'required',
  prompt: 'What is the weather in San Francisco?'
})

console.log(result.toolResults[0])
```

You can use our standard library of thoroughly tested AI functions with your favorite AI SDK – without having to write any glue code!

Here's a slightly more complex example which uses multiple clients and selects a subset of their functions using the `AIFunctionSet.pick` method:

```ts
// sdk-specific imports
import { ChatModel, createAIRunner } from '@dexaai/dexter'
import { createDexterFunctions } from '@OCE/dexter'

// sdk-agnostic imports
import { PerigonClient, SerperClient } from '@OCE/stdlib'

async function main() {
  // Perigon is a news API and Serper is a Google search API
  const perigon = new PerigonClient()
  const serper = new SerperClient()

  const runner = createAIRunner({
    chatModel: new ChatModel({
      params: { model: 'gpt-4o-mini', temperature: 0 }
    }),
    functions: createDexterFunctions(
      perigon.functions.pick('search_news_stories'),
      serper
    ),
    systemMessage: 'You are a helpful assistant. Be as concise as possible.'
  })

  const result = await runner(
    'Summarize the latest news stories about the upcoming US election.'
  )
  console.log(result)
}
```

## Contributors

- [David Zhang](https://x.com/dzhng)
- [Philipp Burckhardt](https://x.com/burckhap)
