# Agents


Eino is a modular, developer‑friendly framework for building AI agents and  
autonomous workflows. It focuses on clarity, composability, and practical  
engineering patterns that make it easier to design, test, and deploy systems  
powered by large language models. Instead of locking developers into a rigid  
agent paradigm, Eino provides a flexible toolkit that encourages assembling  
agents from small, well‑defined components. This approach supports both simple  
single‑step interactions and complex multi‑stage reasoning pipelines.  

Eino’s design philosophy centers on three principles. First, **explicitness**,  
ensuring that agent behavior, tool usage, and decision paths remain  
transparent. Second, **flexibility**, allowing developers to mix different  
models, tools, memory stores, and execution strategies without friction.  
Third, **practicality**, emphasizing real‑world usability, predictable  
behavior, and maintainability over theoretical complexity.  

Because of this, Eino is well suited for a wide range of applications,  
including conversational assistants, data‑processing pipelines, retrieval‑  
augmented generation systems, and orchestrated multi‑agent workflows. Its  
architecture encourages developers to think in terms of capabilities rather  
than monolithic agents, making it easier to scale prototypes into production  
systems.  

## Architecture Overview  

Eino’s architecture is built around a set of composable building blocks that  
work together to form an agent or workflow. Each block has a clear purpose,  
and the framework allows developers to replace or extend any component  
without disrupting the overall design.  

### 1. Models  

Models represent the reasoning engine of an agent. Eino supports multiple  
model backends, enabling developers to choose between local models, cloud  
APIs, or hybrid setups. Models are treated as interchangeable components,  
allowing experimentation with different capabilities or cost profiles.  

### 2. Tools  

Tools are external functions or services that an agent can call to perform  
actions beyond pure text generation. Examples include database queries, API  
calls, file operations, or domain‑specific utilities. Eino provides a clean  
interface for defining tools, validating inputs, and integrating them into  
the agent’s reasoning loop.  

### 3. Memory  

Memory components allow agents to store and retrieve information across  
interactions. Eino supports short‑term scratchpads, long‑term vector stores,  
and structured state objects. This enables agents to maintain context, track  
progress, and build persistent knowledge over time.  

### 4. Execution Graphs  

Execution graphs define how reasoning steps, tool calls, and decision points  
are connected. Instead of relying on opaque chains of prompts, Eino models  
agent behavior as an explicit graph of nodes and transitions. This makes  
complex workflows easier to debug, visualize, and extend.  

### 5. Runners  

Runners orchestrate the execution of an agent or workflow. They manage  
control flow, handle retries, enforce constraints, and coordinate the  
interaction between models, tools, and memory. Runners can be configured for  
single‑step execution, iterative reasoning, or fully autonomous operation.  

### 6. Observability Layer  

Eino includes hooks for logging, tracing, and introspection. Developers can  
inspect intermediate reasoning steps, tool inputs and outputs, and decision  
paths. This observability layer is essential for debugging and for ensuring  
that agents behave predictably in production environments.  

## Putting It All Together  

By combining these components, developers can construct agents that reason  
effectively, interact with external systems, and adapt to changing  
requirements. Eino’s architecture encourages experimentation while providing  
a stable foundation for long‑term maintainability. Whether you are building a  
simple assistant or a complex multi‑agent system, Eino offers the structure  
and flexibility needed to move from prototype to production with confidence.  

  

If you want, I can also generate:  
- a **diagram‑style architecture summary**,  
- a **short executive‑summary version**,  
- or a **README‑ready introduction** tailored to your repository.

## Simple message

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	"github.com/cloudwego/eino/adk"
	"github.com/cloudwego/eino/schema"

	"github.com/cloudwego/eino-ext/components/model/openai"
)

func main() {
	ctx := context.Background()

	// Step 1: Create a chat model backed by DeepSeek V4 Flash.
	// DeepSeek exposes an OpenAI-compatible API, so we use the OpenAI adapter.
	// Set the DEEPSEEK_API_KEY environment variable before running.
	model, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
		APIKey:  os.Getenv("DEEPSEEK_API_KEY"), // required: your DeepSeek API key
		Model:   "deepseek-v4-flash",           // the DeepSeek model ID
		BaseURL: "https://api.deepseek.com/v1", // OpenAI-compatible endpoint
	})
	if err != nil {
		log.Fatal(err)
	}

	// Step 2: Wrap the model in an ADK (Agent Development Kit) ChatModelAgent.
	// The agent combines the model with a name, description, and system instruction.
	agent, err := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
		Name:        "hello_agent",
		Description: "A friendly greeting assistant",
		Instruction: "You are a friendly assistant. Please respond to the user in a warm tone.",
		Model:       model,
	})
	if err != nil {
		log.Fatal(err)
	}

	// Step 3: Create a Runner that orchestrates the agent execution.
	// EnableStreaming=true delivers partial tokens as they are generated.
	runner := adk.NewRunner(ctx, adk.RunnerConfig{
		Agent:           agent,
		EnableStreaming: true,
	})

	// Step 4: Define the conversation input — one or more messages to send to the agent.
	input := []adk.Message{
		schema.UserMessage("Explain AI agents in a paragraph."),
	}

	// Step 5: Run the agent and consume the stream of events.
	// Each event may contain a partial or full message output.
	events := runner.Run(ctx, input)
	for {
		event, ok := events.Next()
		if !ok {
			break
		}

		if event.Err != nil {
			log.Printf("error: %v", event.Err)
			break
		}

		if msg, err := event.Output.MessageOutput.GetMessage(); err == nil {
			fmt.Printf("Agent: %s\n", msg.Content)
		}
	}
}
```
