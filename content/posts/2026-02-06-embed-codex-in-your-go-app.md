---
title:  "Embed Codex in your Go application"
slug: embed-codex-in-your-go-app
date:   2026-02-06 12:40:00
draft: false
categories: ["golang", "ai", "codex"]
---
If you already have a go application with real users, running Codex as a separate tool gets awkward pretty fast.

I wanted to run prompts from backend code, stream updates to my own UI, and keep approval behavior under my own control. To make that easier I put together [`github.com/pmenglund/codex-sdk-go`](https://github.com/pmenglund/codex-sdk-go), which talks to `codex app-server` over JSON-RPC.

The basic flow is simple:

1. create a client
2. start a thread
3. run turns
4. close the client when done

A minimal example:

```go
package main

import (
    "context"
    "fmt"
    "log/slog"
    "os"

    "github.com/pmenglund/codex-sdk-go"
)

func main() {
    ctx := context.Background()
    logger := slog.New(slog.NewTextHandler(os.Stderr, &slog.HandlerOptions{Level: slog.LevelInfo}))

    client, err := codex.New(ctx, codex.Options{Logger: logger})
    if err != nil {
        panic(err)
    }
    defer client.Close()

    thread, err := client.StartThread(ctx, codex.ThreadStartOptions{})
    if err != nil {
        panic(err)
    }

    result, err := thread.Run(ctx, "Diagnose the test failure and propose a fix", nil)
    if err != nil {
        panic(err)
    }

    fmt.Println(result.FinalResponse)
}
```

I designed `codex.New` so the context is only used during initialization. Once the client has started, process lifetime is handled by `Close`.

For quick demos I usually do auto-approval:

```go
client, err := codex.New(ctx, codex.Options{
    Logger:          logger,
    ApprovalHandler: codex.AutoApproveHandler{Logger: logger},
})
```

For production I recommend writing your own approval handler and explicitly deciding what should be allowed automatically.

Another thing I like is structured output. Instead of parsing free-form text, you can give Codex a schema and get predictable data back:

```go
schema := codex.MustJSON(map[string]any{
    "type": "object",
    "properties": map[string]any{
        "summary": map[string]any{"type": "string"},
        "status":  map[string]any{"type": "string", "enum": []string{"ok", "action_required"}},
    },
    "required":             []string{"summary", "status"},
    "additionalProperties": false,
})

_, err := thread.RunInputs(
    ctx,
    []codex.Input{codex.TextInput("Summarize repo status")},
    &codex.TurnOptions{OutputSchema: schema},
)
```

If you're building tooling like code review helpers, migration assistants, or incident support inside an existing go service, this has worked well for me.
