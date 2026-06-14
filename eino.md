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

## Templates

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"

	"github.com/cloudwego/eino/adk"
	"github.com/cloudwego/eino/components/prompt"
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

	// Step 2: Define a prompt template using FString syntax.
	// Variables like {user_name} and {current_date} are injected at runtime.
	chatTemplate := prompt.FromMessages(
		schema.FString,
		schema.SystemMessage("You are a friendly assistant speaking to {user_name}.\nToday's date is {current_date}. Keep your answers warm and concise."),
		schema.UserMessage("{question}"),
	)

	// Step 3: Prepare dynamic values for the template variables.
	// These could come from user input, a database, or any runtime source.
	vars := map[string]any{
		"user_name":    "Alice",
		"current_date": time.Now().Format("2006-01-02"),
		"question":     "What can you help me with today?",
	}

	// Step 4: Render the template into concrete messages.
	messages, err := chatTemplate.Format(ctx, vars)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Messages:")
	for _, msg := range messages {
		fmt.Printf("%s: %s\n", msg.Role, msg.Content)
	}

	// Step 5: Wrap the model in an ADK (Agent Development Kit) ChatModelAgent.
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

	// Step 6: Create a Runner that orchestrates the agent execution.
	// EnableStreaming=true delivers partial tokens as they are generated.
	runner := adk.NewRunner(ctx, adk.RunnerConfig{
		Agent:           agent,
		EnableStreaming: true,
	})

	// Step 7: Copy rendered messages into ADK input.
	input := make([]adk.Message, len(messages))
	copy(input, messages)

	// Step 8: Run the agent and consume the stream of events.
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

Jinja template 

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	"github.com/cloudwego/eino/adk"
	"github.com/cloudwego/eino/components/prompt"
	"github.com/cloudwego/eino/schema"

	"github.com/cloudwego/eino-ext/components/model/openai"
)

func main() {
	ctx := context.Background()

	// Step 1: Create the DeepSeek model (same as before).
	model, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
		APIKey:  os.Getenv("DEEPSEEK_API_KEY"),
		Model:   "deepseek-v4-flash",
		BaseURL: "https://api.deepseek.com/v1",
	})
	if err != nil {
		log.Fatal(err)
	}

	// Step 2: Define a prompt template using Jinja2 syntax.
	// Variables use {{double_curlies}} — perfect for multi-turn scenarios.
	// MessagesPlaceholder("chat_history", true) injects past messages if provided.
	chatTemplate := prompt.FromMessages(
		schema.Jinja2,
		schema.SystemMessage("You are a {{role}} expert. Answer in {{language}}."),
		schema.MessagesPlaceholder("chat_history", true),
		schema.UserMessage("{{query}}"),
	)

	// Step 3: Prepare variables — including a chat history with a prior exchange.
	vars := map[string]any{
		"role":     "cybersecurity",
		"language": "simple, plain English",
		"chat_history": []*schema.Message{
			schema.UserMessage("What is a phishing attack?"),
			schema.AssistantMessage("Phishing is a social engineering attack where attackers pose as trusted entities to steal sensitive data.", nil),
		},
		"query": "How can I recognize one?",
	}

	// Step 4: Render the template (Jinja2 engine handles substitution).
	messages, err := chatTemplate.Format(ctx, vars)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("=== Rendered Messages (Jinja2) ===")
	for i, msg := range messages {
		fmt.Printf("[%d] %s: %s\n", i+1, msg.Role, msg.Content)
	}

	// Step 5: Wrap in ADK agent (SystemMessage from template replaces Instruction).
	agent, err := adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
		Name:        "jinja2_agent",
		Description: "Demonstrates Jinja2 template injection",
		Model:       model,
	})
	if err != nil {
		log.Fatal(err)
	}

	// Step 6: Runner.
	runner := adk.NewRunner(ctx, adk.RunnerConfig{
		Agent:           agent,
		EnableStreaming: true,
	})

	// Step 7: Run.
	input := make([]adk.Message, len(messages))
	copy(input, messages)

	fmt.Println("\n=== Agent Response ===")
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

