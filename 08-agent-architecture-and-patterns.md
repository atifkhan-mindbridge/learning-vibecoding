# Step 08: Agent Architecture and Patterns

**Level:** Intermediate-Advanced
**Time:** ~40 minutes

In [Step 07](07-Test-Driven-Development-with-AI.md), you learned how TDD provides verifiable development with AI. This step explores how AI coding assistants work under the hood as agents, and introduces advanced patterns for orchestrating multiple agents on complex tasks.

## Learning Objectives

By the end of this step, you will be able to:

1. Understand the agent loop: gather context → take action → verify → repeat
2. Learn MCP (Model Context Protocol) and tool integration
3. Master patterns: single-agent, multi-agent, orchestrator
4. Apply advanced patterns: Ralph Wiggum, Gas Town, Generator-Critic

---

## What Is an Agent?

An **agent** is an AI system that can:
1. Take actions in an environment
2. Observe the results of those actions
3. Decide what to do next based on results
4. Work toward a goal autonomously

**Key distinction:** A chatbot responds to prompts. An agent acts, observes, and iterates.

### The Agent Equation

```
Agent = LLM + Tools + Autonomy
```

- **LLM:** The reasoning engine (Claude, GPT-4, etc.)
- **Tools:** Capabilities to act in the world (file editing, shell commands, APIs)
- **Autonomy:** Ability to decide what to do next without human intervention

### Agent vs. Assistant

Understanding the spectrum between assistant and agent helps clarify what you're working with:

| Aspect | Assistant | Agent |
|--------|-----------|-------|
| Response style | Answers questions | Completes tasks |
| Tool use | On request | Autonomous selection |
| Iteration | Single response | Loop until done |
| Human involvement | Every turn | Checkpoints only |
| Error handling | Reports errors | Attempts recovery |

Claude Code operates as an **agent** when given autonomous tasks, but can function as an **assistant** for Q&A and guidance.

---

## ACE and AEE: Two Workbench Perspectives

The SASE (Structured Agentic Software Engineering) framework [^1] introduces two complementary perspectives on agent environments:

[^1]: Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)

### ACE: Agent Command Environment

The **Agent Command Environment (ACE)** is the human-facing interface where:
- Humans set strategy and goals
- Oversight and monitoring happens
- Approval gates are presented
- High-level orchestration is controlled

**ACE is where humans coach and direct agents.**

### AEE: Agent Execution Environment

The **Agent Execution Environment (AEE)** is where agents actually work:
- Tools and actions are available
- Memory and context are managed
- Feedback loops operate
- Iteration happens

**AEE is where agents execute and iterate.**

### ACE/AEE Workbench Architecture

The following diagram illustrates how ACE and AEE work together:

```
┌─────────────────────────────────────────────────────────────────┐
│              ACE/AEE WORKBENCH ARCHITECTURE                     │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │           ACE (Agent Command Environment)               │   │
│   │           Human-oriented orchestration                  │   │
│   │                                                         │   │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │   │
│   │   │  Strategy   │  │  Oversight  │  │  Approval   │    │   │
│   │   │   Setting   │  │   Panels    │  │   Gates     │    │   │
│   │   └─────────────┘  └─────────────┘  └─────────────┘    │   │
│   └────────────────────────────┬────────────────────────────┘   │
│                                │                                │
│                     Human sets goals, monitors, approves        │
│                                │                                │
│                                ▼                                │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │           AEE (Agent Execution Environment)             │   │
│   │           Agent-oriented execution                      │   │
│   │                                                         │   │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │   │
│   │   │  Tools &    │  │  Memory &   │  │  Feedback   │    │   │
│   │   │  Actions    │  │  Context    │  │   Loop      │    │   │
│   │   └─────────────┘  └─────────────┘  └─────────────┘    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  ACE: Where humans coach and oversee                    │   │
│   │  AEE: Where agents execute and iterate                  │   │
│   │                                                         │   │
│   │  The boundary between them defines the handoff points   │   │
│   │  and level of autonomy granted to the agent.            │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Why the Distinction Matters

Different concerns apply to each environment:

**ACE concerns:**
- How do humans understand what agents are doing?
- Where should approval gates be placed?
- How do humans intervene when needed?
- What metrics indicate agent effectiveness?

**AEE concerns:**
- What tools does the agent need?
- How is context managed across iterations?
- What feedback signals guide improvement?
- How does the agent know when it's done?

When building agent workflows, consider both perspectives:
- Design ACE interfaces for human oversight and control
- Design AEE capabilities for agent effectiveness

---

## The Agent Loop

The core pattern behind all effective agents, with emphasis on evidence collection:

```
┌─────────────────────────────────────────────────────────────────┐
│              AGENT LOOP WITH EVIDENCE                           │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  GATHER CONTEXT                                         │   │
│   │  • Read files        (evidence: file contents)          │   │
│   │  • Search code       (evidence: search results)         │   │
│   │  • Fetch docs        (evidence: documentation)          │   │
│   └───────────────────────────┬─────────────────────────────┘   │
│                               │                                 │
│                               ▼                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  TAKE ACTION                                            │   │
│   │  • Edit files        (evidence: diff)                   │   │
│   │  • Run commands      (evidence: output)                 │   │
│   │  • Call APIs         (evidence: response)               │   │
│   └───────────────────────────┬─────────────────────────────┘   │
│                               │                                 │
│                               ▼                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  VERIFY WORK                                            │   │
│   │  • Run tests         (evidence: test results)           │   │
│   │  • Check output      (evidence: screenshots)            │   │
│   │  • Validate logic    (evidence: analysis)               │   │
│   └───────────────────────────┬─────────────────────────────┘   │
│                               │                                 │
│                               ▼                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  DONE? (with evidence bundle)                           │   │
│   │  No  ──► gather more context (loop back)                │   │
│   │  Yes ──► return result with evidence                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Key Insight: Every action produces evidence.           │   │
│   │  Evidence enables:                                      │   │
│   │  • Verification that work was done correctly            │   │
│   │  • Audit trails for review                              │   │
│   │  • Debugging when things go wrong                       │   │
│   │  • Learning from successes and failures                 │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Phase 1: Gather Context

The agent collects information it needs:
- Read relevant files
- Search the codebase
- Fetch documentation
- Query databases
- Access external APIs

**Key insight:** Good agents spend significant time gathering context before acting.

### Phase 2: Take Action

The agent modifies the environment:
- Edit files
- Run shell commands
- Call APIs
- Create new files
- Make commits

### Phase 3: Verify Work

The agent validates its actions:
- Run tests
- Check for errors
- Validate output format
- Compare to expected results

