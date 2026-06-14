# Agents

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
