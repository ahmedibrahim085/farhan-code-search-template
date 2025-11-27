# FarhanAliRaza Code Search - Project Template

This template repository provides **100% local semantic code search** using [FarhanAliRaza/claude-context-local](https://github.com/FarhanAliRaza/claude-context-local) for any project.

## ✨ What You Get

- 🔍 **Semantic Code Search**: Find code by meaning, not just keywords
- 🔒 **100% Private**: No cloud APIs, your code never leaves your machine
- 💰 **Zero Cost**: No API fees, completely free
- ⚡ **Fast**: Local FAISS vector search
- 🎯 **Per-Project**: Isolated index for each project
- 🌐 **Multi-Language**: Python, JS/TS, Go, Rust, Java, C/C++, C#, Markdown

## 🚀 Quick Start (2 Minutes)

### For New Projects

```bash
# 1. Use this template for your new project
git clone <your-template-url> your-project
cd your-project

# 2. Install FarhanAliRaza globally (one-time, ~1 minute)
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash

# 3. Restart Claude Code
# Close and reopen Claude Code completely

# 4. Index your codebase
# In Claude Code, type: "Index this codebase"

# 5. Start searching!
# "Find authentication code"
# "Show me the API client class"
```

### For Existing Projects

```bash
# 1. Copy these files to your project root:
cp .mcp.json your-project/
cp .mcp-server-wrapper.sh your-project/
cp .gitignore your-project/.gitignore  # Or merge with existing

# 2. Make wrapper executable
chmod +x your-project/.mcp-server-wrapper.sh

# 3. Follow steps 2-5 from above
```

## 📋 Prerequisites

Only needs to be installed **once per machine** (not per project):

```bash
# Check if already installed:
ls ~/.local/share/claude-context-local/

# If not found, install:
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
```

**System Requirements:**
- Python 3.12+
- 1-2 GB disk space (for model + index)
- Optional: GPU for faster indexing (works on CPU)

## 📁 What's Included

```
.
├── .mcp.json                    ✅ Commit - MCP server config
├── .mcp-server-wrapper.sh       ✅ Commit - Server wrapper script
├── .gitignore                   ✅ Commit - Excludes index
├── README.md                    ✅ Commit - This file
└── .code-search-index/          ❌ DON'T commit - Local index (auto-generated)
```

## 🎮 Usage Examples

### Index Your Codebase

```
In Claude Code:
> Index this codebase

Expected output:
✅ Indexing /path/to/project...
✅ Found 121 files
✅ Created 1804 code chunks
✅ Indexed in 23 seconds
```

### Semantic Search

```
> Find all HTTP request handling code

Returns code chunks that handle HTTP requests,
even if they don't contain the exact words "HTTP request"!
```

### Class/Function Search

```
> Show me the DatabaseConnection class

Returns:
1. database/connection.py:15-45 (class DatabaseConnection) ⭐ EXACT
2. database/pool.py:20-40 (ConnectionPool)
3. database/manager.py:10-30 (DBConnectionManager)
```

### Feature Search

```
> Find error handling patterns

Returns all error handling code across your codebase,
understanding the semantic meaning of "error handling"
```

### Check Status

```
> What's the indexing status?

Returns:
{
  "total_chunks": 1804,
  "files_indexed": 121,
  "top_folders": { "src": 450, "tests": 610 }
}
```

## 🔧 How It Works

```
┌─────────────────────────────────────────────────┐
│ Your Project                                    │
│ ├── .mcp.json            ← Claude Code reads   │
│ ├── .mcp-server-wrapper.sh ← Launches server   │
│ └── .code-search-index/  ← Local FAISS index   │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ ~/.local/share/claude-context-local/            │
│ ├── mcp_server/          ← MCP server code      │
│ ├── embeddings/          ← EmbeddingGemma model │
│ └── chunking/            ← AST-based parser     │
└─────────────────────────────────────────────────┘
```

**Key Points:**
- FarhanAliRaza installed once globally in `~/.local/share/`
- Each project has its own isolated index in `.code-search-index/`
- No conflicts between projects
- Safe to delete and regenerate index anytime

## 🛠️ Troubleshooting

### "claude-context-local not found"

**Fix:**
```bash
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
```

### "uv command not found"

**Fix:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Restart terminal
```

### MCP Server Not Loading

**Checklist:**
1. Is `.mcp.json` in project root? ✓
2. Did you restart Claude Code? ✓
3. Is wrapper executable? `ls -la .mcp-server-wrapper.sh`
4. Does global install exist? `ls ~/.local/share/claude-context-local/`

### Re-index from Scratch

```bash
# Delete index:
rm -rf .code-search-index/

# Re-index in Claude Code:
"Index this codebase"
```

## 📊 Performance

| Project Size | Files | Indexing Time | Storage |
|--------------|-------|---------------|---------|
| Small | <100 | 10-20 sec | ~5 MB |
| Medium | 1,000 | 1-2 min | ~50 MB |
| Large | 10,000 | 10-15 min | ~500 MB |

**Search Speed:** 50-200ms per query

## 🌟 Why FarhanAliRaza?

We tested **3 local code search implementations**. Only FarhanAliRaza works reliably:

| Implementation | Status | Issues |
|----------------|--------|--------|
| danielbowne/claude-context-mcp | ❌ Broken | State tracking bug (GitHub #226) |
| MikeO-AI/claude-context-local-mcp | ❌ Broken | 100-chunk hard limit |
| **FarhanAliRaza/claude-context-local** | ✅ **Working** | None found |

**Tested with:** 121 files, 1804 chunks, production-ready

## 📚 Architecture

- **Embeddings:** Google EmbeddingGemma (768 dimensions)
- **Vector DB:** FAISS (IndexFlatIP)
- **Chunking:** AST-based (respects code structure)
- **Languages:** Python, JS/TS, Go, Rust, Java, C/C++, C#, Markdown
- **Storage:** Project-local `.code-search-index/`

## 🤝 Contributing

Found this template useful? Consider:
- ⭐ Starring the original [FarhanAliRaza/claude-context-local](https://github.com/FarhanAliRaza/claude-context-local)
- 📝 Improving this template via pull requests
- 🐛 Reporting issues you encounter

## 📄 License

This template configuration is provided as-is.

FarhanAliRaza/claude-context-local has its own license - see their repository.

## 🔗 Resources

- [FarhanAliRaza Repository](https://github.com/FarhanAliRaza/claude-context-local)
- [Setup Guide](SETUP_GUIDE.md) - Detailed instructions
- [Comparison](THREE_WAY_COMPARISON.md) - Why we chose FarhanAliRaza

---

**Last Updated:** 2025-11-27
**Status:** ✅ Production-Ready
**Tested On:** lmstudio-bridge-enhanced (1804 chunks, 121 files)