### Phase 4: Iterate or Complete

Based on verification:
- If successful: return result
- If failed: gather more context and try again
- If stuck: escalate to human

---

## MCP: Model Context Protocol

### What Is MCP?

**Model Context Protocol (MCP)** is a standard for connecting AI agents to external tools and services. It provides:

- **Standardized interfaces** for tools
- **Authentication handling** for services
- **Consistent tool discovery** across providers

### MCP Architecture

```
┌─────────────────────────────────────────────────────┐
│                     HOST                            │
│            (Claude Code, Cursor, etc.)              │
│                                                     │
│  ┌─────────────────┐   ┌─────────────────┐         │
│  │   MCP CLIENT    │   │   MCP CLIENT    │         │
│  └────────┬────────┘   └────────┬────────┘         │
└───────────┼─────────────────────┼───────────────────┘
            │                     │
            ▼                     ▼
     ┌──────────────┐     ┌──────────────┐
     │  MCP SERVER  │     │  MCP SERVER  │
     │   (GitHub)   │     │  (Puppeteer) │
     └──────────────┘     └──────────────┘
```

**Components:**
- **Host:** The application running the agent (Claude Code, Cursor)
- **Client:** Connects to MCP servers on behalf of the agent
- **Server:** Provides tools and capabilities (file access, APIs, browser control)

### Setting Up MCP

**In Claude Code:**

1. **Project config** (available in specific directory):
```json
// .mcp.json
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"]
    },
    "puppeteer": {
      "command": "npx",
      "args": ["@anthropic/puppeteer-mcp-server"]
    }
  }
}
```

2. **Global config** (available everywhere):
```bash
claude config add-mcp github "npx @modelcontextprotocol/server-github"
```

### Popular MCP Servers

| Server | Capabilities |
|--------|-------------|
| GitHub | Issues, PRs, repositories |
| Puppeteer | Browser automation, screenshots |
| Slack | Messages, channels |
| Google Drive | File access, search |
| Postgres | Database queries |
| Sentry | Error monitoring |

### Using MCP Tools

Once configured, tools appear automatically:

```
Search GitHub issues for authentication bugs in our repo.
```

Claude Code will use the GitHub MCP server to execute this.

### Complete MCP Server Implementation

Here's a fully working MCP server that provides weather data. This demonstrates the key patterns for building your own MCP servers:

```typescript
// weather-mcp-server/src/index.ts
// A complete MCP server implementation

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ErrorCode,
  McpError,
} from "@modelcontextprotocol/sdk/types.js";

// =============================================================================
// Types
// =============================================================================

interface WeatherData {
  location: string;
  temperature: number;
  unit: "celsius" | "fahrenheit";
  conditions: string;
  humidity: number;
  wind_speed: number;
  timestamp: string;
}

interface ForecastDay {
  date: string;
  high: number;
  low: number;
  conditions: string;
}

// =============================================================================
// Weather Service (simulated - replace with real API in production)
// =============================================================================

async function fetchCurrentWeather(location: string): Promise<WeatherData> {
  // In production: call OpenWeatherMap, WeatherAPI, etc.
  // For demo: return simulated data

  // Simulate API latency
  await new Promise(resolve => setTimeout(resolve, 100));

  return {
    location: location,
    temperature: 72,
    unit: "fahrenheit",
    conditions: "Partly cloudy",
    humidity: 45,
    wind_speed: 10,
    timestamp: new Date().toISOString(),
  };
}

async function fetchForecast(location: string, days: number): Promise<ForecastDay[]> {
  await new Promise(resolve => setTimeout(resolve, 100));

  const forecast: ForecastDay[] = [];
  const baseDate = new Date();

  for (let i = 0; i < days; i++) {
    const date = new Date(baseDate);
    date.setDate(date.getDate() + i);
    forecast.push({
      date: date.toISOString().split('T')[0],
      high: 75 + Math.floor(Math.random() * 10),
      low: 55 + Math.floor(Math.random() * 10),
      conditions: ["Sunny", "Cloudy", "Partly cloudy", "Rain"][Math.floor(Math.random() * 4)],
    });
  }

  return forecast;
}

// =============================================================================
// MCP Server Setup
// =============================================================================

const server = new Server(
  {
    name: "weather-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// =============================================================================
// Tool Definitions
// =============================================================================

server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_current_weather",
        description: `Get the current weather for a location.

Returns temperature, conditions, humidity, and wind speed.
Use this when the user asks about current weather conditions.`,
        inputSchema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "City name, optionally with state/country (e.g., 'San Francisco, CA' or 'London, UK')",
            },
            units: {
              type: "string",
              enum: ["celsius", "fahrenheit"],
              description: "Temperature unit (default: fahrenheit)",
            },
          },
          required: ["location"],
        },
      },
      {
        name: "get_forecast",
        description: `Get a weather forecast for upcoming days.

Returns daily high/low temperatures and conditions.
Use this when the user asks about future weather.`,
        inputSchema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "City name, optionally with state/country",
            },
            days: {
              type: "number",
              minimum: 1,
              maximum: 10,
              description: "Number of days to forecast (1-10, default: 5)",
            },
          },
          required: ["location"],
        },
      },
      {
        name: "compare_weather",
        description: `Compare current weather between two locations.

Use when the user wants to compare weather in different cities.`,
        inputSchema: {
          type: "object",
          properties: {
            location1: {
              type: "string",
              description: "First city to compare",
            },
            location2: {
              type: "string",
              description: "Second city to compare",
            },
          },
          required: ["location1", "location2"],
        },
      },
    ],
  };
});

// =============================================================================
// Tool Handlers
// =============================================================================

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  try {
    switch (name) {
      case "get_current_weather": {
        const location = args?.location as string;
        if (!location) {
          throw new McpError(ErrorCode.InvalidParams, "Location is required");
        }

        const weather = await fetchCurrentWeather(location);

        // Convert units if requested
        let temp = weather.temperature;
        let unit = weather.unit;
        if (args?.units === "celsius" && weather.unit === "fahrenheit") {
          temp = Math.round((weather.temperature - 32) * 5 / 9);
          unit = "celsius";
        }

        return {
          content: [
            {
              type: "text",
              text: `Current weather in ${weather.location}:
