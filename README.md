# 🔍 DevCheck (Entire Graph Buildathon)

**Track 2: Build with Graph Intelligence**

DevCheck is a local **Model Context Protocol (MCP)** server powered by Entire Graph. It equips AI coding assistants (like Antigravity) with the ability to safely and intelligently predict the "blast radius" of code changes before they are made.

By turning raw structural graph evidence into an actionable, hedged, and verified workflow, DevCheck bridges the gap between static analysis and agentic coding.

---

## 🚀 The Problem

When developers or AI agents modify complex codebases, they risk breaking unseen dependencies (callers, callees, type consumers, or data flows). Standard text-based search (grep) is often too noisy or misses structural relationships.

While **Entire Graph** provides powerful structural data, raw graph output is dense. It requires interpretation, uncertainty handling (for things like dynamic dispatch), and safe fallback mechanisms.

## 💡 The Solution

DevCheck wraps the Entire Graph CLI into a standardized MCP server. When an AI agent needs to understand the impact of modifying a function, it silently calls DevCheck. 

DevCheck executes the Graph queries, interprets the results, classifies the evidence (Confirmed vs. Heuristic), handles unresolved patterns, and returns a beautifully formatted Markdown report with actionable fallbacks directly into the chat.

---

## ⚙️ How It Works

### 1. High-Level Architecture
DevCheck acts as a bridge between the AI Assistant and the local codebase's Entire Graph index.

```mermaid
sequenceDiagram
    participant U as Developer
    participant A as AI Agent (Antigravity)
    participant M as DevCheck (MCP Server)
    participant G as Entire Graph CLI
    participant C as Local Codebase

    U->>A: "What happens if I change formatImpactReport?"
    A->>M: Call tool: check_change_impact(target)
    M->>G: entire graph search
    M->>G: entire graph impact
    G->>C: Query local graph index
    C-->>G: Raw impact data (Callers, data flows)
    G-->>M: Text output
    M->>M: Parse, classify evidence, generate fallbacks
    M-->>A: Formatted Markdown Report
    A-->>U: Displays actionable impact analysis
```

### 2. Evidence Processing Pipeline
The core value of DevCheck is its parsing and classification engine (`report.ts`). It doesn't just pass raw data; it structures and scrutinizes it.

```mermaid
flowchart TD
    A[Raw Entire Graph Output] --> B{Parse Sections}
    B -->|Callers, Callees, Types| C[Evaluate Evidence]
    B -->|Co-change, Siblings| D[Mark as HEURISTIC 🟡]
    
    C --> E{Contains Unresolved Patterns?}
    E -->|Yes (e.g. dynamic dispatch)| F[Mark as NEEDS VERIFICATION 🔴]
    E -->|No| G[Mark as CONFIRMED 🟢]
    
    D --> H[Assemble Affected Items]
    F --> H
    G --> H
    
    H --> I{Is Analysis Partial?}
    I -->|Yes| J[Add Strict Fallbacks<br/>1. Full-text search<br/>2. E2E Test Suite]
    I -->|No| K[Add Standard Checks<br/>1. Unit Tests<br/>2. Manual search]
    
    J --> L[Generate Final Markdown Report]
    K --> L
```

---

## ✨ Key Features

1. **Uncertainty Handling**: DevCheck never presents incomplete relationships as absolute truth. If Entire Graph flags an "unresolved caller" (e.g., due to reflection or dynamic dispatch), DevCheck explicitly triggers a `[!WARNING]` block.
2. **Evidence Distinction**: Every affected item is badged:
   * 🟢 `[CONFIRMED]` - Strict structural dependencies (Callers, Type Consumers).
   * 🟡 `[HEURISTIC]` - Statistical relationships (Co-change files, Siblings).
   * 🔴 `[NEEDS VERIFICATION]` - Unresolved patterns requiring human eyes.
3. **Smart Fallbacks**: If the analysis is partial, DevCheck alters its "Recommended Checks" to enforce safe fallbacks (e.g., demanding a repository-wide grep and E2E test runs).
4. **Agent-Optimized Formatting**: The output uses hedging language ("Changing X *may* affect Y") and always concludes with a mandatory verification disclaimer to prevent AI hallucinations.

---

## 🛠️ Usage & Integration

### Requirements
* Node.js (v18+)
* `entire` CLI installed and authenticated
* `graph` plugin installed (`entire plugin install graph`)

### Setup
1. Clone this repository.
2. Navigate to `devcheck/` and run `npm install`.
3. Build the project: `npm run build`.
4. Add the MCP server to your AI IDE (e.g., Antigravity or Cursor) by pointing the configuration to `node /path/to/devcheck/dist/server.js`.

### Available MCP Tools
* `scan_project`: Diagnoses the codebase to ensure Entire Graph is initialized and ready.
* `check_change_impact`: The primary tool. Accepts a `target` symbol and returns the comprehensive blast radius report.
