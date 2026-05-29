---
name: webmcp-chrome-devtools-quickstart
description: AI-driven browser automation using Chrome DevTools MCP with WebMCP tools for structured, token-efficient interactions
triggers:
  - how do I set up WebMCP tools with Chrome DevTools
  - integrate AI agents with browser automation using WebMCP
  - create structured browser tools for AI instead of screenshots
  - register JavaScript functions as AI-callable tools
  - connect Claude or Cursor to browser automation with MCP
  - reduce token usage in browser automation workflows
  - implement WebMCP tool discovery and execution
  - build AI-driven web interactions with Chrome DevTools MCP
---

# WebMCP Chrome DevTools Quickstart

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

This skill teaches AI agents how to use WebMCP with Chrome DevTools MCP to enable structured, token-efficient browser automation. Instead of screenshot-based workflows, WebMCP lets you register JavaScript functions as AI-callable tools, reducing token usage by up to 89%.

## What This Project Does

WebMCP turns your website's JavaScript functions into AI-callable tools using the Model Context Protocol (MCP). The Chrome DevTools MCP server connects to Chrome via the Chrome DevTools Protocol (CDP) and provides:

1. **26 browser automation tools** (navigation, interaction, inspection, tab management)
2. **WebMCP tool discovery** (`list_webmcp_tools`)
3. **WebMCP tool execution** (`call_webmcp_tool`)

**Architecture:**
```
AI Client → Chrome DevTools MCP → Chrome (CDP) → Your Website (navigator.modelContext)
```

## Installation

### 1. Clone and Run Demo

```bash
git clone https://github.com/WebMCP-org/chrome-devtools-quickstart.git
cd chrome-devtools-quickstart
npm install
npm run dev
# Opens http://localhost:5173
```

### 2. Add MCP Server to AI Client

**Claude Code:**
```bash
claude mcp add chrome-devtools npx @mcp-b/chrome-devtools-mcp@latest
claude mcp add --transport http webmcp-docs https://docs.mcp-b.ai/mcp
```

**Cursor (`.cursor/mcp.json`):**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@mcp-b/chrome-devtools-mcp@latest"]
    },
    "webmcp-docs": {
      "url": "https://docs.mcp-b.ai/mcp"
    }
  }
}
```

**Claude Desktop (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@mcp-b/chrome-devtools-mcp@latest"]
    },
    "webmcp-docs": {
      "url": "https://docs.mcp-b.ai/mcp"
    }
  }
}
```

**Windsurf (`mcp_config.json`):**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@mcp-b/chrome-devtools-mcp@latest"]
    },
    "webmcp-docs": {
      "command": "npx",
      "args": ["mcp-remote", "https://docs.mcp-b.ai/mcp"]
    }
  }
}
```

### 3. Install in Your Own Project

```bash
npm install @mcp-b/global
```

```javascript
// Must be imported FIRST
import '@mcp-b/global';

// Now navigator.modelContext is available
```

## Registering WebMCP Tools

### Basic Tool (No Parameters)

```javascript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: "get_page_title",
  description: "Returns the current page title",
  inputSchema: {
    type: "object",
    properties: {}
  },
  async execute() {
    return {
      content: [
        { type: "text", text: document.title }
      ]
    };
  }
});
```

### Tool with Parameters

```javascript
import '@mcp-b/global';

let counter = 0;