- Temperature: ${temp}°${unit === "celsius" ? "C" : "F"}
- Conditions: ${weather.conditions}
- Humidity: ${weather.humidity}%
- Wind: ${weather.wind_speed} mph
- Updated: ${weather.timestamp}`,
            },
          ],
        };
      }

      case "get_forecast": {
        const location = args?.location as string;
        if (!location) {
          throw new McpError(ErrorCode.InvalidParams, "Location is required");
        }

        const days = (args?.days as number) || 5;
        const forecast = await fetchForecast(location, days);

        const forecastText = forecast
          .map(day => `${day.date}: High ${day.high}°F, Low ${day.low}°F - ${day.conditions}`)
          .join("\n");

        return {
          content: [
            {
              type: "text",
              text: `${days}-day forecast for ${location}:\n\n${forecastText}`,
            },
          ],
        };
      }

      case "compare_weather": {
        const loc1 = args?.location1 as string;
        const loc2 = args?.location2 as string;

        if (!loc1 || !loc2) {
          throw new McpError(ErrorCode.InvalidParams, "Both locations are required");
        }

        // Fetch both in parallel
        const [weather1, weather2] = await Promise.all([
          fetchCurrentWeather(loc1),
          fetchCurrentWeather(loc2),
        ]);

        const tempDiff = weather1.temperature - weather2.temperature;
        const comparison = tempDiff > 0
          ? `${loc1} is ${tempDiff}°F warmer`
          : tempDiff < 0
            ? `${loc2} is ${Math.abs(tempDiff)}°F warmer`
            : "Both locations have the same temperature";

        return {
          content: [
            {
              type: "text",
              text: `Weather Comparison:

${loc1}:
- ${weather1.temperature}°F, ${weather1.conditions}

${loc2}:
- ${weather2.temperature}°F, ${weather2.conditions}

${comparison}`,
            },
          ],
        };
      }

      default:
        throw new McpError(ErrorCode.MethodNotFound, `Unknown tool: ${name}`);
    }
  } catch (error) {
    if (error instanceof McpError) {
      throw error;
    }
    // Wrap unexpected errors
    throw new McpError(
      ErrorCode.InternalError,
      `Weather service error: ${error instanceof Error ? error.message : 'Unknown error'}`
    );
  }
});

// =============================================================================
// Server Startup
// =============================================================================

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP server running on stdio");
}

main().catch((error) => {
  console.error("Server error:", error);
  process.exit(1);
});
```

**Package.json for the MCP server:**

```json
{
  "name": "weather-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**Key patterns in this implementation:**

1. **Clear tool descriptions** - Help the AI understand when to use each tool
2. **Input validation** - Check required parameters and throw McpError
3. **Structured output** - Return formatted text that's easy to parse
4. **Error handling** - Distinguish between user errors and internal errors
5. **Parallel operations** - Use Promise.all for independent fetches

### MCP Configuration Examples

**Project-level configuration (.mcp.json in project root):**

```json
{
  "$schema": "https://modelcontextprotocol.io/schema/mcp-config.json",
  "servers": {
    "weather": {
      "command": "node",
      "args": ["./tools/weather-mcp-server/dist/index.js"],
      "description": "Local weather server for development"
    },
    "database": {
      "command": "npx",
      "args": ["@mcp/postgres-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      },
      "description": "PostgreSQL database access"
    },
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_OWNER": "myorg",
        "GITHUB_REPO": "myrepo"
      }
    },
    "browser": {
      "command": "npx",
      "args": ["@anthropic/puppeteer-mcp-server"],
      "description": "Browser automation for visual testing"
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "--allowed-directories", "/Users/me/projects/myapp/src",
        "--allowed-directories", "/Users/me/projects/myapp/tests"
      ],
      "description": "Restricted filesystem access"
    }
  }
}
```

**Global configuration (for tools you want everywhere):**

```bash
# Add GitHub tools globally
claude config add-mcp github "npx @modelcontextprotocol/server-github"

# Add with environment variables
claude config add-mcp slack "npx @mcp/slack-server" \
  --env SLACK_TOKEN='${SLACK_TOKEN}'

# List configured servers
claude config list-mcp
```

**Environment-specific configuration pattern:**

```json
// .mcp.json - shared base configuration
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

```json
// .mcp.development.json - development extras
{
  "extends": "./.mcp.json",
  "servers": {
    "database": {
      "command": "npx",
      "args": ["@mcp/postgres-server"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/dev"
      }
    }
  }
}
```

---

## Tool Design Principles

When building custom MCP servers or designing tool interfaces for agents, follow these principles from Anthropic's "Building Effective Agents" guide:

### Principle 1: Single Responsibility

Each tool should do one thing well.

**Good:** `search_codebase(query)` returns file paths matching query
**Good:** `read_file(path)` returns contents of a file
**Bad:** `search_and_read_all(query)` returns all matching file contents

Why? Single-responsibility tools let agents compose operations efficiently. The bad example fills context with potentially irrelevant content.

### Principle 2: Efficient Output

Return only what's needed for the next decision.

**Good:** Search returns file paths (agent selects which to read)
**Bad:** Search returns full file contents (agent context fills up)

Agents iterate—give them information to make good next decisions, not everything they might eventually need.

### Principle 3: Composable Operations

Design tools to chain together naturally.

A good tool set might be:
- `search(query)` → returns file paths
- `filter(paths, criteria)` → narrows results
- `read(path, lines)` → returns specific content
- `analyze(content)` → extracts information

Each step focused, each output feeds the next.

### Principle 4: Clear Boundaries

Tools should clearly specify:
- What they do
- What they don't do
- Error conditions
- Return format

Ambiguous tools confuse agents. Explicit boundaries help agents predict behavior and handle edge cases.

### Principle 5: Feedback for Iteration

Tools should return information that helps agents improve:
- Was the operation successful?
- What changed?
- What warnings or notes apply?
- What should the agent check next?

### Tool Design Principles Summary

```
┌─────────────────────────────────────────────────────────────────┐
│              TOOL DESIGN PRINCIPLES                             │
│                                                                 │
│   1. SINGLE RESPONSIBILITY                                      │
│      ┌───────────────────┐     not    ┌───────────────────┐    │
│      │  search_files()   │     ───►   │  search_and_read  │    │
│      │  read_file()      │            │  _everything()    │    │
│      └───────────────────┘            └───────────────────┘    │
│      Focused, composable              Monolithic, rigid         │
│                                                                 │
│   2. EFFICIENT OUTPUT                                           │
│      Return: file paths               not: file contents        │
│      Agent: selects relevant              context overload      │
│      Next: decides what to read           decisions made        │
│                                                                 │
│   3. COMPOSABLE PIPELINE                                        │
│      search() ──► filter() ──► read() ──► analyze()            │
│      Each step: focused, chainable                              │
│                                                                 │
│   4. CLEAR BOUNDARIES                                           │
│      ┌─────────────────────────────────────────────────────┐   │
│      │  DOES: Replace exact text match                     │   │
│      │  DOES NOT: Create files, handle regex               │   │
│      │  ERRORS: FileNotFoundError if path invalid          │   │
│      │  RETURNS: { success, diff, warnings }               │   │
│      └─────────────────────────────────────────────────────┘   │
│                                                                 │
│   5. FEEDBACK FOR ITERATION                                     │
│      { success: false,                                          │
│        error: "...",                                            │
│        suggestions: ["Try X", "Check Y"],                       │
│        next_steps: ["Verify Z"] }                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Tool Design Examples: Good vs Bad

Here are concrete examples showing these principles in practice:

```python
"""
Tool Design Examples
Demonstrating Anthropic's principles for effective agent tools.
"""

