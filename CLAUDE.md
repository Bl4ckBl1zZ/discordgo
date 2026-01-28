# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Run all tests
go test -v -race ./...

# Run a single test
go test -v -race -run TestFunctionName ./...

# Check formatting (CI enforces this)
diff <(gofmt -d .) <(echo -n)

# Format code
gofmt -w .

# Run linter
go vet -x ./...

# Generate event handlers (after modifying events.go)
go generate ./...
```

## Architecture Overview

DiscordGo is a low-level Go binding for the Discord API. It provides direct mappings to Discord's REST API, WebSocket gateway, and voice connections.

### Core Components

**Session (`structs.go`, `discord.go`)**: The central struct that holds all connection state, configuration, and provides methods for API interactions. Created via `discordgo.New("Bot token")`.

**REST API (`restapi.go`, `endpoints.go`)**: HTTP client methods for Discord's REST API. All endpoint URL builders are in `endpoints.go`. REST methods accept `RequestOption` functional options for per-request customization.

**WebSocket Gateway (`wsapi.go`)**: Manages the persistent WebSocket connection to Discord for real-time events. Handles connection lifecycle, heartbeating, and reconnection logic.

**Event System (`events.go`, `eventhandlers.go`)**:
- `events.go` defines all event structs (DO NOT add non-event structs here)
- `eventhandlers.go` is **generated code** - do not edit directly
- Run `go generate ./...` after modifying `events.go` to regenerate handlers
- The generator is at `tools/cmd/eventhandlers/main.go`

**State Tracking (`state.go`)**: Optional in-memory cache of guilds, channels, users, etc. Updated automatically from gateway events when `StateEnabled` is true.

**Voice (`voice.go`)**: Handles voice channel connections with both WebSocket and UDP components for real-time audio.

**Components (`components.go`)**: Message component types (buttons, select menus, modals, etc.) with JSON marshaling via the `MessageComponent` interface.

**Interactions (`interactions.go`)**: Application commands (slash commands), context menus, and interaction response handling.

### Key Patterns

- Event handlers are registered via `Session.AddHandler(func(*Session, *EventType))`
- All API methods are on the `Session` type
- Snowflake IDs are strings throughout the codebase
- The `Marshal` and `Unmarshal` package variables allow custom JSON encoding

### Directory Structure

- Root: Core library code (single package `discordgo`)
- `examples/`: Working examples demonstrating various features
- `tools/cmd/eventhandlers/`: Code generator for event handlers