navigator.modelContext.registerTool({
  name: "set_counter",
  description: "Sets the counter to the desired value",
  inputSchema: {
    type: "object",
    properties: {
      newCounterValue: {
        type: "number",
        description: "The number to set the counter to"
      }
    },
    required: ["newCounterValue"]
  },
  async execute(args) {
    counter = args.newCounterValue;
    // Update UI
    document.getElementById('counter').textContent = counter;
    
    return {
      content: [
        { type: "text", text: `Counter is now ${counter}` }
      ]
    };
  }
});
```

### Tool with Complex Input Schema

```javascript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: "create_calendar_event",
  description: "Creates a new calendar event",
  inputSchema: {
    type: "object",
    properties: {
      title: {
        type: "string",
        description: "Event title"
      },
      date: {
        type: "string",
        description: "Event date in YYYY-MM-DD format"
      },
      startTime: {
        type: "string",
        description: "Start time in HH:MM format"
      },
      endTime: {
        type: "string",
        description: "End time in HH:MM format"
      },
      description: {
        type: "string",
        description: "Event description (optional)"
      }
    },
    required: ["title", "date", "startTime", "endTime"]
  },
  async execute(args) {
    const event = {
      id: crypto.randomUUID(),
      ...args
    };
    
    // Save to state
    window.calendarEvents = window.calendarEvents || [];
    window.calendarEvents.push(event);
    
    // Update UI
    renderCalendar();
    
    return {
      content: [
        { 
          type: "text", 
          text: `Created event "${event.title}" on ${event.date} from ${event.startTime} to ${event.endTime}`
        }
      ]
    };
  }
});
```

### Tool with Error Handling

```javascript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: "toggle_theme",
  description: "Toggles between light and dark theme",
  inputSchema: {
    type: "object",
    properties: {}
  },
  async execute() {
    try {
      const body = document.body;
      const currentTheme = body.getAttribute('data-theme') || 'light';
      const newTheme = currentTheme === 'light' ? 'dark' : 'light';
      
      body.setAttribute('data-theme', newTheme);
      
      return {
        content: [
          { type: "text", text: `Theme switched to ${newTheme} mode` }
        ]
      };
    } catch (error) {
      return {
        content: [
          { type: "text", text: `Error toggling theme: ${error.message}` }
        ],
        isError: true
      };
    }
  }
});
```

## Key Chrome DevTools MCP Tools

### Navigation

```javascript
// Navigate to a page
await use_mcp_tool("chrome-devtools", "navigate_page", {
  url: "http://localhost:5173"
});

// Go back
await use_mcp_tool("chrome-devtools", "go_back", {});

// Refresh
await use_mcp_tool("chrome-devtools", "refresh", {});
```

### WebMCP Tool Discovery

```javascript
// List all WebMCP tools on current page
const result = await use_mcp_tool("chrome-devtools", "list_webmcp_tools", {});

// Result:
// {
//   content: [
//     {
//       type: "text",
//       text: JSON.stringify([
//         {
//           name: "get_counter",
//           description: "Returns the current counter value",
//           inputSchema: { type: "object", properties: {} }
//         },
//         {
//           name: "set_counter",
//           description: "Sets the counter to the desired value",
//           inputSchema: {
//             type: "object",
//             properties: {
//               newCounterValue: { type: "number", description: "..." }
//             }
//           }
//         }
//       ])
//     }
//   ]
// }
```

### WebMCP Tool Execution

```javascript
// Call a WebMCP tool
const result = await use_mcp_tool("chrome-devtools", "call_webmcp_tool", {
  name: "set_counter",
  arguments: {
    newCounterValue: 42
  }
});

// Result:
// {
//   content: [
//     { type: "text", text: "Counter is now 42" }
//   ]
// }
```

### Screenshots and Inspection

```javascript
// Take screenshot
const screenshot = await use_mcp_tool("chrome-devtools", "take_screenshot", {});

// Evaluate JavaScript
const result = await use_mcp_tool("chrome-devtools", "evaluate_script", {
  script: "document.querySelectorAll('.event-card').length"
});
```

### Interaction

```javascript
// Click element
await use_mcp_tool("chrome-devtools", "click", {
  selector: "#submit-button"
});

// Fill input
await use_mcp_tool("chrome-devtools", "fill", {
  selector: "#email-input",
  value: "user@example.com"
});

// Hover
await use_mcp_tool("chrome-devtools", "hover", {
  selector: ".tooltip-trigger"
});
```

## Common Patterns

### Complete AI Workflow

```javascript
// 1. Navigate to page
await use_mcp_tool("chrome-devtools", "navigate_page", {
  url: "http://localhost:5173"
});

// 2. Discover available tools
const tools = await use_mcp_tool("chrome-devtools", "list_webmcp_tools", {});
console.log("Available tools:", tools);

// 3. Call a tool
const result = await use_mcp_tool("chrome-devtools", "call_webmcp_tool", {
  name: "set_counter",
  arguments: { newCounterValue: 42 }
});

// 4. Verify with another tool call
const verification = await use_mcp_tool("chrome-devtools", "call_webmcp_tool", {
  name: "get_counter",
  arguments: {}
});
```

### Multi-Step Form Interaction

```javascript
import '@mcp-b/global';