# =============================================================================
# PRINCIPLE 1: SINGLE RESPONSIBILITY
# =============================================================================

# BAD: Does too much, fills context with potentially irrelevant data
def search_and_read_all(query: str) -> dict:
    """
    Search codebase and return ALL matching file contents.
    """
    matching_files = ripgrep_search(query)
    results = {}
    for file_path in matching_files:
        with open(file_path) as f:
            results[file_path] = f.read()  # Could be megabytes!
    return results


# GOOD: Separate concerns, agent decides what to read
def search_codebase(query: str, file_pattern: str = "*") -> list[str]:
    """
    Search codebase for files matching query.

    Returns list of file paths (not contents).
    Use read_file() to get contents of interesting files.

    Args:
        query: Text pattern to search for
        file_pattern: Glob pattern to filter files (e.g., "*.py")

    Returns:
        List of file paths that contain matches, sorted by relevance
    """
    return ripgrep_search(query, glob=file_pattern)


def read_file(path: str, start_line: int = 0, end_line: int = -1) -> str:
    """
    Read file contents with optional line range.

    Use search_codebase() first to find relevant files.
    For large files, use start_line/end_line to read sections.

    Args:
        path: File path to read
        start_line: First line to read (0-indexed, default: start)
        end_line: Last line to read (-1 means end of file)

    Returns:
        File contents as string, with line numbers prefixed
    """
    # Implementation reads only requested lines
    pass


# =============================================================================
# PRINCIPLE 2: EFFICIENT OUTPUT
# =============================================================================

# BAD: Returns everything, overwhelming context
def get_project_structure_bad() -> dict:
    """Returns complete file tree with ALL file contents."""
    result = {}
    for root, dirs, files in os.walk("."):
        for file in files:
            path = os.path.join(root, file)
            with open(path) as f:
                result[path] = {
                    "content": f.read(),
                    "size": os.path.getsize(path),
                    "modified": os.path.getmtime(path),
                }
    return result  # Could be gigabytes!


# GOOD: Returns summary, agent requests details as needed
def get_project_structure(
    path: str = ".",
    max_depth: int = 3,
    show_hidden: bool = False
) -> str:
    """
    Get project structure as a tree view.

    Returns directory tree (not file contents).
    Use read_file() to examine specific files.

    Args:
        path: Root path to explore
        max_depth: How deep to recurse (default: 3)
        show_hidden: Include hidden files/dirs (default: False)

    Returns:
        ASCII tree representation like:
        src/
        ├── components/
        │   ├── Button.tsx
        │   └── Modal.tsx
        └── lib/
            └── utils.ts
    """
    pass


# GOOD: Provide statistics for decision-making
def get_file_info(path: str) -> dict:
    """
    Get metadata about a file without reading contents.

    Useful for deciding whether to read a file (check size first).

    Returns:
        {
            "path": "/src/big-file.ts",
            "size_bytes": 50000,
            "size_human": "49 KB",
            "lines": 1200,
            "language": "typescript",
            "last_modified": "2024-01-15T10:30:00Z"
        }
    """
    pass


# =============================================================================
# PRINCIPLE 3: COMPOSABLE OPERATIONS
# =============================================================================

# GOOD: Pipeline of focused tools
# Agent can chain: search -> filter -> read -> analyze

def search(query: str, path: str = ".") -> list[SearchResult]:
    """Find files containing query. Returns paths with match context."""
    pass


def filter_files(
    paths: list[str],
    language: str = None,
    modified_after: str = None,
    max_size: int = None
) -> list[str]:
    """Filter file list by criteria. Use after search() to narrow results."""
    pass


def read_section(path: str, around_line: int, context: int = 10) -> str:
    """Read lines around a specific line number. Useful after search()."""
    pass


def analyze_imports(path: str) -> dict:
    """Analyze import statements in a file. Returns dependency graph."""
    pass


# Example agent workflow:
# 1. search("handleAuth") -> ["src/auth/login.ts", "src/auth/session.ts", ...]
# 2. filter_files(results, language="typescript") -> ["src/auth/login.ts", ...]
# 3. read_section("src/auth/login.ts", around_line=45) -> relevant code
# 4. analyze_imports("src/auth/login.ts") -> {"express": {...}, "bcrypt": {...}}


# =============================================================================
# PRINCIPLE 4: CLEAR BOUNDARIES
# =============================================================================

# BAD: Ambiguous, agent can't predict behavior
def edit_file(path: str, changes: str) -> str:
    """Edit a file with the given changes."""
    # What format are "changes"? Natural language? Diff? Full content?
    # What happens if file doesn't exist?
    # Is there any validation?
    pass


# GOOD: Explicit contract
def replace_in_file(
    path: str,
    old_text: str,
    new_text: str,
    occurrence: int = 1
) -> dict:
    """
    Replace text in a file.

    DOES:
    - Replaces exact match of old_text with new_text
    - By default, replaces only first occurrence
    - Returns the changes made for verification

    DOES NOT:
    - Create files (use create_file instead)
    - Handle regex (use replace_pattern for regex)
    - Make backup (caller should commit first)

    Args:
        path: File to modify (must exist)
        old_text: Exact text to find (must exist in file)
        new_text: Replacement text
        occurrence: Which occurrence to replace (1=first, -1=all)

    Returns:
        {
            "success": True,
            "file": "/path/to/file.ts",
            "replacements": 1,
            "diff": "- old line\\n+ new line"
        }

    Raises:
        FileNotFoundError: If path doesn't exist
        ValueError: If old_text not found in file
    """
    pass


# =============================================================================
# PRINCIPLE 5: FEEDBACK FOR ITERATION
# =============================================================================

# BAD: Minimal feedback
def run_tests_bad(path: str) -> bool:
    """Run tests. Returns True if passed."""
    result = subprocess.run(["pytest", path])
    return result.returncode == 0  # Agent knows nothing useful


