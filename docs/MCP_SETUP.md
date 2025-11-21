# MCP Server Setup Guide

This guide walks you through setting up the Model Context Protocol (MCP) servers required for this workflow. MCP allows Claude Code to interact with external services like Linear, perform web searches, and use enhanced reasoning.

## Table of Contents

1. [What is MCP?](#what-is-mcp)
2. [Required: Linear MCP](#required-linear-mcp)
3. [Recommended: Perplexity MCP](#recommended-perplexity-mcp)
4. [Recommended: Sequential Thinking MCP](#recommended-sequential-thinking-mcp)
5. [Optional: Playwright MCP](#optional-playwright-mcp)
6. [Complete Configuration Example](#complete-configuration-example)
7. [Verification and Testing](#verification-and-testing)
8. [Troubleshooting](#troubleshooting)

---

## What is MCP?

**Model Context Protocol (MCP)** is a way for Claude Code to connect to external tools and services. Think of it like installing apps on your phone—each MCP server gives Claude new capabilities.

**Without MCP servers**, Claude Code can read/write files and run terminal commands.

**With MCP servers**, Claude Code can:
- Create and manage Linear tickets
- Search the web for current information
- Use enhanced reasoning for complex problems
- Control a web browser for testing

### How MCP Works

```
You → Claude Code → MCP Server → External Service
                         ↓
                    Linear, Perplexity, etc.
```

When you run a command like `/epic-planning`, Claude Code uses the Linear MCP server to create tickets directly in Linear.

---

## Required: Linear MCP

Linear MCP is **required** for this workflow. It allows Claude Code to create and manage issues, projects, and comments in Linear.

### Step 1: Get Your Linear API Key

1. Log in to [Linear](https://linear.app)
2. Click your profile icon (bottom left)
3. Go to **Settings** → **API**
4. Click **Create Key**
5. Give it a name like "Claude Code"
6. Copy the key (starts with `lin_api_...`)

**Important**: Save this key securely. You won't be able to see it again.

### Step 2: Set the Environment Variable

**On Mac/Linux:**
```bash
# Add to your shell profile (~/.zshrc or ~/.bashrc)
export LINEAR_API_KEY="lin_api_your_key_here"

# Reload your profile
source ~/.zshrc
```

**On Windows (PowerShell):**
```powershell
[Environment]::SetEnvironmentVariable("LINEAR_API_KEY", "lin_api_your_key_here", "User")
# Restart PowerShell
```

### Step 3: Install Linear MCP

```bash
claude mcp add linear --scope user
```

When prompted, confirm the installation.

### Step 4: Verify Connection

```bash
# Check MCP status
claude mcp list
```

You should see `linear` in the list of active servers.

### Alternative: Manual Configuration

If the automatic installation doesn't work, you can configure manually.

**Find your config file:**
- **Mac**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux**: `~/.config/claude/claude_desktop_config.json`

**Add this configuration:**

```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-linear"],
      "env": {
        "LINEAR_API_KEY": "${LINEAR_API_KEY}"
      }
    }
  }
}
```

**For Windows**, use this format instead:
```json
{
  "mcpServers": {
    "linear": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-linear"],
      "env": {
        "LINEAR_API_KEY": "${LINEAR_API_KEY}"
      }
    }
  }
}
```

---

## Recommended: Perplexity MCP

Perplexity MCP allows Claude Code to search the web for current information. This is valuable when:
- Researching best practices during discovery
- Looking up framework documentation
- Finding solutions to common problems

### Step 1: Get Your Perplexity API Key

1. Go to [perplexity.ai](https://www.perplexity.ai/)
2. Create an account or log in
3. Navigate to API settings
4. Generate an API key
5. Copy the key

**Cost Note**: Perplexity API has usage-based pricing. For typical PM workflows, expect $5-20/month depending on usage.

### Step 2: Set the Environment Variable

**On Mac/Linux:**
```bash
export PERPLEXITY_API_KEY="pplx-your-key-here"
```

Add to your shell profile (`~/.zshrc` or `~/.bashrc`) for permanence.

**On Windows:**
```powershell
[Environment]::SetEnvironmentVariable("PERPLEXITY_API_KEY", "pplx-your-key-here", "User")
```

### Step 3: Install Perplexity MCP

```bash
claude mcp add perplexity --scope user
```

### Alternative: Manual Configuration

Add to your config file:

```json
{
  "mcpServers": {
    "perplexity": {
      "command": "npx",
      "args": ["-y", "perplexity-mcp"],
      "env": {
        "PERPLEXITY_API_KEY": "${PERPLEXITY_API_KEY}"
      }
    }
  }
}
```

---

## Recommended: Sequential Thinking MCP

Sequential Thinking MCP enhances Claude's reasoning for complex, multi-step problems. It's particularly useful during:
- Architecture planning in `/discovery`
- Complex bug analysis in `/implementation`
- Security reasoning in `/security_review`

**No API key required** — this runs locally.

### Installation

```bash
claude mcp add sequential-thinking --scope user
```

### Alternative: Manual Configuration

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

---

## Optional: Playwright MCP

Playwright MCP allows Claude Code to control a web browser for visual testing and E2E testing. This is used by the `/testing` command for browser-based tests.

**When to install:**
- If your project has a web UI that needs testing
- If you want automated visual regression testing
- If you're building user-facing features

**Skip if:**
- You're building backend-only services
- You don't need E2E testing
- You prefer manual browser testing

### Installation

```bash
claude mcp add playwright --scope user
```

### Alternative: Manual Configuration

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"]
    }
  }
}
```

---

## Complete Configuration Example

Here's a complete config file with all recommended MCP servers:

### Mac/Linux Configuration

**File**: `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac) or `~/.config/claude/claude_desktop_config.json` (Linux)

```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-linear"],
      "env": {
        "LINEAR_API_KEY": "${LINEAR_API_KEY}"
      }
    },
    "perplexity": {
      "command": "npx",
      "args": ["-y", "perplexity-mcp"],
      "env": {
        "PERPLEXITY_API_KEY": "${PERPLEXITY_API_KEY}"
      }
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"]
    }
  }
}
```

### Windows Configuration

**File**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "linear": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-linear"],
      "env": {
        "LINEAR_API_KEY": "${LINEAR_API_KEY}"
      }
    },
    "perplexity": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "perplexity-mcp"],
      "env": {
        "PERPLEXITY_API_KEY": "${PERPLEXITY_API_KEY}"
      }
    },
    "sequential-thinking": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "playwright": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@anthropic/mcp-playwright"]
    }
  }
}
```

### Environment Variables Setup

Create or update your shell profile with all required variables:

**Mac/Linux** (`~/.zshrc` or `~/.bashrc`):
```bash
# Claude Code API Keys
export ANTHROPIC_API_KEY="sk-ant-your-anthropic-key"
export LINEAR_API_KEY="lin_api_your-linear-key"
export PERPLEXITY_API_KEY="pplx-your-perplexity-key"
```

**Windows** (run in PowerShell as Administrator):
```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-ant-your-anthropic-key", "User")
[Environment]::SetEnvironmentVariable("LINEAR_API_KEY", "lin_api_your-linear-key", "User")
[Environment]::SetEnvironmentVariable("PERPLEXITY_API_KEY", "pplx-your-perplexity-key", "User")
```

---

## Verification and Testing

After setting up MCP servers, verify everything works:

### Step 1: Check MCP Server Status

```bash
claude mcp list
```

Expected output:
```
Active MCP Servers:
- linear (running)
- perplexity (running)
- sequential-thinking (running)
- playwright (running)
```

### Step 2: Test Linear Connection

Start Claude Code and run:
```
Can you list my Linear teams?
```

Claude should use `mcp__linear-server__list_teams` and show your Linear teams.

### Step 3: Test Perplexity Connection

```
Search the web for "best practices for REST API design 2024"
```

Claude should use `mcp__perplexity__search` and return current information.

### Step 4: Test Sequential Thinking

```
Think through the architectural implications of adding real-time notifications to a monolithic application.
```

Claude should use `mcp__sequential-thinking__sequentialthinking` for structured reasoning.

---

## Troubleshooting

### MCP Server Won't Start

**Symptoms**: Server shows as "not running" or "error" in `claude mcp list`

**Solutions**:

1. **Check Node.js is installed**:
   ```bash
   node --version
   ```
   Must be v18 or higher.

2. **Clear npx cache**:
   ```bash
   npx clear-npx-cache
   ```

3. **Try manual npx run**:
   ```bash
   npx -y @modelcontextprotocol/server-linear
   ```
   This should start the server. If it fails, you'll see the actual error.

4. **Check environment variables**:
   ```bash
   echo $LINEAR_API_KEY
   echo $PERPLEXITY_API_KEY
   ```
   Should show your keys, not empty strings.

### "Invalid API Key" Errors

**For Linear**:
- Verify key starts with `lin_api_`
- Check for extra spaces when copying
- Try generating a new key

**For Perplexity**:
- Verify key starts with `pplx-`
- Check your Perplexity account has API access enabled
- Verify billing is set up if on paid tier

### Windows-Specific Issues

**Problem**: MCP servers don't start on Windows

**Solution**: Ensure you're using the Windows-specific config format with `cmd /c npx`:
```json
{
  "command": "cmd",
  "args": ["/c", "npx", "-y", "package-name"]
}
```

**Problem**: Environment variables not recognized

**Solution**:
1. Set variables using System Properties (not just PowerShell export)
2. Restart your terminal completely
3. Verify with `echo $env:LINEAR_API_KEY` in PowerShell

### Config File Not Found

If Claude Code can't find your config:

1. **Find the correct path**:
   ```bash
   # Mac
   ls ~/Library/Application\ Support/Claude/

   # Linux
   ls ~/.config/claude/

   # Windows (PowerShell)
   dir $env:APPDATA\Claude\
   ```

2. **Create the directory and file if missing**:
   ```bash
   # Mac
   mkdir -p ~/Library/Application\ Support/Claude
   touch ~/Library/Application\ Support/Claude/claude_desktop_config.json
   ```

### Servers Disconnect Randomly

**Cause**: Network issues or server timeouts

**Solutions**:
1. Restart Claude Code
2. Check your internet connection
3. Verify API keys haven't expired
4. Try reinstalling the problematic server:
   ```bash
   claude mcp remove linear
   claude mcp add linear --scope user
   ```

---

## Security Best Practices

### API Key Safety

1. **Never commit keys to git**:
   - Add `.env` to `.gitignore`
   - Use environment variables, not hardcoded values

2. **Use scoped keys when possible**:
   - Linear keys can be scoped to specific teams
   - Create dedicated keys for Claude Code

3. **Rotate keys periodically**:
   - Every 90 days for production use
   - Immediately if you suspect exposure

### Principle of Least Privilege

Only install the MCP servers you need:
- **Always needed**: Linear
- **Usually helpful**: Perplexity, Sequential Thinking
- **Only if needed**: Playwright

### Audit Usage

Periodically review:
- Linear API key usage in Linear settings
- Perplexity API costs in your account

---

## Next Steps

With MCP servers configured:

1. **Return to the main guide** — See [PM_GUIDE.md](../PM_GUIDE.md) for workflow instructions

2. **Run your first discovery** — Test that Linear integration works

3. **Review troubleshooting** — See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) if you encounter issues

---

## Quick Reference

| MCP Server | Required? | API Key Needed? | What It Does |
|------------|-----------|-----------------|--------------|
| Linear | **Yes** | Yes (`lin_api_...`) | Creates/manages tickets |
| Perplexity | Recommended | Yes (`pplx-...`) | Web search |
| Sequential Thinking | Recommended | No | Enhanced reasoning |
| Playwright | Optional | No | Browser automation |

**Install command pattern**:
```bash
claude mcp add <server-name> --scope user
```

**Check status**:
```bash
claude mcp list
```