// Register form submission tool
navigator.modelContext.registerTool({
  name: "submit_contact_form",
  description: "Submits the contact form with user details",
  inputSchema: {
    type: "object",
    properties: {
      name: { type: "string", description: "User's full name" },
      email: { type: "string", description: "User's email address" },
      message: { type: "string", description: "Contact message" }
    },
    required: ["name", "email", "message"]
  },
  async execute(args) {
    document.getElementById('name').value = args.name;
    document.getElementById('email').value = args.email;
    document.getElementById('message').value = args.message;
    
    // Trigger validation
    const form = document.getElementById('contact-form');
    const isValid = form.checkValidity();
    
    if (!isValid) {
      return {
        content: [{ type: "text", text: "Form validation failed" }],
        isError: true
      };
    }
    
    form.submit();
    
    return {
      content: [{ type: "text", text: "Contact form submitted successfully" }]
    };
  }
});
```

### State Management Tool

```javascript
import '@mcp-b/global';

// Register state getter
navigator.modelContext.registerTool({
  name: "get_app_state",
  description: "Returns the current application state",
  inputSchema: {
    type: "object",
    properties: {}
  },
  async execute() {
    const state = {
      user: window.currentUser || null,
      theme: document.body.getAttribute('data-theme'),
      activeView: window.location.hash.slice(1),
      notifications: window.notifications?.length || 0
    };
    
    return {
      content: [
        { type: "text", text: JSON.stringify(state, null, 2) }
      ]
    };
  }
});

// Register state setter
navigator.modelContext.registerTool({
  name: "update_app_state",
  description: "Updates application state",
  inputSchema: {
    type: "object",
    properties: {
      theme: { type: "string", enum: ["light", "dark"] },
      activeView: { type: "string" }
    }
  },
  async execute(args) {
    if (args.theme) {
      document.body.setAttribute('data-theme', args.theme);
    }
    
    if (args.activeView) {
      window.location.hash = args.activeView;
    }
    
    return {
      content: [{ type: "text", text: "State updated successfully" }]
    };
  }
});
```

### Tool with DOM Manipulation

```javascript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: "add_todo_item",
  description: "Adds a new todo item to the list",
  inputSchema: {
    type: "object",
    properties: {
      text: {
        type: "string",
        description: "The todo item text"
      },
      priority: {
        type: "string",
        enum: ["low", "medium", "high"],
        description: "Priority level"
      }
    },
    required: ["text"]
  },
  async execute(args) {
    const todoList = document.getElementById('todo-list');
    const todoItem = document.createElement('li');
    todoItem.className = `todo-item priority-${args.priority || 'medium'}`;
    todoItem.innerHTML = `
      <span class="todo-text">${args.text}</span>
      <button class="delete-btn">Delete</button>
    `;
    
    todoList.appendChild(todoItem);
    
    // Add event listener
    todoItem.querySelector('.delete-btn').addEventListener('click', () => {
      todoItem.remove();
    });
    
    return {
      content: [
        { 
          type: "text", 
          text: `Added todo: "${args.text}" with ${args.priority || 'medium'} priority` 
        }
      ]
    };
  }
});
```

## Running Benchmarks

Compare token usage between screenshot-based and WebMCP-based workflows:

```bash
# Add your API key
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env

# Simple benchmark (counter app)
npm run benchmark:simple:direct

# Complex benchmark (calendar app)
npm run benchmark:complex:direct
```

**Expected Results:**
- Simple task: 89% token reduction
- Complex task: 77% token reduction

## Configuration

### Custom Chrome Launch Options

Create `chrome-devtools.config.js`:

```javascript
export default {
  chromeOptions: {
    headless: false,
    userDataDir: './chrome-profile',
    args: [
      '--window-size=1920,1080',
      '--disable-blink-features=AutomationControlled'
    ]
  },
  port: 9222
};
```

### Tool Registration Best Practices

```javascript
import '@mcp-b/global';

// Wait for DOM ready
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', registerTools);
} else {
  registerTools();
}