# GOOD: Rich feedback for next decision
def run_tests(
    path: str = ".",
    pattern: str = None,
    verbose: bool = False
) -> dict:
    """
    Run test suite and return detailed results.

    Returns structured data to help agent decide next steps:
    - If tests fail: which tests, what errors, what files
    - If tests pass: coverage info, duration
    - Always: suggestions for what to do next

    Returns:
        {
            "success": False,
            "summary": "2 passed, 1 failed, 0 errors",
            "duration_seconds": 3.5,
            "failed_tests": [
                {
                    "name": "test_user_login",
                    "file": "tests/test_auth.py",
                    "line": 45,
                    "error": "AssertionError: Expected 200, got 401",
                    "relevant_code": "assert response.status == 200"
                }
            ],
            "passed_tests": ["test_user_register", "test_user_logout"],
            "coverage": {
                "total": 78,
                "uncovered_files": ["src/auth/oauth.py"]
            },
            "suggestions": [
                "Check authentication logic in src/auth/login.py",
                "The test expects status 200 but got 401 (unauthorized)"
            ]
        }
    """
    pass


# GOOD: Operation result with next steps
def create_file(path: str, content: str) -> dict:
    """
    Create a new file with given content.

    Returns:
        {
            "success": True,
            "path": "/src/new-file.ts",
            "size_bytes": 1234,
            "next_steps": [
                "Run 'npm run typecheck' to verify TypeScript",
                "Add tests in tests/new-file.test.ts",
                "Update exports in src/index.ts if public API"
            ],
            "warnings": [
                "File created in src/ - remember to add to git"
            ]
        }
    """
    pass
```

**Summary Table: Good vs Bad Tool Design**

| Aspect | Bad Pattern | Good Pattern |
|--------|-------------|--------------|
| **Scope** | search_and_read_everything() | search() + read() separately |
| **Output** | Return all file contents | Return paths, let agent choose |
| **Composability** | Monolithic operations | Pipeline of focused tools |
| **Contract** | Vague parameters | Explicit types, clear docs |
| **Errors** | Silent failures, booleans | Structured error info |
| **Feedback** | "Done" or True/False | What changed, what to do next |

---

## Agent Patterns

### Pattern 1: Single Agent

The simplest pattern—one agent handles everything:

```
┌────────────────────────────┐
│      Single Agent          │
│                            │
│  Context → Action → Verify │
│                            │
└────────────────────────────┘
```

**Best for:**
- Simple, focused tasks
- Tasks requiring consistent state
- Sequential operations

**Limitations:**
- Context window fills quickly
- Single point of failure
- Cannot parallelize

### Pattern 2: Subagents

A main agent spawns specialized subagents:

```
┌────────────────────────────────────────────┐
│            Orchestrator Agent              │
│                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │ Research │ │  Code    │ │  Test    │   │
│  │ Subagent │ │ Subagent │ │ Subagent │   │
│  └──────────┘ └──────────┘ └──────────┘   │
└────────────────────────────────────────────┘
```

**Benefits:**
- Each subagent has its own context window
- Parallel execution possible
- Isolated failures
- Focused expertise

**Using subagents in Claude Code:**
```
Use subagents to research the following in parallel:
1. Subagent 1: How authentication works in this codebase
2. Subagent 2: What testing patterns are used
3. Subagent 3: How similar features are documented

Return only relevant findings from each.
```

### Pattern 3: Multi-Claude Workflows

Multiple independent Claude instances working together:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  ┌───────────┐         ┌───────────┐               │
│  │  Claude 1 │  ──→    │  Claude 2 │               │
│  │  (Writer) │         │ (Reviewer)│               │
│  └───────────┘         └───────────┘               │
│       │                      │                     │
│       ▼                      ▼                     │
│  ┌───────────┐         ┌───────────┐               │
│  │ Codebase  │         │  Review   │               │
│  │  Changes  │         │ Feedback  │               │
│  └───────────┘         └───────────┘               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Three approaches:**

1. **Sequential verification:**
   - Claude 1 writes code
   - `/clear` or new terminal
   - Claude 2 reviews the code

2. **Multiple checkouts:**
   - 3-4 git checkouts in separate folders
   - Separate Claude instance in each
   - Work on different tasks simultaneously

3. **Git worktrees:**
   ```bash
   git worktree add ../feature-a feature-a
   git worktree add ../feature-b feature-b
   # Run claude in each worktree directory
   ```

### Pattern 4: Generator-Critic

Separate creation from validation:

```
┌────────────────────────────────────────────────────┐
│                                                    │
│  ┌───────────┐         ┌───────────┐              │
│  │ Generator │  ──→    │   Critic  │              │
│  │           │  ←──    │           │              │
│  └───────────┘         └───────────┘              │
│       │                      │                    │
│       │    Iterate until     │                    │
│       │    Critic approves   │                    │
│       ▼                      │                    │
│  ┌───────────┐              │                    │
│  │  Output   │ ◀────────────┘                    │
│  └───────────┘                                    │
│                                                    │
└────────────────────────────────────────────────────┘
```

**Implementation:**
```
Use a Generator-Critic pattern for this implementation:

1. Generator: Create the authentication middleware
2. Critic: Review for security issues, performance problems, and code quality
3. Generator: Address critic feedback
4. Repeat until Critic approves

Critic should be strict - only approve when truly ready for production.
```

---

## Advanced Patterns

### Ralph Wiggum Pattern

**Origin:** An emerging pattern for persistent iteration across multiple context windows.

**Core idea:** Loop through fresh context windows, with each iteration building on artifacts from the previous.

```
┌────────────────────────────────────────────────────────────┐
│                     Ralph Wiggum Loop                       │
│                                                            │
│  Window 1          Window 2          Window 3              │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐            │
│  │ Initial │  →   │Continue │  →   │Continue │  → ...     │
│  │  Work   │      │from file│      │from file│            │
│  └────┬────┘      └────┬────┘      └────┬────┘            │
│       │                │                │                  │
│       ▼                ▼                ▼                  │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐            │
│  │ progress│  →   │ progress│  →   │ progress│            │
│  │   .txt  │      │   .txt  │      │   .txt  │            │
│  └─────────┘      └─────────┘      └─────────┘            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Key elements:**
1. **Progress file:** Records what's been done
2. **Feature list:** Tracks completion status
3. **Git commits:** Create checkpoints
4. **Fresh context:** Each window starts clean

