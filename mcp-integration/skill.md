---
name: mcp-integration
description: Connect external tools, databases, and services to Claude Code through the Model Context Protocol (MCP). MCP servers give Claude Code access to your tools and data sources. Use code execution with MCP servers instead of direct tool calls to reduce token usage and improve agent efficiency
---

#Key Concepts
What is MCP?
MCP (Model Context Protocol) is an open-source standard that allows AI tools like Claude Code to connect to external services. Think of it as a universal adapter that lets Claude Code "talk to" hundreds of different applications and data sources.
Three Types of MCP Servers

HTTP Servers (Recommended for cloud services)

Run remotely on the internet
Most common for cloud-based services
Example: Connecting to Notion, GitHub, Stripe


SSE Servers (Deprecated, use HTTP when available)

Server-Sent Events transport
Older method, being phased out


Stdio Servers (For local tools)

Run as local processes on your machine
Good for tools that need direct system access
Example: Local database connections, custom scripts



##MCP Configuration Scopes
Local Scope (default)

Private to you in the current project
Stored in project-specific user settings
Best for: Personal experiments, sensitive credentials

Project Scope

Shared with everyone via .mcp.json file
Checked into version control
Best for: Team collaboration, shared tools

User Scope

Available across all your projects
Private to your user account
Best for: Personal utilities you use everywhere

Installation Commands
Basic HTTP Server (Most Common)
bash# Basic syntax
claude mcp add --transport http <name> <url>

# Example: Connect to Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# With authentication header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
SSE Server (Older Method)
bash# Basic syntax
claude mcp add --transport sse <name> <url>

# Example: Connect to Asana
claude mcp add --transport sse asana https://mcp.asana.com/sse
Local Stdio Server
bash# Basic syntax
claude mcp add --transport stdio <name> <command> [args...]

# Example: Airtable with API key
claude mcp add --transport stdio airtable --env AIRTABLE_API_KEY=YOUR_KEY \
  -- npx -y airtable-mcp-server
Important: The -- (double dash) separates Claude's flags from the server command:

Before --: Options for Claude (like --env, --scope)
After --: The actual command to run the MCP server

Scope Options
bash# Local scope (default) - only you, only this project
claude mcp add --transport http stripe --scope local https://mcp.stripe.com

# Project scope - shared with team via .mcp.json
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# User scope - you across all projects
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
Management Commands
bash# List all configured servers
claude mcp list

# Get details for a specific server
claude mcp get github

# Remove a server
claude mcp remove github

# Check server status (within Claude Code)
/mcp

# Reset project approval choices
claude mcp reset-project-choices
Popular MCP Servers by Category
Development & Testing

Sentry: Error monitoring, debugging

bash  claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

Socket: Security analysis for dependencies
Hugging Face: Access AI models and applications
Jam: Debug with recordings, logs, network requests

Project Management

Notion: Read docs, update pages, manage tasks

bash  claude mcp add --transport http notion https://mcp.notion.com/mcp

Linear: Issue tracking, project management
Jira/Confluence: Atlassian workspace integration
Asana: Task and project management
ClickUp: Comprehensive project tracking
Monday: Board and workflow management

Databases & Data

Airtable: Base and table management
HubSpot: CRM data access
Daloopa: Financial data from SEC filings

Payments & Commerce

Stripe: Payment processing

bash  claude mcp add --transport http stripe https://mcp.stripe.com

PayPal: Commerce and transactions
Square: Payments, inventory, orders
Plaid: Banking data, account linking

Design & Media

Figma: Design file access for better code generation
Canva: Browse and generate designs
Cloudinary: Media asset management
invideo: Video creation capabilities

Infrastructure & DevOps

Cloudflare: Application deployment, traffic analysis
Netlify: Website deployment and management
Vercel: Project and deployment management
Stytch: Authentication services

Automation & Integration

Zapier: Connect to 8,000+ apps
Workato: Access applications and workflows

Authentication
Many MCP servers require OAuth 2.0 authentication:

Add the server first:

bash   claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

Use /mcp command in Claude Code to authenticate
Follow browser prompts to login

Tips:

Tokens are stored securely and auto-refresh
Use "Clear authentication" in /mcp menu to revoke access
If browser doesn't open, copy the provided URL

Advanced Features
Environment Variable Expansion
In .mcp.json files, you can use environment variables:
json{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
Syntax:

${VAR} - Use environment variable VAR
${VAR:-default} - Use VAR if set, otherwise use default value

MCP Resources
Reference resources from MCP servers using @ mentions:
@github:issue://123
@postgres:schema://users
@docs:file://api/authentication
Type @ to see available resources in autocomplete.
MCP Prompts as Slash Commands
MCP servers can expose prompts that become / commands:
/mcp__github__list_prs
/mcp__github__pr_review 456
/mcp__jira__create_issue "Bug in login flow" high
Type / to see all available commands.
Output Limits

Default warning threshold: 10,000 tokens
Default maximum: 25,000 tokens
Customize with environment variable:

bash  export MAX_MCP_OUTPUT_TOKENS=50000
  claude
Startup Timeout
Configure MCP server startup timeout:
bashexport MCP_TIMEOUT=10000  # 10 seconds
claude
Windows-Specific Notes
On Windows (not WSL), local MCP servers using npx need the cmd /c wrapper:
bashclaude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
Without this wrapper, you'll get "Connection closed" errors.
Import from Claude Desktop
If you've already configured MCP servers in Claude Desktop:
bash# Import servers (macOS and WSL only)
claude mcp add-from-claude-desktop

# With user scope
claude mcp add-from-claude-desktop --scope user
This shows an interactive dialog to select which servers to import.
Add Servers from JSON
If you have JSON configuration:
bash# HTTP server
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

# Stdio server
claude mcp add-json local-weather '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"],"env":{"CACHE_DIR":"/tmp"}}'
Use Claude Code as MCP Server
You can use Claude Code itself as an MCP server:
bash# Start Claude as stdio MCP server
claude mcp serve
Add to Claude Desktop's claude_desktop_config.json:
json{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
Enterprise Configuration
For organizations needing centralized control:
Managed MCP Configuration
Deploy managed-mcp.json at:

macOS: /Library/Application Support/ClaudeCode/managed-mcp.json
Windows: C:\ProgramData\ClaudeCode\managed-mcp.json
Linux: /etc/claude-code/managed-mcp.json

Access Control
In managed-settings.json:
json{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "sentry" }
  ],
  "deniedMcpServers": [
    { "serverName": "filesystem" }
  ]
}
Allowlist behavior:

undefined: No restrictions
Empty array []: Complete lockdown
List of names: Only specified servers allowed

Denylist behavior:

Takes absolute precedence
Blocks specified servers across all scopes

Practical Examples
Example 1: Monitor Errors with Sentry
bash# 1. Add Sentry MCP server
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Authenticate (in Claude Code)
> /mcp

# 3. Debug production issues
> "What are the most common errors in the last 24 hours?"
> "Show me the stack trace for error ID abc123"
Example 2: GitHub Code Reviews
bash# 1. Add GitHub MCP server
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. Authenticate if needed
> /mcp

# 3. Work with GitHub
> "Review PR #456 and suggest improvements"
> "Create a new issue for the bug we just found"
Example 3: Database Queries
bash# 1. Add database server with connection string
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"

# 2. Query naturally
> "What's our total revenue this month?"
> "Show me the schema for the orders table"
Example 4: Notion Integration
bash# 1. Add Notion server
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 2. Authenticate
> /mcp

# 3. Work with Notion
> "Find all project documentation pages"
> "Update the status on the Q4 planning page"
Communication Guidelines
When helping users with MCP:

Always explain what you're doing - Users are not developers, so break down each step
Use the simplest method - Prefer HTTP servers over stdio when available
Ask about scope - Help users choose between local, project, or user scope based on their needs
Verify before removing - Confirm before removing any MCP servers
Troubleshoot systematically - Check authentication, network, and configuration in order
Provide context - Explain why a particular MCP server would be useful for their use case

