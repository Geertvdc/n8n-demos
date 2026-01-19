# AGENTS.md - n8n Workflow Automation Project

This document provides guidelines for AI coding agents working on n8n workflow automation projects.

## Project Overview

This is an **n8n workflow automation project** focused on multi-agent customer support workflows. The primary codebase consists of JSON workflow configurations with embedded JavaScript code nodes, not traditional application source code.

**Technology Stack:**
- n8n (workflow automation platform)
- Docker (containerized deployment)
- JavaScript (embedded in Code nodes)
- JSON (workflow configuration)
- OpenCode AI (development tooling)
- MCP (Model Context Protocol)

## Build/Deploy/Test Commands

### Container Management
```bash
# Pull n8n image
docker pull docker.n8n.io/n8nio/n8n:latest

# Create persistent volume
docker volume create n8n_demos_data

# Run n8n container
docker run -it --rm \
  --name n8n-demos \
  -p 5678:5678 \
  -e GENERIC_TIMEZONE="CET" \
  -e TZ="CET" \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
  -e N8N_RUNNERS_ENABLED=true \
  -v n8n_demos_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n:latest
```

### MCP Server (for AI integration)
```bash
# Pull n8n MCP server
docker pull ghcr.io/czlonkowski/n8n-mcp:latest

# MCP server runs automatically via OpenCode configuration
# Configuration: .opencode/opencode.jsonc
```

### Development Tools
```bash
# Install OpenCode AI dependencies (from .opencode/ directory)
cd .opencode && bun install

# No traditional build/test/lint commands - this is a workflow configuration project
```