**Implementation:**
```
At the start of each session:
1. Read progress.txt to understand what was accomplished
2. Read feature_list.json to see what remains
3. Check git log for recent changes
4. Continue from where the last session ended

At the end of each session:
1. Update progress.txt with what was done
2. Update feature_list.json with completed items
3. Commit changes with descriptive message
```

### Gas Town Pattern

**Origin:** Multi-agent orchestration with 20-30 specialized agents.

**Named after:** Mad Max: Fury Road's Gas Town, where specialized workers each have distinct roles.

```
┌─────────────────────────────────────────────────────────────┐
│                      Gas Town Architecture                   │
│                                                             │
│                     ┌──────────────┐                        │
│                     │    MAYOR     │                        │
│                     │ (Orchestrator)│                        │
│                     └──────┬───────┘                        │
│                            │                                │
│           ┌────────────────┼────────────────┐               │
│           │                │                │               │
│           ▼                ▼                ▼               │
│    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│    │     CREW     │ │     CREW     │ │     CREW     │      │
│    │   (Workers)  │ │   (Workers)  │ │   (Workers)  │      │
│    └──────┬───────┘ └──────┬───────┘ └──────┬───────┘      │
│           │                │                │               │
│           ▼                ▼                ▼               │
│    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│    │   POLECATS   │ │   POLECATS   │ │   POLECATS   │      │
│    │ (Specialists)│ │ (Specialists)│ │ (Specialists)│      │
│    └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Roles:**
- **Mayor:** High-level orchestration and task assignment
- **Crew:** Worker agents that execute main tasks
- **Polecats:** Specialized agents for specific subtasks (testing, documentation, security review)

**Benefits:**
- Massive parallelization
- Specialized expertise per agent
- Fault isolation
- Scale to large projects

**When to use:**
- Large codebases
- Complex features requiring many subtasks
- Team-scale projects
- When context windows are limiting

---

## A2A Protocol: Agent-to-Agent Communication

### What Is A2A?

The **Agent-to-Agent (A2A) Protocol** is an emerging standard for how AI agents communicate with each other in multi-agent systems. While MCP handles agent-to-tool communication, A2A handles agent-to-agent coordination.

### Why A2A Matters

As systems grow more complex, single agents hit limitations:
- Context windows fill up
- Diverse expertise needed
- Parallel work opportunities
- Verification requires independence

A2A enables structured collaboration between specialized agents.

### A2A Protocol Flow

The following diagram illustrates how agents communicate in a multi-agent system:

```
┌─────────────────────────────────────────────────────────────────┐
│              AGENT-TO-AGENT COMMUNICATION                       │
│                                                                 │
│   ┌──────────────┐            ┌──────────────┐                 │
│   │   Planning   │  ────1───► │    Coding    │                 │
│   │    Agent     │   task     │    Agent     │                 │
│   └──────────────┘            └──────┬───────┘                 │
│          ▲                           │                          │
│          │                           │ 2 (code)                 │
│          │                           ▼                          │
│          │                    ┌──────────────┐                 │
│          │       4            │   Testing    │                 │
│          └───(report)─────────│    Agent     │                 │
│                               └──────┬───────┘                 │
│                                      │                          │
│                                      │ 3 (results)              │
│                                      ▼                          │
│                               ┌──────────────┐                 │
│                               │   Review     │                 │
│                               │    Agent     │                 │
│                               └──────────────┘                 │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Protocol Elements:                                     │   │
│   │                                                         │   │
│   │  1. Task Assignment: Specs, acceptance criteria         │   │
│   │  2. Artifact Handoff: Code with commit references       │   │
│   │  3. Verification Result: Pass/fail with evidence        │   │
│   │  4. Status Report: What's done, blocked, remaining      │   │
│   │                                                         │   │
│   │  Key Properties:                                        │   │
│   │  • Structured message format (typed)                    │   │
│   │  • Typed artifacts (code, tests, reports)               │   │
│   │  • Evidence chains for auditability                     │   │
│   │  • Explicit handoffs with context                       │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### A2A Communication Concepts

**Task Assignment**
An orchestrator agent assigns work to worker agents:
- Task ID and description
- Specification to follow
- Acceptance criteria
- Dependencies on other tasks

**Artifact Sharing**
Agents share work products:
- Code changes with commit references
- Test results with evidence
- Documentation with locations
- Analysis with supporting data

**Verification Requests**
Agents ask other agents to verify work:
- What artifact to verify
- What criteria to check
- What evidence to return

**Status Reporting**
Agents report progress to orchestrators:
- What's complete
- What's in progress
- What's blocked and why
- Evidence of completion

### A2A Design Principles

**Typed Messages:** Communications have explicit types (task_assignment, verification_request, status_report) so receivers know how to process them.

**Evidence Chains:** Each message references the artifacts and evidence it's based on, creating auditable chains.

**Explicit Handoffs:** When work transfers between agents, the handoff includes everything the receiving agent needs.

**Failure Handling:** Messages include how to handle failures—retry, escalate, or abandon.

### Current State of A2A

A2A is still emerging. Current multi-agent systems often use ad-hoc coordination. Standardization efforts are underway, and you'll see more structured A2A patterns in future tooling.

For now, when building multi-agent workflows:
- Think explicitly about what agents need to communicate
- Document the "protocol" even if informal
- Build in verification steps between agent handoffs
- Create audit trails of agent decisions

### A2A Protocol Examples

While A2A is emerging, here are concrete examples of structured agent-to-agent communication patterns you can implement today:

**Task Assignment Message (Orchestrator to Worker)**

```json
{
  "message_type": "task_assignment",
  "message_id": "task-2024-01-15-001",
  "timestamp": "2024-01-15T10:30:00Z",

  "from": {
    "agent_id": "orchestrator-main",
    "agent_type": "orchestrator"
  },

  "to": {
    "agent_id": "coder-auth",
    "agent_type": "implementation"
  },

  "task": {
    "id": "impl-auth-001",
    "title": "Implement password reset endpoint",
    "priority": "high",

    "specification": {
      "endpoint": "POST /api/auth/reset-password/request",
      "input": { "email": "string (required)" },
      "output": { "message": "string" },
      "behavior": "If email exists, send reset email. Always return same message to prevent enumeration."
    },

    "acceptance_criteria": [
      "Endpoint returns 200 for valid email format",
      "Endpoint returns 200 for invalid email (same response)",
      "Reset email sent only for existing users",
      "Rate limited to 3 requests per email per hour"
    ],

    "context_files": [
      "src/auth/login.ts (reference for patterns)",
      "src/lib/email/client.ts (email sending)",
      "PRD.md#phase-2-authentication"
    ],

    "constraints": [
      "Use existing email service, do not add new dependencies",
      "Follow error handling patterns in src/lib/api-handler.ts",
      "Must include unit tests"
    ]
  },

  "dependencies": [],

  "deadline": "2024-01-15T14:00:00Z",

  "response_expected": {
    "type": "task_completion",
    "include": ["files_changed", "tests_added", "commit_hash"]
  }
}
```