function registerTools() {
  // Register all tools here
  navigator.modelContext.registerTool({
    name: "my_tool",
    description: "Clear, specific description of what this tool does",
    inputSchema: {
      type: "object",
      properties: {
        // Use descriptive property names
        // Add helpful descriptions
        // Include type constraints
      },
      required: [] // Be explicit about required fields
    },
    async execute(args) {
      try {
        // Validate inputs
        // Perform action
        // Return structured response
        return {
          content: [{ type: "text", text: "Success message" }]
        };
      } catch (error) {
        return {
          content: [{ type: "text", text: error.message }],
          isError: true
        };
      }
    }
  });
}
```

## Troubleshooting

### `navigator.modelContext is undefined`

**Cause:** `@mcp-b/global` not imported or imported after tool registration.

**Solution:**
```javascript
// CORRECT: Import first
import '@mcp-b/global';
import './my-tools.js';

// WRONG: Tool imports before @mcp-b/global
import './my-tools.js';
import '@mcp-b/global';
```

### No tools found by `list_webmcp_tools`

**Cause:** Tools not registered yet or page not fully loaded.

**Solution:**
```javascript
// Wait for page load
window.addEventListener('load', () => {
  navigator.modelContext.registerTool({
    // ...
  });
});

// Or check in AI workflow
await use_mcp_tool("chrome-devtools", "evaluate_script", {
  script: "typeof navigator.modelContext !== 'undefined' && navigator.modelContext.tools.size"
});
```

### Tool execution fails with `isError: true`

**Debug by inspecting console:**
```javascript
// In AI workflow
const consoleOutput = await use_mcp_tool("chrome-devtools", "evaluate_script", {
  script: `
    const errors = [];
    console.error = (...args) => errors.push(args.join(' '));
    // ... trigger tool call ...
    JSON.stringify(errors);
  `
});
```

### Chrome connection fails

**Cause:** Chrome not running or firewall blocking port 9222.

**Solution:**
```bash
# Launch Chrome manually with remote debugging
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-debug

# Or check if port is in use
lsof -i :9222
```

### Hot reload breaks tool registration

**Cause:** Vite HMR doesn't re-run `@mcp-b/global` initialization.

**Solution:**
```javascript
// Force full reload for tool changes
if (import.meta.hot) {
  import.meta.hot.accept(() => {
    window.location.reload();
  });
}
```

## Advanced Usage

### Multiple Tools in One File

```javascript
import '@mcp-b/global';

const tools = [
  {
    name: "get_user_info",
    description: "Get current user information",
    inputSchema: { type: "object", properties: {} },
    async execute() {
      return {
        content: [{ type: "text", text: JSON.stringify(window.currentUser) }]
      };
    }
  },
  {
    name: "logout",
    description: "Log out the current user",
    inputSchema: { type: "object", properties: {} },
    async execute() {
      window.currentUser = null;
      localStorage.removeItem('authToken');
      return {
        content: [{ type: "text", text: "Logged out successfully" }]
      };
    }
  }
];

tools.forEach(tool => navigator.modelContext.registerTool(tool));
```

### Async Tool with External API

```javascript
import '@mcp-b/global';

navigator.modelContext.registerTool({
  name: "fetch_weather",
  description: "Fetches weather data for a city",
  inputSchema: {
    type: "object",
    properties: {
      city: { type: "string", description: "City name" }
    },
    required: ["city"]
  },
  async execute(args) {
    try {
      const apiKey = import.meta.env.VITE_WEATHER_API_KEY;
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${args.city}&appid=${apiKey}`
      );
      
      if (!response.ok) {
        throw new Error(`Weather API returned ${response.status}`);
      }
      
      const data = await response.json();
      
      return {
        content: [
          { 
            type: "text", 
            text: `Weather in ${args.city}: ${data.weather[0].description}, ${Math.round(data.main.temp - 273.15)}°C` 
          }
        ]
      };
    } catch (error) {
      return {
        content: [{ type: "text", text: `Failed to fetch weather: ${error.message}` }],
        isError: true
      };
    }
  }
});
```

## Resources

- [WebMCP Documentation](https://docs.mcp-b.ai)
- [Chrome DevTools MCP Package](https://docs.mcp-b.ai/packages/chrome-devtools-mcp)
- [@mcp-b/global NPM](https://www.npmjs.com/package/@mcp-b/global)
- [MCP-B Chrome Extension](https://chromewebstore.google.com/detail/mcp-b-extension/daohopfhkdelnpemnhlekblnikhdhfa)
- [WebMCP Examples](https://github.com/WebMCP-org/examples)
- [Live Demo](https://webmcp.sh)
- [Discord Community](https://discord.gg/ZnHG4csJRB)