### Testing Workflows
- **Manual Testing**: Import workflow JSON into n8n UI (http://localhost:5678)
- **Workflow Validation**: Use n8n's built-in workflow execution testing
- **Node Testing**: Test individual nodes using n8n's node execution feature
- **Integration Testing**: Test with real WhatsApp/Email triggers in sandbox mode

## Code Style Guidelines

### JavaScript Code in n8n Code Nodes

#### Naming Conventions
```javascript
// Variables: camelCase
const messageText = input.messageText;
const detectedLang = input.detectedSourceLanguage;

// Functions: camelCase with descriptive names
function generateSummary(text) { /* ... */ }
function getPriority(text) { /* ... */ }

// Constants: UPPER_SNAKE_CASE for arrays/configs
const URGENT_KEYWORDS = ['urgent', 'emergency', 'asap'];
const HIGH_PRIORITY_WORDS = ['problem', 'issue', 'error'];
```

#### Error Handling
```javascript
// Always validate inputs explicitly
if (!messageText || messageText.trim().length === 0) {
  throw new Error('No message content found in input');
}

// Use optional chaining for nested properties
const messageBody = input.messages?.[0]?.text?.body;

// Provide fallbacks for undefined values
const sourceType = input.sourceType || 'unknown';
const detectedLang = input.detectedSourceLanguage || 'en';
```

#### Data Processing Patterns
```javascript
// Defensive programming with size limits
if (messageText.length > 5000) {
  messageText = messageText.substring(0, 5000) + '... [truncated]';
}

// Cross-node data access pattern
const sourceData = $('Data Normalizer & Validator').first().json;
const translatedText = $('Smart Language Translator').first().json.translatedText;

// Return structured objects
return {
  originalMessage: messageText,
  sourceType: sourceType,
  sourceId: sourceId,
  timestamp: new Date().toISOString()
};
```

### n8n Workflow Configuration

#### Node Naming Conventions
- **Descriptive & Action-Oriented**: "Data Normalizer & Validator"
- **Service + Action Format**: "Smart Language Translator", "Admin Email Notification"  
- **Source Identification**: "WhatsApp Trigger", "Email Trigger (IMAP)"
- **Purpose Clarity**: "Enhanced Summary & Priority Processor"

#### Parameter Structure
```json
{
  "parameters": {
    "operation": "specific_action",
    "options": {},
    "additionalFields": {},
    // Service-specific configuration follows
  }
}
```

#### Expression Patterns
```javascript
// Dynamic value references
"subject": "={{ $json.emailSubject }}"
"text": "={{ $('NodeName').first().json.propertyName }}"

// Conditional expressions  
"conditions": [{
  "leftValue": "={{ $json.sourceType }}",
  "rightValue": "whatsapp",
  "operator": {"type": "string", "operation": "equals"}
}]
```

### Import Standards
```javascript
// n8n Code nodes - no imports needed, use built-in functions
// Access node data: $('NodeName').first().json
// Access current item: $json
// Access all items: $input.all()
```

### Type Handling
```javascript
// Always validate types before processing
if (typeof messageText !== 'string') {
  throw new Error('Expected string input for message text');
}

// Use explicit type conversion when needed
const wordCount = parseInt(input.wordCount) || 0;
const priority = String(input.priority).toLowerCase();
```

## Architecture Patterns

### Workflow Design
- **Pipeline Pattern**: Raw Input → Normalization → Translation → Processing → Multiple Outputs
- **Fan-out Pattern**: Single processor node outputting to multiple parallel branches
- **Error Isolation**: Each Code node handles its own validation and error cases

### Data Flow
- **Structured Data Objects**: Consistent property naming across nodes
- **Reference Pattern**: Use `$('NodeName').first().json.property` for cross-node access
- **State Management**: Pass processing results through structured return objects

### Integration Patterns
```json
// Credential structure
"credentials": {
  "serviceType": {
    "id": "unique_identifier", 
    "name": "service-environment-suffix"
  }
}

// Multi-service authentication
"authentication": "serviceAccount"  // For Google services
"authentication": "webhook"         // For WhatsApp/external APIs
```

## Best Practices

### Performance
- Use early returns in functions for efficiency
- Implement data size limits (5000 chars for message text)
- Structure algorithms with reusable functions
- Collect metrics for monitoring (processing time, word counts)

### Security
- Never hardcode credentials in workflow JSON
- Use n8n credential management system
- Validate all external inputs
- Sanitize data before processing

### Maintainability
- Use descriptive node names that indicate purpose
- Structure Code nodes with clear separation of concerns
- Document complex algorithms with comments
- Implement comprehensive error messages

### Monitoring
- Include timestamp generation in processing nodes
- Log structured data for debugging
- Implement reference ID generation for tracking
- Use console.log sparingly (only for debugging)

## File Structure

```
/
├── .opencode/                 # OpenCode AI configuration
│   ├── opencode.jsonc        # MCP server configuration
│   └── package.json          # Development dependencies
├── email-whatsapp-workflow.json  # Main n8n workflow (435 lines)
├── .env                      # Environment variables (N8N_API_KEY)
├── README.md                 # Project documentation
└── AGENTS.md                 # This file
```

## Environment Setup

1. **Required Environment Variables:**
   ```bash
   N8N_API_KEY=your_jwt_token_here
   ```

2. **Container Requirements:**
   - Docker installed and running
   - Port 5678 available for n8n UI
   - Network access for MCP server communication

3. **Development Tools:**
   - OpenCode AI plugin installed
   - n8n MCP server configured
   - Bun package manager (for OpenCode dependencies)

## Workflow Development Guidelines

- **Test Incrementally**: Test each node individually before connecting the full workflow
- **Use Descriptive Names**: Every node should clearly indicate its purpose
- **Handle Edge Cases**: Account for missing data, API failures, and malformed inputs  
- **Implement Logging**: Include structured logging for debugging and monitoring
- **Follow Data Flow**: Maintain clear data transformation pipeline from input to output
- **Validate Configurations**: Use n8n's workflow validation before deployment

This project demonstrates enterprise-level workflow automation with robust error handling, comprehensive logging, and sophisticated text processing algorithms.