**Task Completion Message (Worker to Orchestrator)**

```json
{
  "message_type": "task_completion",
  "message_id": "completion-2024-01-15-001",
  "timestamp": "2024-01-15T12:45:00Z",
  "in_response_to": "task-2024-01-15-001",

  "from": {
    "agent_id": "coder-auth",
    "agent_type": "implementation"
  },

  "to": {
    "agent_id": "orchestrator-main",
    "agent_type": "orchestrator"
  },

  "status": "completed",

  "result": {
    "summary": "Password reset request endpoint implemented with rate limiting",

    "artifacts": {
      "files_created": [
        "src/app/api/auth/reset-password/request/route.ts",
        "src/auth/reset-tokens.ts",
        "tests/auth/reset-password.test.ts"
      ],
      "files_modified": [
        "prisma/schema.prisma (added PasswordResetToken model)"
      ],
      "commit_hash": "a1b2c3d4",
      "branch": "feature/password-reset"
    },

    "verification": {
      "tests_run": 8,
      "tests_passed": 8,
      "type_check": "passed",
      "lint": "passed"
    },

    "acceptance_criteria_status": [
      { "criterion": "Endpoint returns 200 for valid email format", "status": "met", "evidence": "test_valid_email_returns_200" },
      { "criterion": "Endpoint returns 200 for invalid email", "status": "met", "evidence": "test_invalid_email_same_response" },
      { "criterion": "Reset email sent only for existing users", "status": "met", "evidence": "test_email_sent_only_for_existing" },
      { "criterion": "Rate limited to 3 requests per email per hour", "status": "met", "evidence": "test_rate_limiting" }
    ]
  },

  "notes": [
    "Used existing Resend client for email delivery",
    "Token expiration set to 1 hour per security best practices",
    "Added index on token column for fast lookups"
  ],

  "ready_for": "verification_request"
}
```

**Verification Request (Orchestrator to Reviewer)**

```json
{
  "message_type": "verification_request",
  "message_id": "verify-2024-01-15-001",
  "timestamp": "2024-01-15T12:50:00Z",

  "from": {
    "agent_id": "orchestrator-main",
    "agent_type": "orchestrator"
  },

  "to": {
    "agent_id": "reviewer-security",
    "agent_type": "verification"
  },

  "artifact": {
    "type": "code_change",
    "commit": "a1b2c3d4",
    "branch": "feature/password-reset",
    "files": [
      "src/app/api/auth/reset-password/request/route.ts",
      "src/auth/reset-tokens.ts"
    ]
  },

  "verification_scope": {
    "focus_areas": [
      "security",
      "authentication"
    ],

    "specific_checks": [
      "No user enumeration vulnerability",
      "Token generation uses cryptographically secure random",
      "Rate limiting cannot be bypassed",
      "No sensitive data in logs or responses"
    ],

    "out_of_scope": [
      "Performance optimization",
      "UI implementation"
    ]
  },

  "context": {
    "original_task": "task-2024-01-15-001",
    "requirements_doc": "PRD.md#phase-2-authentication",
    "implementation_notes": "Uses existing Resend client, 1-hour token expiration"
  },

  "response_expected": {
    "type": "verification_result",
    "include": ["passed", "findings", "recommendations"]
  }
}
```

**Verification Result (Reviewer to Orchestrator)**

```json
{
  "message_type": "verification_result",
  "message_id": "result-2024-01-15-001",
  "timestamp": "2024-01-15T13:15:00Z",
  "in_response_to": "verify-2024-01-15-001",

  "from": {
    "agent_id": "reviewer-security",
    "agent_type": "verification"
  },

  "to": {
    "agent_id": "orchestrator-main",
    "agent_type": "orchestrator"
  },

  "verification": {
    "passed": true,
    "confidence": "high",

    "checks_performed": [
      {
        "check": "User enumeration prevention",
        "status": "passed",
        "evidence": "Same response returned for existing and non-existing emails",
        "files_reviewed": ["src/app/api/auth/reset-password/request/route.ts:23-35"]
      },
      {
        "check": "Cryptographically secure token",
        "status": "passed",
        "evidence": "Uses crypto.randomBytes(32), appropriate for security tokens",
        "files_reviewed": ["src/auth/reset-tokens.ts:12"]
      },
      {
        "check": "Rate limiting implementation",
        "status": "passed",
        "evidence": "Uses proven rate-limit middleware, 3 requests/email/hour",
        "files_reviewed": ["src/app/api/auth/reset-password/request/route.ts:8-10"]
      },
      {
        "check": "No sensitive data exposure",
        "status": "passed",
        "evidence": "Logs sanitized, no PII in responses",
        "files_reviewed": ["src/app/api/auth/reset-password/request/route.ts"]
      }
    ],

    "findings": [],

    "recommendations": [
      {
        "severity": "info",
        "description": "Consider adding audit logging for password reset attempts",
        "location": "src/app/api/auth/reset-password/request/route.ts",
        "action_required": false
      }
    ]
  },

  "summary": "Security review passed. Implementation follows best practices for password reset flows. No vulnerabilities found.",

  "ready_for": "merge_approval"
}
```

**Implementing A2A in Practice**

You don't need a formal protocol to use these patterns. Here's how to apply them with Claude Code today:

```markdown
# Multi-Agent Workflow Prompt

## Setting Up Agent Roles

"For this feature implementation, we'll use a multi-agent approach:

1. **Planning Agent** (this conversation): Creates specifications
2. **Implementation Agent** (subagent): Writes code
3. **Review Agent** (subagent): Verifies work

I'll coordinate between agents using structured handoff documents."

## Handoff to Implementation

"Create a subagent for implementation with this context:

TASK ASSIGNMENT:
- Task: Implement password reset endpoint
- Files to create: [list]
- Patterns to follow: See src/auth/login.ts
- Acceptance criteria: [list]
- Constraints: Use existing email service, include tests

Return when complete with:
- List of files changed
- Test results
- Any blockers encountered"

## Handoff to Review

"Create a subagent for security review with this context:

VERIFICATION REQUEST:
- Review: Password reset implementation
- Focus: Security vulnerabilities
- Check for: User enumeration, token security, rate limiting
- Do NOT: Review performance or suggest refactors

Return with:
- Pass/fail status
- Any security findings
- Recommendations"
```

