---
author: alvinashcraft
title: Get started with AI Toolkit for Visual Studio Code
description: Learn to use the AI Toolkit for VS Code to download, test, fine-tune, and deploy AI models with your apps or in the cloud.
ms.author: aashcraft
ms.date: 09/11/2024
ms.topic: how-to
no-loc: [VS Code, Visual Studio Code, REST]
#customer intent: As a Windows developer, I want to learn how to download and run AI models locally using the AI Toolkit for Visual Studio Code.
---

# Get started with AI Toolkit for Visual Studio Code

The AI Toolkit for VS Code (AI Toolkit) is a VS Code extension that enables you to download, test, fine-tune, and deploy AI models with your apps or in the cloud. For more information, see the [AI Toolkit overview](index.md).

In this article, you'll learn how to:

> [!div class="checklist"]
> - Install the AI Toolkit for VS Code
> - Download a model from the catalog
> - Run the model locally using the playground
> - Integrate an AI model into your application using REST or the ONNX Runtime

## Prerequisites

- **VS Code** must be installed. For more information, see [Download VS Code](https://code.visualstudio.com/download) and [Getting started with VS Code](https://code.visualstudio.com/docs/introvideos/basics).

## Install

The [AI Toolkit is available in the Visual Studio Marketplace](https://aka.ms/aitoolkit) and can be installed like any other VS Code extension. If you're unfamiliar with installing VS Code extensions, follow these steps:

1. In the Activity Bar in VS Code select **Extensions**
1. In the Extensions Search bar type "AI Toolkit"
1. Select the "AI Toolkit for Visual Studio code"
1. Select **Install**

Once the extension has been installed you'll see the AI Toolkit icon appear in your Activity Bar.

## Download a model from the catalog

The primary sidebar of the AI Toolkit is organized into **Models** and **Resources**. The **Playground** and **Fine-tuning** features are available in the Resources section. To get started select **Model Catalog**:

:::image type="content" source="../images/toolkit-getting-started/model_catalog.png" alt-text="AI Toolkit model catalog":::

> [!TIP]
> You'll notice that the model cards show the model size, the platform and accelerator type (CPU, GPU). For optimized performance on **Windows devices that have at least one GPU**, select model versions that only target Windows. This ensures you have a model optimized for the [DirectML](../directml/dml-intro.md) accelerator. The model names are in the format of `{model_name}-{accelerator}-{quantization}-{format}`.
>
> To check whether you have a GPU on your Windows device, open **Task Manager** and then select the **Performance** tab. If you have GPU(s), they will be listed under names like "GPU 0" or "GPU 1".

Next, download the following model depending on the availability of a GPU on your device.

| Platform(s) | GPU available | Model name | Size (GB) |
|---------|---------|--------|--------|
| Windows | Yes | Phi-3-mini-4k-**directml**-int4-awq-block-128-onnx | 2.13GB |
| Linux | Yes | Phi-3-mini-4k-**cuda**-int4-onnx | 2.30GB |
| Windows<br>Mac<br>Linux | No | Phi-3-mini-4k-**cpu**-int4-rtn-block-32-acc-level-4-onnx | 2.72GB |

> [!NOTE]
> The Phi3-mini (int4) model is approximately 2GB-3GB in size. Depending on your network speed, it could take a few minutes to download.

## Run the model in the playground

Once your model has downloaded, select **Load in Playground** on the model card in the catalog:

:::image type="content" source="../images/toolkit-getting-started/load_model_into_playground.png" alt-text="Load into playground":::

In the chat interface of the playground enter the following message followed by the **Enter** key:

:::image type="content" source="../images/toolkit-getting-started/message-box.png" alt-text="Playground selection":::

You should see the model response streamed back to you:

:::image type="content" source="../images/toolkit-getting-started/generation-gif.gif" alt-text="Generation response":::

> [!WARNING]
> If you do **not** have a **GPU** available on your device but you selected the Phi-3-mini-4k-**directml**-int4-awq-block-128-onnx model, the model response will be *very slow*. You should instead download the CPU optimized version: Phi-3-mini-4k-**cpu**-int4-rtn-block-32-acc-level-4-onnx.

It is also possible to change:

- **Context Instructions:** Help the model understand the bigger picture of your request. This could be background information, examples/demonstrations of what you want or explaining the purpose of your task.
- **Inference parameters:**
  - *Maximum response length*: The maximum number of tokens the model will return.
  - *Temperature*: Model temperature is a parameter that controls how random a language model's output is. A higher temperature means the model takes more risks, giving you a diverse mix of words. On the other hand, a lower temperature makes the model play it safe, sticking to more focused and predictable responses.
  - *Top P*: Also known as nucleus sampling, is a setting that controls how many possible words or phrases the language model considers when predicting the next word
  - *Frequency penalty*: This parameter influences how often the model repeats words or phrases in its output. The higher the value (closer to 1.0) encourages the model to *avoid* repeating words or phrases.
  - *Presence penalty*: This parameter is used in generative AI models to encourage diversity and specificity in the generated text. A higher value (closer to 1.0) encourages the model to include more novel and diverse tokens. A lower value is more likely for the model to generate common or cliche phrases.

## Integrate an AI model into your application

There are two options to integrate the model into your application:

1. The AI Toolkit comes with a *local REST API web server* that uses the [OpenAI chat completions format](https://platform.openai.com/docs/api-reference/chat/create). This enables you to test your application locally - using the endpoint `http://127.0.0.1:5272/v1/chat/completions` - without having to rely on a cloud AI model service. Use this option if you intend to switch to a cloud endpoint in production. You can use OpenAI client libraries to connect to the web server.
1. Using the *ONNX Runtime*. Use this option if you intend to ship the model *with* your application with inferencing on device.

### Local REST API web server

The local REST API web server allows you to build-and-test your application locally without having to rely on a cloud AI model service. You can interact with the web server using REST, or with an OpenAI client library:

# [REST](#tab/rest)
Here is an example body for your REST request:

```json
{
    "model": "Phi-3-mini-4k-directml-int4-awq-block-128-onnx",
    "messages": [
        {
            "role": "user",
            "content": "what is the golden ratio?"
        }
    ],
    "temperature": 0.7,
    "top_p": 1,
    "top_k": 10,
    "max_tokens": 100,
    "stream": true
}'
```

> [!NOTE]
> You may need to update the model field to the name of the model you downloaded.

You can test the REST endpoint using an API tool like [Postman](https://www.postman.com/) or the CURL utility:

```bash
curl -vX POST http://127.0.0.1:5272/v1/chat/completions -H 'Content-Type: application/json' -d @body.json
```

# [Python](#tab/python)

Install the OpenAI Python library:

```bash
pip install openai
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:5272/v1/",
    api_key="x" # required by API but not used
)

chat_completion = client.chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "what is the golden ratio?",
        }
    ],
    model="Phi-3-mini-4k-directml-int4-awq-block-128-onnx",
)

print(chat_completion.choices[0].message.content)
```

# [C#](#tab/csharp)

Add the [Azure OpenAI client library for .NET](https://www.nuget.org/packages/Azure.AI.OpenAI/) to your project using NuGet:

```bash
dotnet add {project_name} package Azure.AI.OpenAI --version 1.0.0-beta.17
```

Add a C# file called **OverridePolicy.cs** to your project and paste the following code:

```csharp
// OverridePolicy.cs
using Azure.Core.Pipeline;
using Azure.Core;

internal partial class OverrideRequestUriPolicy(Uri overrideUri)
    : HttpPipelineSynchronousPolicy
{
    private readonly Uri _overrideUri = overrideUri;

    public override void OnSendingRequest(HttpMessage message)
    {
        message.Request.Uri.Reset(_overrideUri);
    }
}
```

Next, paste the following code into your **Program.cs** file:

```csharp
// Program.cs
using Azure.AI.OpenAI;

Uri localhostUri = new("http://localhost:5272/v1/chat/completions");

OpenAIClientOptions clientOptions = new();
clientOptions.AddPolicy(
    new OverrideRequestUriPolicy(localhostUri),
    Azure.Core.HttpPipelinePosition.BeforeTransport);
OpenAIClient client = new(openAIApiKey: "unused", clientOptions);

ChatCompletionsOptions options = new()
{
    DeploymentName = "Phi-3-mini-4k-directml-int4-awq-block-128-onnx",
    Messages =
    {
        new ChatRequestSystemMessage("You are a helpful assistant. Be brief and succinct."),
        new ChatRequestUserMessage("What is the golden ratio?"),
    }
};

StreamingResponse<StreamingChatCompletionsUpdate> streamingChatResponse
    = await client.GetChatCompletionsStreamingAsync(options);

await foreach (StreamingChatCompletionsUpdate chatChunk in streamingChatResponse)
{
    Console.Write(chatChunk.ContentUpdate);
}
```

> [!NOTE]
> If you downloaded the CPU version of the Phi3 model, you need to update the model field to Phi-3-mini-4k-cpu-int4-rtn-block-32-acc-level-4-onnx.

---

### ONNX Runtime

The [ONNX Runtime Generate API](https://onnxruntime.ai/docs/genai/) provides the generative AI loop for ONNX models, including inference with ONNX Runtime, logits processing, search and sampling, and KV cache management. You can call a high level `generate()` method, or run each iteration of the model in a loop, generating one token at a time, and optionally updating generation parameters inside the loop.

It has support for greedy/beam search and TopP, TopK sampling to generate token sequences and built-in logits processing like repetition penalties. The following code is an example of how you can leverage the ONNX runtime in your applications.

# [REST](#tab/rest)

Please refer to the example shown in [Local REST API web server](#local-rest-api-web-server). The AI Toolkit REST web server is built using the ONNX Runtime.

# [Python](#tab/python)

Install Numpy:

```bash
pip install numpy
```

Next, install the ONNX Runtime Python package into your project according to your platform and GPU availability:

| Platform | GPU Available | PyPI |
| ---------| ------------- | -------------------|
| Windows  | Yes<br>(AMD, NVIDIA, Intel, Qualcomm, plus others supported)           | `pip install --pre onnxruntime-genai-directml`  |
| Linux    | Yes<br>(Nvidia CUDA)          | `pip install --pre onnxruntime-genai-cuda --index-url=https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-genai/pypi/simple/` |
| Windows<br>Linux    | No           | `pip install --pre onnxruntime-genai` |

> [!TIP]
> We recommend installing Python packages into a virtual environment using either **venv** or **conda**.

Next, copy-and-paste the following code into a Python file named **app.py**:

```python
# app.py
import onnxruntime_genai as og
import argparse

def main(args):
    print("Loading model...")
    model = og.Model(f'{args.model}')
    print("Model loaded")
    tokenizer = og.Tokenizer(model)
    tokenizer_stream = tokenizer.create_stream()
    search_options = {
        'max_length': 2048
    }

    chat_template = '<|user|>\n{input} <|end|>\n<|assistant|>'

    # Keep asking for input prompts in a loop
    while True:
        text = input("Input: ")
    
        # If there is a chat template, use it
        prompt = f'{chat_template.format(input=text)}'

        input_tokens = tokenizer.encode(prompt)

        params = og.GeneratorParams(model)
        params.set_search_options(**search_options)
        params.input_ids = input_tokens
        
        generator = og.Generator(model, params)
        print("\nOutput: ", end='', flush=True)
        while not generator.is_done():
            generator.compute_logits()
            generator.generate_next_token()
            new_token = generator.get_next_tokens()[0]
            print(tokenizer_stream.decode(new_token), end='', flush=True)
              
        print()
        print()

        # Delete the generator to free the captured graph for the next generator, if graph capture is enabled
        del generator


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('-m', '--model', type=str, required=True, help='Onnx model folder path (must contain config.json and model.onnx)')
    args = parser.parse_args()
    main(args)
```

To run the Python app use the following code:

```bash
python app.py --model ~/.aitk/models/{path_to_folder_containing_onnx_file}
```

> [!NOTE]
> The AI Toolkit caches model downloads into a hidden folder named `.aitk` in your user directory - you'll need to update the path used for the `--model` parameter to the location of the *folder* containing the ONNX model file. For example `~/.aitk/models/microsoft/Phi-3-mini-4k-instruct-onnx/directml/Phi-3-mini-4k-directml-int4-awq-block-128-onnx/`

# [C#](#tab/csharp)

Install the ONNX Runtime NuGet package into your project according to your platform and GPU availability:

| Platform | GPU Available | Nuget |
| ---------| ------------- | -------------------|
| Windows  | Yes<br>(AMD, NVIDIA, Intel, Qualcomm, plus others supported)           | [Microsoft.ML.OnnxRuntimeGenAI.DirectML](https://www.nuget.org/packages/Microsoft.ML.OnnxRuntimeGenAI.DirectML/) |
| Linux    | Yes<br>(Nvidia CUDA)          | [Microsoft.ML.OnnxRuntimeGenAI.Cuda](https://www.nuget.org/packages/Microsoft.ML.OnnxRuntimeGenAI.Cuda/) |
| Windows<br>Linux    | No           | [Microsoft.ML.OnnxRuntimeGenAI ](https://www.nuget.org/packages/Microsoft.ML.OnnxRuntimeGenAI/) |

Copy-and-paste the following code into your C# file:

```csharp
using Microsoft.ML.OnnxRuntimeGenAI;

// update user_name and path placeholders
string modelPath = "C:\\Users\\{user_name}\\.aitk\\models\\{path}"; 
Console.Write("Loading model from " + modelPath + "...");
using Model model = new(modelPath);
Console.Write("Done\n");
using Tokenizer tokenizer = new(model);
using TokenizerStream tokenizerStream = tokenizer.CreateStream();

while (true)
{
    Console.Write("User:");
   
    string? input = Console.ReadLine();
    string prompt = "<|user|>\n" + input + "<|end|>\n<|assistant|>";

    var sequences = tokenizer.Encode(prompt);

    using GeneratorParams generatorParams = new GeneratorParams(model);
    generatorParams.SetSearchOption("max_length", 200);
    generatorParams.SetInputSequences(sequences);

    Console.Out.Write("\nAI:");
    using Generator generator = new(model, generatorParams);
    while (!generator.IsDone())
    { 
        generator.ComputeLogits();
        generator.GenerateNextToken();
        Console.Out.Write(tokenizerStream.Decode(generator.GetSequence(0)[^1]));
        Console.Out.Flush();
    }
    Console.WriteLine();
}
```

> [!NOTE]
> The AI Toolkit caches model downloads into a hidden folder named `.aitk` in your user directory - you'll need to update the `modelPath` in the code to the location of the *folder* containing the ONNX model file. For example `~/.aitk/models/microsoft/Phi-3-mini-4k-instruct-onnx/directml/Phi-3-mini-4k-directml-int4-awq-block-128-onnx/`

---

## Next Step

> [!div class="nextstepaction"]
> [Fine-tune a model with AI Toolkit for VS Code](toolkit-fine-tune.md)