Common Issues and Solutions
"Connection closed" error on Windows
Solution: Add cmd /c wrapper before npx commands
Authentication fails
Solutions:

Use /mcp command to re-authenticate
Check if OAuth flow completed in browser
Clear authentication and try again

Server not appearing in /mcp list
Solutions:

Check if server was added successfully with claude mcp list
Restart Claude Code
Verify server URL is correct

MCP output too large warning
Solution: Increase limit with MAX_MCP_OUTPUT_TOKENS environment variable
Project MCP servers not working
Solution: Approve project servers when prompted, or reset with claude mcp reset-project-choices
Security Considerations

Use trusted MCP servers - Anthropic hasn't verified all third-party servers
Be careful with untrusted content - Servers fetching untrusted content pose prompt injection risk
Protect sensitive credentials - Use local scope for servers with secrets
Review project .mcp.json - Check team-shared configurations before approving
Use environment variables - Don't hardcode API keys in configuration files

Best Practices

Start with HTTP servers - They're the most widely supported and easiest to use
Use appropriate scopes - Local for experiments, project for teams, user for utilities
Document team servers - Add comments to .mcp.json explaining server purposes
Regular cleanup - Remove unused MCP servers to keep configuration clean
Test incrementally - Add one server at a time and verify it works before adding more
Use environment variables - Keep configurations flexible and secure
Monitor output size - Be aware of token limits for large data operations

Quick Reference Card
bash# Add servers
claude mcp add --transport http <name> <url>           # HTTP server
claude mcp add --transport sse <name> <url>            # SSE server
claude mcp add --transport stdio <name> -- <command>   # Local server

# Manage servers
claude mcp list                    # List all servers
claude mcp get <name>              # Get server details
claude mcp remove <name>           # Remove server
claude mcp reset-project-choices   # Reset approvals

# Import/export
claude mcp add-from-claude-desktop     # Import from Desktop
claude mcp add-json <name> '<json>'    # Add from JSON

# In Claude Code
/mcp                              # Check status, authenticate
@<server>:<resource>              # Reference resources
/<command>                        # Use MCP prompts

# Environment variables
MAX_MCP_OUTPUT_TOKENS=50000       # Increase output limit
MCP_TIMEOUT=10000                 # Set startup timeout (ms)

##Additional Resources

Find more servers: Search GitHub for MCP servers
Documentation: Visit docs.claude.com for detailed guides
Security: Always verify third-party MCP servers before use



# Call MCP with the Anthropic Python SDK

To call MCP (Model Context Protocol) with the Anthropic Python SDK, you typically use the `client.beta.messages.create()` method in the Anthropic Python SDK, passing the MCP server details via the `mcp_servers` parameter along with your prompt messages. Here's a concise example pattern from tutorials and example code:

```python
import anthropic

# Set your MCP server URL here
mcp_server_url = 'https://your-server-url.com/mcp/'

# Initialize Anthropic client (make sure ANTHROPIC_API_KEY is set in your environment)
client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-20250514",  # or other available Claude model
    max_tokens=1000,
    messages=[{"role": "user", "content": "Your query or prompt here"}],
    mcp_servers=[
        {
            "type": "url",
            "url": mcp_server_url,
            "name": "your-server-name",
        }
    ],
    extra_headers={
        "anthropic-beta": "mcp-client-2025-04-04"  # required header for MCP usage
    }
)

print(response.content)
```

Key notes:
- `mcp_servers` is a list of MCP server descriptions where each server is an object with keys like `type` (usually `"url"`), `url` (the MCP endpoint), and `name`.
- `extra_headers` must include `"anthropic-beta": "mcp-client-2025-04-04"` for MCP client identification.
- Your Anthropic API key should be configured as the `ANTHROPIC_API_KEY` environment variable.
- The MCP server endpoint URL often ends with `/mcp/` by convention but may vary depending on your deployment.

