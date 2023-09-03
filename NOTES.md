# Skills

Semantic Kernel uses *skills* that define named *functions*. These functions communicate using text - the basic building block of large language models. Functions can take a `string` as a parameter, return a `string` or both. You can then chain these functions to build a pipeline.

The `ConsoleSkill` implements 3 semantic kernel functions:

```csharp
[SKFunction("Get console input.")]
[SKFunctionName("Listen")]
public Task<string> Listen(SKContext context);

[SKFunction("Write a response to the console.")]
[SKFunctionName("Respond")]
public Task<string> Respond(string message, SKContext context);

[SKFunction("Did the user say goodbye.")]
[SKFunctionName("IsGoodbye")]
public Task<string> IsGoodbye(SKContext context);
```

The `Listen` function takes input from the console, the `Respond` function writes a response to the console (and returns it), and the `IsGoodbye` function returns if the `Listen` function received "goodbye" as it's input.

The `ChatSkill` implements 2 functions:

```csharp
[SKFunction("Send a prompt to the LLM.")]
[SKFunctionName("Prompt")]
public async Task<string> Prompt(string prompt);

[SKFunction("Log the history of the chat with the LLM.")]
[SKFunctionName("LogChatHistory")]
public Task LogChatHistory();
```

The `Prompt` function sends the given prompt to the LLM and returns the response. This uses the `IChatCompletion` service from semantic kernel. This is a service that has methods to create and manage chats. You can create a chat with a system prompt that gives the LLM background information to use when crafting responses. For example:

_You are a ... assistant who is good at .... Your name is [...](https://...)._

This prompt is set in the `configuration.json` file, so can be changed to suit your needs. This prompt sets up all chats, so if you were to ask:

```output
What is your name?
```

You would get the response:

```output
My name is .... How can I assist you today?
```

# Creating functions from prompts

An example function that converts text to poetry... functions do not need to be built in code, but can be created using a text prompt. The poetry function is created with the following code:

```csharp
string poemPrompt = """
  Take this "{{$INPUT}}" and convert it to a poem in iambic pentameter.
  """;

_poemFunction = _semanticKernel.CreateSemanticFunction(poemPrompt, maxTokens: openAIOptions.Value.MaxTokens,
    temperature: openAIOptions.Value.Temperature, frequencyPenalty: openAIOptions.Value.FrequencyPenalty,
    presencePenalty: openAIOptions.Value.PresencePenalty, topP: openAIOptions.Value.TopP);
```

Createing the function using the prompt _Take this "{{$INPUT}}" and convert it to a poem in iambic pentameter._. The function that is created with the call to `CreateSemanticFunction` is a function that takes a string as input, replaces the `{{$INPUT}}` field in the text with that string, sends it to the text completion service, and returns the result. This allows you to quickly create libraries of standard prompts in text that can be used in pipelines to process data.

# Pipelines

Semantic Kernel functions take and return text, so you can chain them together into pipelines. For example, chaining the `Listen`, `Prompt` and `Respond` functions to create a chatbot.

```csharp
await _semanticKernel.RunAsync(_speechSkill["Listen"], _chatSkill["Prompt"], _speechSkill["Respond"]);
```

This works as long as any function returns the right input for the next function in the pipeline. For example, the `Listen` function returns a `string`, which is the input for the `Prompt` function, which in turn returns a `string`, which is the input for the `Respond` function.

If you wanted to add more functions to the pipeline you can, for example inserting the `_poemFunction` mentioned above before the call to the `Respond` function to get the results in poetry.

These pipelines can be defined in code, so can be constructed on the fly if needed.