## Extract data

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"os"

	"github.com/cloudwego/eino-ext/components/model/openai"
	"github.com/cloudwego/eino/schema"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
	City string `json:"city"`
}

type PeopleExtraction struct {
	People []Person `json:"people"`
}

func main() {
	ctx := context.Background()

	apiKey := os.Getenv("DEEPSEEK_API_KEY")
	if apiKey == "" {
		log.Fatal("Error: DEEPSEEK_API_KEY environment variable is not set.")
	}

	chatModel, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
		APIKey:  apiKey,
		BaseURL: "https://api.deepseek.com",
		Model:   "deepseek-v4-flash",
		ResponseFormat: &openai.ChatCompletionResponseFormat{
			Type: openai.ChatCompletionResponseFormatTypeJSONObject,
		},
	})
	if err != nil {
		log.Fatalf("Failed to create chat model: %v", err)
	}

	text := `John Doe is a software engineer living in New York. He is 30 years old
and enjoys hiking and photography. Jane Smith is a graphic designer based in San
Francisco. She is 28 years old and loves painting and traveling.`

	systemPrompt := `You are a precise data extraction assistant. Always respond 
with valid JSON only — no markdown fences, no extra text.`

	userPrompt := fmt.Sprintf(
		"Extract all people from the following text. Return a JSON object with a 'people'\n"+
			"array, each entry having fields: name (string), age (number), city (string).\n\n"+
			"Text: %s", text)

	resp, err := chatModel.Generate(ctx, []*schema.Message{
		{
			Role:    schema.System,
			Content: systemPrompt,
		},
		{
			Role:    schema.User,
			Content: userPrompt,
		},
	})
	if err != nil {
		log.Fatalf("Generate failed: %v", err)
	}

	var result PeopleExtraction
	if err := json.Unmarshal([]byte(resp.Content), &result); err != nil {
		log.Fatalf("Failed to parse response: %v", err)
	}

	fmt.Printf("Extracted info: %+v\n", result)

	fmt.Println("\nPeople found:")
	for _, person := range result.People {
		fmt.Printf("- %s, %d years old, from %s\n", person.Name, person.Age, person.City)
	}
}
```

## Reasoning effort 


In this example, we demonstrate how to control the **reasoning effort** of an Eino-powered  chat model. Many modern LLM backends support configurable reasoning depth, giving developers  fine-grained control over the trade-off between response speed, token cost, and solution  quality. By adjusting the reasoning effort level, you can instruct the model to perform  shallow, fast reasoning for simple tasks — or deeper, more deliberate analysis for complex  problems that demand higher accuracy.

The example illustrates two complementary approaches: 

1. **Static configuration via `ChatModelConfig`** — set the effort level at model creation  time, so every call uses the same depth. 
2. **Per-request functional options** — create a single model instance and vary the effort  dynamically per `Generate()` call using `WithReasoningEffort` and `WithExtraFields`. 

Both approaches are demonstrated against DeepSeek V4 Flash, with the chain-of-thought  (`thinking`) mode enabled to surface the model's internal reasoning alongside the final  answer.

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"

	"github.com/cloudwego/eino-ext/components/model/openai"
	"github.com/cloudwego/eino/schema"
)

func main() {
	ctx := context.Background()

	apiKey := os.Getenv("DEEPSEEK_API_KEY")
	if apiKey == "" {
		log.Fatal("DEEPSEEK_API_KEY environment variable is not set")
	}

	question := "9.11 and 9.8, which is greater? Explain step by step."

	// Reasoning effort levels available in Eino's OpenAI adapter.
	// These mirror the OpenAI API constants:
	//   Reason. Effort  |  OpenAI value  |  DeepSeek mapping
	//   ────────────────┼────────────────┼───────────────────
	//   Low             |  "low"         |  →  "high"
	//   Medium          |  "medium"      |  →  "high"
	//   High            |  "high"        |  →  "high" / "max" (for agent requests)
	//
	// DeepSeek V4 Flash maps "low" and "medium" to "high" for compatibility,
	// so all three levels produce similar reasoning length on DeepSeek.
	// The API accepts them regardless; the mapping happens server-side.

	levels := []struct {
		label  string
		effort openai.ReasoningEffortLevel
	}{
		{"Low", openai.ReasoningEffortLevelLow},
		{"Medium", openai.ReasoningEffortLevelMedium},
		{"High", openai.ReasoningEffortLevelHigh},
	}

	// ── Approach 1: static ChatModelConfig ──────────────────────────────────
	//
	// Here we create a NEW ChatModel per effort level. The ReasoningEffort
	// is set in ChatModelConfig at creation time and every Generate() call
	// on that instance uses the same level. This is simpler but means you
	// need separate model instances if you want different effort levels.

	for _, lvl := range levels {
		fmt.Printf("\n══════════════ Reasoning Effort: %s ══════════════\n", lvl.label)
		fmt.Printf("Question: %s\n\n", question)

		// ChatModelConfig.ReasoningEffort sets a static default that all
		// Generate() calls on this instance will use.
		cm, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
			APIKey:          apiKey,
			BaseURL:         "https://api.deepseek.com/v1",
			Model:           "deepseek-v4-flash",
			Timeout:         120 * time.Second,
			ReasoningEffort: lvl.effort,
			// DeepSeek requires "thinking" in extra_body to enable the
			// chain-of-thought mode. Eino passes ExtraFields as extra_body.
			ExtraFields: map[string]any{
				"thinking": map[string]any{"type": "enabled"},
			},
		})
		if err != nil {
			log.Fatalf("Failed to create chat model: %v", err)
		}

		resp, err := cm.Generate(ctx, []*schema.Message{
			schema.UserMessage(question),
		})
		if err != nil {
			log.Printf("Error: %v\n", err)
			continue
		}

		// schema.Message exposes ReasoningContent directly — the Eino
		// adapter automatically populates it from the API response's
		// reasoning_content field.
		if resp.ReasoningContent != "" {
			fmt.Printf("─── Reasoning Content ───\n%s\n\n", resp.ReasoningContent)
		}
		if resp.Content != "" {
			fmt.Printf("─── Final Answer ───\n%s\n", resp.Content)
		}
	}

	// ── Approach 2: per-request WithReasoningEffort option ──────────────────
	//
	// Here we create ONE ChatModel WITHOUT reasoning effort in the config,
	// then pass it as a functional option to each Generate() call. This lets
	// you reuse the same model instance but vary the effort per request —
	// useful when different prompts need different reasoning depth (e.g. a
	// simple classification needs "low", a complex analysis needs "high").

	fmt.Printf("\n══════════════ Per-Request Options ══════════════\n")

	cm, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
		APIKey:  apiKey,
		BaseURL: "https://api.deepseek.com/v1",
		Model:   "deepseek-v4-flash",
		Timeout: 120 * time.Second,
		// Note: no ReasoningEffort or ExtraFields here — those will be
		// supplied per-request via functional options.
	})
	if err != nil {
		log.Fatalf("Failed to create chat model: %v", err)
	}

	efforts := []openai.ReasoningEffortLevel{
		openai.ReasoningEffortLevelLow,
		openai.ReasoningEffortLevelMedium,
		openai.ReasoningEffortLevelHigh,
	}
	labels := []string{"Low", "Medium", "High"}

	for i, effort := range efforts {
		msg := fmt.Sprintf("Explain the concept of %s reasoning effort in one paragraph.",
			labels[i])

		// WithReasoningEffort and WithExtraFields are per-request functional
		// options. They override any config-level defaults and are scoped
		// to just this single Generate() call.
		resp, err := cm.Generate(ctx, []*schema.Message{
			schema.UserMessage(msg),
		},
			openai.WithReasoningEffort(effort),
			openai.WithExtraFields(map[string]any{
				"thinking": map[string]any{"type": "enabled"},
			}),
		)
		if err != nil {
			log.Printf("Error with %s effort: %v\n", labels[i], err)
			continue
		}

		fmt.Printf("\n─── %s Effort ───\n", labels[i])
		if resp.ReasoningContent != "" {
			fmt.Printf("Reasoning: %s...\n",
				resp.ReasoningContent[:min(len(resp.ReasoningContent), 200)])
		}
		fmt.Printf("Answer: %s\n",
			resp.Content[:min(len(resp.Content), 300)])
	}
}
```