This approach lets you send messages to Claude with access to the MCP servers, enabling the model to invoke external tools and services through MCP behind the scenes.






# Appendix: Konfigurace Make.com MCP serveru pro Claude Code

🎯 Co je potřeba vědět
Make.com MCP server má dva endpointy:
/sse - Server-Sent Events (nefunguje v Claude Code)
/stream - Streamable HTTP (POUŽIJ TENTO! ✅)
Důležité: Claude Code vyžaduje /stream endpoint!
🔧 Kroky nastavení
1️⃣ Získej přístupové údaje z Make.com
V Make.com potřebuješ dva údaje: a) User Token:
Jdi do Make.com → Profile → API/MCP access
Zkopíruj si svůj user token (dlouhý řetězec znaků)
b) Organization ID:
Jdi do Make.com → Settings → Organization
Zkopíruj si Organization ID (číslo)
2️⃣ Vytvoř soubor .mcp.json v kořenu projektu
{
  "mcpServers": {
    "make": {
      "type": "http",
      "url": "${MAKE_MCP_URL}"
    }
  }
}
Co to dělá:
Definuje MCP server s názvem make
Typ je http (ne stdio, ne sse)
URL načítá z environment proměnné MAKE_MCP_URL
3️⃣ Přidej URL do .env souboru
# Make.com MCP Server Configuration
# Token je vygenerován v Make.com → Profile → API/MCP access
# organizationId zjistíš v Make.com → Settings → Organization
# Uses /stream endpoint (Streamable HTTP) instead of /sse for Claude Code compatibility
MAKE_MCP_URL=https://eu1.make.com/mcp/api/v1/u/{TVŮJ_USER_TOKEN}/stream?organizationId={TVOJE_ORG_ID}
Formát URL:
https://eu1.make.com/mcp/api/v1/u/{user_token}/stream?organizationId={org_id}
                                                ^^^^^^                   ^^^^^^
                                            DŮLEŽITÉ: /stream          Query parameter!
4️⃣ Aktivuj MCP server v projektu
Vytvoř .claude/settings.json:
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["make"]
}
Co to dělá:
enableAllProjectMcpServers: true - aktivuje MCP servery pro projekt
enabledMcpjsonServers: ["make"] - povolí konkrétně server s názvem "make"
5️⃣ Ověř, že to funguje
Spusť v terminálu:
claude mcp list
Očekávaný výstup:
make: https://eu1.make.com/mcp/api/v1/u/xxx/stream?organizationId=xxx (HTTP) - ✓ Connected
✅ Pokud vidíš ✓ Connected = vše funguje! ❌ Pokud ne, zkontroluj:
Správnost user tokenu
Správnost organizationId
Že používáš /stream a ne /sse
⚠️ Časté chyby
❌ Chyba #1: Použití /sse místo /stream
# ŠPATNĚ:
MAKE_MCP_URL=https://eu1.make.com/mcp/api/v1/u/{token}/sse

# SPRÁVNĚ:
MAKE_MCP_URL=https://eu1.make.com/mcp/api/v1/u/{token}/stream?organizationId={org_id}
❌ Chyba #2: Chybějící organizationId
# ŠPATNĚ:
MAKE_MCP_URL=https://eu1.make.com/mcp/api/v1/u/{token}/stream

# SPRÁVNĚ:
MAKE_MCP_URL=https://eu1.make.com/mcp/api/v1/u/{token}/stream?organizationId={org_id}
❌ Chyba #3: Špatný typ v .mcp.json
// ŠPATNĚ:
{
  "mcpServers": {
    "make": {
      "type": "sse",  // ❌ Toto nefunguje!
      "url": "${MAKE_MCP_URL}"
    }
  }
}