The key insight: even without formal A2A tooling, structured communication between agents produces better results than ad-hoc coordination.

---

## Long-Running Agent Patterns

### The Challenge of Long-Running Tasks

Some tasks take longer than a single context window allows:
- Large refactoring projects
- Multi-phase feature implementations
- Codebase-wide changes
- Complex debugging sessions

Long-running patterns solve context continuity.

### Persistent State Patterns

**Progress Files**
Track what's done and what remains:
- `progress.md` with completed items
- `feature-list.json` with status per item
- Timestamps for resumability

**Checkpoint Commits**
Create git commits at meaningful points:
- Each commit represents stable progress
- Can resume from any checkpoint
- Provides rollback points

**Session Handoff Documents**
When ending a session, create a document with:
- What was accomplished
- Current state of work
- What should be done next
- Any blockers or decisions pending

### Error Recovery Patterns

**Graceful Degradation**
When agents encounter errors:
- Capture the error state
- Save progress so far
- Create a clear error report
- Don't lose work done before the error

**Retry Strategies**
For transient failures:
- Exponential backoff for API errors
- Alternative approaches for logical failures
- Human escalation after retry limits

**Rollback Capability**
Design for recoverability:
- Checkpoint before risky operations
- Keep undo information
- Test rollback paths

### Session Continuity Across Context Windows

When you must split work across sessions:

**End of Session:**
1. Summarize progress in a file
2. Commit current state
3. List remaining work
4. Note any context the next session needs

**Start of Session:**
1. Read progress files
2. Review recent commits
3. Load context documents
4. Resume from documented state

This is the essence of the Ralph Wiggum pattern—using files and commits to bridge context windows.

---

## ReACT Pattern

**ReACT** = **Re**asoning + **Act**ing, interleaved. [^2]

[^2]: Yao, S., et al. "ReAct: Synergizing Reasoning and Acting in Language Models." ICLR 2023. arXiv:2210.03629

Instead of thinking completely, then acting, the agent alternates:

```
Thought: I need to understand the auth system
Action: Read src/auth/middleware.ts
Observation: The middleware checks JWT tokens...

Thought: Now I understand auth. I should check how it handles expiry
Action: Read src/auth/token.ts
Observation: Tokens expire after 1 hour...

Thought: I have enough context. Let me implement the refresh flow
Action: Edit src/auth/refresh.ts
Observation: File created successfully

Thought: Let me verify this works
Action: Run tests
Observation: All tests pass
```

**Benefits:**
- Natural reasoning flow
- Corrects course quickly
- Transparent decision-making
- Easier to debug agent behavior

---

## Choosing the Right Pattern

| Situation | Recommended Pattern |
|-----------|-------------------|
| Simple task, focused scope | Single Agent |
| Research + implementation | Subagents |
| Needs independent review | Generator-Critic |
| Multiple parallel tasks | Multi-Claude (git worktrees) |
| Long-running project | Ralph Wiggum |
| Large scale, many features | Gas Town |

---

## Key Takeaways

1. **Understand ACE vs AEE** — Human command interface vs agent execution environment
2. **The agent loop** (gather → act → verify → repeat) is the foundation of all patterns
3. **MCP** provides standardized tool integration for agents
4. **Design tools well** — Single responsibility, efficient output, composable, clear boundaries
5. **Subagents** solve context window limits through parallelization
6. **Generator-Critic** improves quality by separating creation from validation
7. **Multi-Claude workflows** enable independent verification
8. **A2A protocol** enables structured agent-to-agent communication
9. **Ralph Wiggum** handles long-running tasks across context windows
10. **Gas Town** scales to large projects with specialized agent teams
11. **Plan for errors** — Graceful degradation, retry strategies, rollback capability

---

## Practical Exercise: Multi-Agent Implementation

Design a multi-agent system for a feature:

### The Feature
Implement user notifications (email + in-app) for a web application.

### Your Task

1. **Design the agent architecture:**
   - What agents do you need?
   - What is each agent's responsibility?
   - How do they communicate?

2. **Plan the workflow:**
   - What order do agents work in?
   - What artifacts do they share?
   - How is work verified?

3. **Write the orchestration prompt:**
   - How would you instruct Claude Code to execute this?

### Reflection
- Which pattern(s) did you use?
- Why did you choose them?
- What would change for a larger feature?

---

## Next Step

[Step 09: Security in AI-Assisted Development](09-security-in-ai-assisted-development.md) — Understand the security implications of AI-generated code.

---

## Sources

### Primary References

1. **Anthropic Engineering Blog**
   - "Building Agents with the Claude Agent SDK" (2025) - https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
   - "Building Effective Agents" (2025) - https://www.anthropic.com/engineering/building-effective-agents
   - "Writing Tools for Agents" (2025) - https://www.anthropic.com/engineering/writing-tools-for-agents
   - "Effective Harnesses for Long-Running Agents" (2025) - https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

2. **Academic Papers**
   - Yao, S., et al. "ReAct: Synergizing Reasoning and Acting in Language Models." ICLR 2023. arXiv:2210.03629
   - Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)
   - "A Survey on Code Generation with LLM-based Agents." arXiv:2508.00083 (2025)

3. **Stanford CS146S: The Modern Software Developer**
   - Week 2: The Anatomy of Coding Agents
   - Week 4: Coding Agent Patterns
   - Course materials on ACE/AEE workbench architecture

4. **Specifications and Standards**
   - Model Context Protocol Specification: https://modelcontextprotocol.io/
   - AGENTS.md Specification: https://agents.md

---

## Further Reading

1. **"Building Effective Agents"** - Anthropic's comprehensive guide to agent design patterns and tool integration principles. Essential reading for understanding the agent loop and tool design.
   https://www.anthropic.com/engineering/building-effective-agents

2. **"ReAct: Synergizing Reasoning and Acting in Language Models"** (arXiv:2210.03629) - The foundational paper introducing the ReACT pattern of interleaved reasoning and acting.
   https://arxiv.org/abs/2210.03629

3. **Model Context Protocol Documentation** - Official specification for MCP, including server implementation guides and best practices.
   https://modelcontextprotocol.io/

4. **SASE Paper: Structured Agentic Software Engineering** (arXiv:2509.06216v2) - Introduces the ACE/AEE workbench concepts and A2A protocol foundations.
   https://arxiv.org/abs/2509.06216

5. **"Effective Harnesses for Long-Running Agents"** - Anthropic's guidance on managing agents across multiple sessions and context windows.
   https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
