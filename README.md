# FarhanAliRaza Code Search - Project Template

This template repository provides **100% local semantic code search** using [FarhanAliRaza/claude-context-local](https://github.com/FarhanAliRaza/claude-context-local) for any project.

## ✨ What You Get

- 🔍 **Semantic Code Search**: Find code by meaning, not just keywords
- 🔒 **100% Private**: No cloud APIs, your code never leaves your machine
- 💰 **Zero Cost**: No API fees, completely free
- ⚡ **Fast**: Local FAISS vector search
- 🎯 **Per-Project**: Isolated index for each project
- 🌐 **Multi-Language**: Python, JS/TS, Go, Rust, Java, C/C++, C#, Markdown

## 🚀 Quick Start

### For New Projects

1. **Use this template for your new project**
   ```bash
   git clone <your-template-url> your-project
   cd your-project
   ```

2. **Install FarhanAliRaza globally** (one-time, ~1 minute)
   ```bash
   curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
   ```

3. **Enable Claude Code MCP approval** (CRITICAL - Required for MCP to load)

   Create or edit `.claude/settings.local.json` in your project:
   ```bash
   mkdir -p .claude
   cat > .claude/settings.local.json << 'EOF'
   {
     "enableAllProjectMcpServers": true
   }
   EOF
   ```

   **Why needed**: Claude Code requires explicit approval to load MCP servers from `.mcp.json`. Without this setting, the server won't activate even after restart.

   **Alternative** (selective approval):
   ```json
   {
     "enabledMcpjsonServers": ["code-search"]
   }
   ```

4. **Restart Claude Code**

   Close and reopen Claude Code completely

5. **Verify MCP server loaded**

   Ask Claude: "List available MCP tools"

   You should see `mcp__code-search__*` tools listed

6. **Index your codebase**

   In Claude Code: "Index this codebase"

7. **Start searching!**

   Examples:
   - "Find authentication code"
   - "Show me the API client class"

### For Existing Projects

1. **Copy configuration files** to your project root:
   ```bash
   cp .mcp.json your-project/
   cp .mcp-server-wrapper.sh your-project/
   # Merge with existing .gitignore if it exists
   cat .gitignore >> your-project/.gitignore
   ```

2. **Make wrapper executable**
   ```bash
   chmod +x your-project/.mcp-server-wrapper.sh
   ```

3. **Enable Claude Code MCP approval** (CRITICAL)

   Create or edit `.claude/settings.local.json` in your project:
   ```bash
   cd your-project
   mkdir -p .claude
   cat > .claude/settings.local.json << 'EOF'
   {
     "enableAllProjectMcpServers": true
   }
   EOF
   ```

4. **Install FarhanAliRaza globally** (if not already installed)
   ```bash
   curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
   ```

5. **Restart Claude Code**

   Close and reopen completely

6. **Verify and index**

   Ask Claude: "List available MCP tools" (should see `mcp__code-search__*`)

   Then: "Index this codebase"

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
├── .mcp.json                         ✅ Commit - MCP server config
├── .mcp-server-wrapper.sh            ✅ Commit - Server wrapper script
├── .gitignore                        ✅ Commit - Excludes index
├── .claude/
│   └── settings.local.json.template  ✅ Commit - Template for MCP approval
├── README.md                         ✅ Commit - This file
├── SETUP_CHECKLIST.md                ✅ Commit - Verification guide
└── .code-search-index/               ❌ DON'T commit - Local index (auto-generated)
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
│ ├── .claude/                                    │
│ │   └── settings.local.json ← Enables MCP      │
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
- Claude Code approval via `.claude/settings.local.json` required
- No conflicts between projects
- Safe to delete and regenerate index anytime

## 🛠️ Troubleshooting

### MCP Server Not Loading After Restart

**Symptom**: No `mcp__code-search__*` tools available after restart

**Cause**: Missing Claude Code approval setting

**Fix**:
1. Create or edit `.claude/settings.local.json` in your project root:
   ```bash
   mkdir -p .claude
   cat > .claude/settings.local.json << 'EOF'
   {
     "enableAllProjectMcpServers": true
   }
   EOF
   ```

2. Restart Claude Code again

**Verify fix**:
```bash
# Check if approval setting exists
cat .claude/settings.local.json | grep enableAllProjectMcpServers
# Should output: "enableAllProjectMcpServers": true
```

**Additional checklist**:
1. ✅ Is `.mcp.json` in project root? `ls .mcp.json`
2. ✅ Is wrapper executable? `ls -la .mcp-server-wrapper.sh` (should show `rwx`)
3. ✅ Is global install present? `ls ~/.local/share/claude-context-local/`
4. ✅ Is approval enabled? `grep enableAllProjectMcpServers .claude/settings.local.json`

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

### Wrapper Script Permission Denied

**Fix:**
```bash
chmod +x .mcp-server-wrapper.sh
```

### Re-index from Scratch

```bash
# Delete index:
rm -rf .code-search-index/

# Re-index in Claude Code:
"Index this codebase"
```

### Index Not Building / Search Returns No Results

**Checklist**:
1. ✅ Did indexing complete successfully?
2. ✅ Check index stats: "What's the indexing status?"
3. ✅ Verify files in `.code-search-index/projects/*/index/`

**Fix**:
```bash
# Re-index from scratch
rm -rf .code-search-index/
# Then in Claude Code: "Index this codebase"
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
- [Setup Checklist](SETUP_CHECKLIST.md) - Step-by-step verification
- [Comparison](THREE_WAY_COMPARISON.md) - Why we chose FarhanAliRaza

---

**Last Updated:** 2025-11-27
**Status:** ✅ Production-Ready
**Tested On:** lmstudio-bridge-enhanced (1804 chunks, 121 files)