// SPRÁVNĚ:
{
  "mcpServers": {
    "make": {
      "type": "http",  // ✅ Použij http!
      "url": "${MAKE_MCP_URL}"
    }
  }
}
📁 Výsledná struktura projektu
project_root/
├── .mcp.json                    ← Definice MCP serverů
├── .env                         ← URL s tokenem a org ID
├── .claude/
│   └── settings.json           ← Aktivace MCP pro projekt
├── .gitignore                   ← Nezapomeň přidat .env!
└── ... (ostatní soubory)
⚠️ BEZPEČNOST: Nikdy necommituj .env soubor do gitu! Přidej ho do .gitignore:
.env
🎉 Shrnutí
Získej údaje z Make.com (token + org ID)
Vytvoř .mcp.json s definicí serveru (type: http)
Nastav .env s URL obsahující /stream a organizationId
Aktivuj v .claude/settings.json
Ověř příkazem claude mcp list
Klíčové:
✅ Používej /stream endpoint
✅ Přidej organizationId jako query parameter
✅ Typ serveru musí být http



# MCP Code Execution Pattern
Building more efficient agents and optimize token consumption 

Use code execution with MCP servers instead of direct tool calls to reduce token usage and improve agent efficiency


## Problem
Direct MCP tool calls consume excessive tokens:
- All tool definitions load into context upfront
- Intermediate results pass through model context
- Large datasets flow through context multiple times

## Solution
Present MCP servers as code APIs that agents can import and use in execution environment.

## Implementation Pattern

### 1. File Structure
```
servers/
├── google-drive/
│   ├── getDocument.ts
│   ├── index.ts
├── salesforce/
│   ├── updateRecord.ts
│   ├── index.ts
```

### 2. Tool File Format
```typescript
// servers/google-drive/getDocument.ts
import { callMCPTool } from "../../../client.js";

interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

/* Read a document from Google Drive */
export async function getDocument(input: GetDocumentInput): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>('google_drive__get_document', input);
}
```

### 3. Usage Example
```typescript
import * as gdrive from './servers/google-drive';
import * as salesforce from './servers/salesforce';

const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

## Key Benefits

**Progressive Disclosure**: Load only needed tools by exploring filesystem
- List `./servers/` to find available servers
- Read specific tool files only when needed
- Reduces token usage by ~98% (150K → 2K tokens)

**Context Efficiency**: Filter data before returning to model
```typescript
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(row => row["Status"] === 'pending');
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // Only return 5 rows to context
```

**Better Control Flow**: Use code patterns instead of chaining tool calls
```typescript
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({ channel: 'C123456' });
  found = messages.some(m => m.text.includes('deployment complete'));
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
```

**Privacy**: Sensitive data stays in execution environment
```typescript
// PII never enters model context
const sheet = await gdrive.getSheet({ sheetId: 'abc123' });
for (const row of sheet.rows) {
  await salesforce.updateRecord({
    objectType: 'Lead',
    recordId: row.salesforceId,
    data: { 
      Email: row.email,  // Tokenized in transit
      Phone: row.phone,
      Name: row.name
    }
  });
}
```

**State Persistence**: Save intermediate results and skills
```typescript
// Save work
await fs.writeFile('./workspace/leads.csv', csvData);

// Create reusable skills
// In ./skills/save-sheet-as-csv.ts
export async function saveSheetAsCsv(sheetId: string) {
  const data = await gdrive.getSheet({ sheetId });
  const csv = data.map(row => row.join(',')).join('\n');
  await fs.writeFile(`./workspace/sheet-${sheetId}.csv`, csv);
  return `./workspace/sheet-${sheetId}.csv`;
}
```

## Discovery Pattern

Option 1: Filesystem exploration
- List directories to find servers
- Read tool files to understand interfaces

Option 2: search_tools function
```typescript
const tools = await searchTools({
  query: "salesforce",
  detail: "name_and_description" // or "full_definition"
});
```

## When to Use
- Connected to many MCP servers (dozens+)
- Working with large datasets
- Need loops, conditionals, or complex logic
- Privacy-sensitive operations
- Building reusable agent capabilities

## Trade-offs
- Requires secure execution environment
- Adds infrastructure complexity
- Need sandboxing and resource limits
- More operational overhead than direct calls
