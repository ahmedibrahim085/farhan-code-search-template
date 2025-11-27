# FarhanAliRaza Code Search MCP - Per-Project Setup Guide

**Last Updated:** 2025-11-27
**Status:** ✅ Production-Ready (Tested on lmstudio-bridge-enhanced)
**Architecture:** 100% Local, No API Keys, Per-Project Indexing

---

## 🎯 What This Setup Provides

- **100% Local**: No cloud APIs, no external dependencies (except initial install)
- **Per-Project Indexing**: Each project has its own isolated search index
- **Google EmbeddingGemma**: State-of-the-art local embeddings
- **FAISS Vector Search**: Fast, proven vector similarity search
- **AST-Based Chunking**: Respects code structure (classes, functions, methods)
- **Multi-Language Support**: Python, JS/TS, Go, Rust, Java, C/C++, C#, Markdown
- **Path Agnostic**: Works in any project directory
- **Git-Aware**: Automatically finds project root

---

## 📋 Prerequisites

### 1. Check if FarhanAliRaza is Already Installed

```bash
ls -la ~/.local/share/claude-context-local/
```

**If exists:** ✅ Skip to "Project Configuration" section
**If not found:** Continue with installation below

### 2. System Requirements

- **Python:** 3.12+ (check: `python3 --version`)
- **uv:** Package manager (auto-installs if missing)
- **Disk:** 1-2 GB for model + index
- **Optional:** GPU for faster embedding (works on CPU too)

---

## 🚀 Installation (One-Time, Global)

**IMPORTANT:** This installs FarhanAliRaza globally in your home directory. You only do this ONCE, then configure per-project.

### Quick Install

```bash
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
```

### What Gets Installed

```
~/.local/share/claude-context-local/   # FarhanAliRaza installation
├── chunking/                           # AST-based code parsing
├── embeddings/                         # EmbeddingGemma model
├── mcp_server/                         # MCP server implementation
├── scripts/                            # Indexing utilities
└── .venv/                              # Python dependencies
```

### Verify Installation

```bash
ls ~/.local/share/claude-context-local/mcp_server/
# Should show: server.py
```

---

## 🔧 Project Configuration

Run these commands **inside your project directory**:

### Step 1: Create .mcp.json

```bash
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "code-search": {
      "type": "stdio",
      "command": "bash",
      "args": [
        "-c",
        "exec \"$(git rev-parse --show-toplevel 2>/dev/null || echo .)/.mcp-server-wrapper.sh\""
      ]
    }
  }
}
EOF
```

### Step 2: Create Wrapper Script

```bash
cat > .mcp-server-wrapper.sh << 'EOF'
#!/usr/bin/env bash
set -e

# Resolve user's home directory
if [ -z "$HOME" ]; then
    echo "ERROR: HOME environment variable not set" >&2
    exit 1
fi

# Standard installation location
INSTALL_DIR="$HOME/.local/share/claude-context-local"

# Check if installation exists
if [ ! -d "$INSTALL_DIR" ]; then
    echo "ERROR: claude-context-local not found at $INSTALL_DIR" >&2
    echo "" >&2
    echo "Please install with:" >&2
    echo "  curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash" >&2
    exit 1
fi

# Find uv executable (check common locations)
UV_CMD=""
if command -v uv &> /dev/null; then
    UV_CMD="uv"
elif [ -f "$HOME/.langflow/uv/uv" ]; then
    UV_CMD="$HOME/.langflow/uv/uv"
elif [ -f "$HOME/.cargo/bin/uv" ]; then
    UV_CMD="$HOME/.cargo/bin/uv"
elif [ -f "$HOME/.local/bin/uv" ]; then
    UV_CMD="$HOME/.local/bin/uv"
else
    echo "ERROR: uv command not found" >&2
    echo "" >&2
    echo "Install uv with:" >&2
    echo "  curl -LsSf https://astral.sh/uv/install.sh | sh" >&2
    exit 1
fi

# Set storage directory to project-local path
PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export CODE_SEARCH_STORAGE="$PROJECT_ROOT/.code-search-index"

# Execute the MCP server using uv
exec "$UV_CMD" run --directory "$INSTALL_DIR" python mcp_server/server.py "$@"
EOF

chmod +x .mcp-server-wrapper.sh
```

### Step 3: Update .gitignore

```bash
# Add to .gitignore (creates if doesn't exist)
cat >> .gitignore << 'EOF'

# Code Search Index (local, per-machine)
.code-search-index/

# MCP Server Wrapper (project-specific, should be committed)
# .mcp-server-wrapper.sh  # DON'T ignore this!

EOF
```

**IMPORTANT:**
- ✅ **DO commit:** `.mcp.json` and `.mcp-server-wrapper.sh` (everyone needs these)
- ❌ **DON'T commit:** `.code-search-index/` (local index, regenerate per machine)

### Step 4: Verify Configuration

```bash
# Should show your new files:
ls -la | grep -E "(\.mcp\.json|\.mcp-server-wrapper\.sh|\.code-search-index)"

# Should show:
# -rwxr-xr-x  .mcp-server-wrapper.sh
# -rw-r--r--  .mcp.json
# drwxr-xr-x  .code-search-index/  (after first indexing)
```

---

## 📦 Directory Structure After Setup

```
your-project/
├── .mcp.json                       ✅ Commit this
├── .mcp-server-wrapper.sh          ✅ Commit this
├── .gitignore                      ✅ Updated (excludes index)
├── .code-search-index/             ❌ DON'T commit (in .gitignore)
│   └── projects/
│       └── your-project_abc123/    # Unique per project
│           ├── index/
│           │   ├── chunk_ids.pkl
│           │   ├── embeddings.faiss
│           │   └── stats.json
│           └── project_info.json
├── src/                            # Your source code
├── tests/                          # Your tests
└── ...
```

---

## 🎮 Usage

### 1. Restart Claude Code

**CRITICAL:** Claude Code must be restarted to load the new `.mcp.json` configuration.

```bash
# Close Claude Code completely, then reopen
# Or use Command Palette > "Reload Window"
```

### 2. Index Your Codebase

In Claude Code, type:

```
Index this codebase
```

Expected output:
```
✅ Indexing /path/to/your-project...
✅ Found 121 files
✅ Created 1804 code chunks
✅ Indexed in 23 seconds
```

### 3. Search Your Code

```
# Semantic search (by meaning):
Find all HTTP request handling code

# Class/function search:
Show me the LLMClient class

# Feature search:
Find error handling patterns
```

### 4. Check Index Status

```
What's the indexing status?
```

Expected response:
```json
{
  "total_chunks": 1804,
  "files_indexed": 121,
  "embedding_dimension": 768,
  "index_type": "IndexFlatIP",
  "top_folders": {
    "src": 450,
    "tests": 610,
    "utils": 111
  }
}
```

---

## 🔍 Verify It's Working

### Test 1: Check MCP Server Loads

In Claude Code, after restart, you should see:
- No errors in output
- MCP server "code-search" listed in status

### Test 2: Index a Small Project

```bash
# Create test project:
mkdir /tmp/test-project && cd /tmp/test-project
echo "def hello(): print('world')" > test.py

# Setup (copy .mcp.json and wrapper from above)
# Restart Claude Code
# Index and search
```

### Test 3: Verify Index Files Created

```bash
ls .code-search-index/projects/*/index/
# Should show:
# chunk_ids.pkl
# embeddings.faiss
# stats.json
```

---

## 🛠️ Troubleshooting

### Error: "claude-context-local not found"

**Cause:** FarhanAliRaza not installed globally

**Fix:**
```bash
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
```

### Error: "uv command not found"

**Cause:** uv package manager not installed

**Fix:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Then restart your terminal
```

### Error: "Permission denied: .mcp-server-wrapper.sh"

**Cause:** Wrapper script not executable

**Fix:**
```bash
chmod +x .mcp-server-wrapper.sh
```

### Error: MCP server not loading after restart

**Checklist:**
1. ✅ Is `.mcp.json` in project root?
2. ✅ Did you restart Claude Code completely?
3. ✅ Is wrapper script executable? (`ls -la .mcp-server-wrapper.sh`)
4. ✅ Does `~/.local/share/claude-context-local/` exist?

**Debug:**
```bash
# Test wrapper directly:
bash .mcp-server-wrapper.sh --version

# Should execute without errors
```

### Error: Indexing is slow

**Expected Performance:**
- Small project (<100 files): 10-20 seconds
- Medium project (1000 files): 1-2 minutes
- Large project (10,000 files): 10-15 minutes

**If slower:**
- First run downloads EmbeddingGemma model (~1GB)
- Subsequent runs use cached model
- GPU acceleration helps (but not required)

### Error: Search returns no results

**Checklist:**
1. ✅ Did indexing complete successfully?
2. ✅ Check index stats: "What's the indexing status?"
3. ✅ Verify files in `.code-search-index/projects/*/index/`

**Fix:**
```bash
# Re-index from scratch:
rm -rf .code-search-index/
# Then in Claude Code: "Index this codebase"
```

---

## 🔄 Multiple Projects

You can set up multiple projects with **independent indexes**:

```bash
# Project A
cd ~/projects/project-a
# Setup .mcp.json + wrapper
# Index creates: .code-search-index/projects/project-a_abc123/

# Project B
cd ~/projects/project-b
# Setup .mcp.json + wrapper (same files!)
# Index creates: .code-search-index/projects/project-b_def456/

# Each project has isolated index!
```

**Key Points:**
- Same `.mcp.json` and wrapper script content for all projects
- Each project gets unique index in `.code-search-index/`
- No conflicts, no shared state
- Switch between projects by opening different Claude Code windows

---

## 📊 Performance & Storage

### Indexing Speed
- **~150 chunks/second** on average hardware
- **Parallelized** across CPU cores
- **First run slower** (downloads model)

### Storage Usage
- **EmbeddingGemma model:** ~1.2 GB (shared across projects)
- **Index per project:** ~50 KB per file
  - 100 files ≈ 5 MB
  - 1000 files ≈ 50 MB
  - 10,000 files ≈ 500 MB

### Search Speed
- **Query latency:** 50-200ms
- **Results:** Top 5-10 most relevant chunks
- **Ranking:** FAISS similarity + metadata filters

---

## 🌟 What Makes This Better

### vs Claude's Default Context
- ✅ **Semantic search** (not just keyword)
- ✅ **No token costs** (searches locally)
- ✅ **Finds similar code** (not just exact matches)
- ✅ **Understands intent** ("find auth code" works!)

### vs Other Local Code Search
- ✅ **Actually works** (tested, not broken like danielbowne #226)
- ✅ **No 100-chunk limit** (unlike MikeO-AI)
- ✅ **No external APIs** (unlike Zilliztech/Milvus)
- ✅ **Per-project isolation** (not global index)

### vs Cloud-Based Search
- ✅ **100% private** (code never leaves machine)
- ✅ **Zero costs** (no API fees)
- ✅ **Offline capable** (after first setup)
- ✅ **Instant results** (no network latency)

---

## 📝 Quick Setup Script

Save this as `setup-farhan-code-search.sh`:

```bash
#!/usr/bin/env bash
set -e

echo "🔍 Setting up FarhanAliRaza Code Search for this project..."

# 1. Check if already installed globally
if [ ! -d "$HOME/.local/share/claude-context-local" ]; then
    echo "❌ FarhanAliRaza not installed globally"
    echo "📥 Installing now..."
    curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
else
    echo "✅ FarhanAliRaza already installed globally"
fi

# 2. Create .mcp.json
echo "📝 Creating .mcp.json..."
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "code-search": {
      "type": "stdio",
      "command": "bash",
      "args": [
        "-c",
        "exec \"$(git rev-parse --show-toplevel 2>/dev/null || echo .)/.mcp-server-wrapper.sh\""
      ]
    }
  }
}
EOF

# 3. Create wrapper script
echo "📝 Creating .mcp-server-wrapper.sh..."
cat > .mcp-server-wrapper.sh << 'EOF'
#!/usr/bin/env bash
set -e

# Resolve user's home directory
if [ -z "$HOME" ]; then
    echo "ERROR: HOME environment variable not set" >&2
    exit 1
fi

# Standard installation location
INSTALL_DIR="$HOME/.local/share/claude-context-local"

# Check if installation exists
if [ ! -d "$INSTALL_DIR" ]; then
    echo "ERROR: claude-context-local not found at $INSTALL_DIR" >&2
    echo "" >&2
    echo "Please install with:" >&2
    echo "  curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash" >&2
    exit 1
fi

# Find uv executable (check common locations)
UV_CMD=""
if command -v uv &> /dev/null; then
    UV_CMD="uv"
elif [ -f "$HOME/.langflow/uv/uv" ]; then
    UV_CMD="$HOME/.langflow/uv/uv"
elif [ -f "$HOME/.cargo/bin/uv" ]; then
    UV_CMD="$HOME/.cargo/bin/uv"
elif [ -f "$HOME/.local/bin/uv" ]; then
    UV_CMD="$HOME/.local/bin/uv"
else
    echo "ERROR: uv command not found" >&2
    echo "" >&2
    echo "Install uv with:" >&2
    echo "  curl -LsSf https://astral.sh/uv/install.sh | sh" >&2
    exit 1
fi

# Set storage directory to project-local path
PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export CODE_SEARCH_STORAGE="$PROJECT_ROOT/.code-search-index"

# Execute the MCP server using uv
exec "$UV_CMD" run --directory "$INSTALL_DIR" python mcp_server/server.py "$@"
EOF

chmod +x .mcp-server-wrapper.sh

# 4. Update .gitignore
echo "📝 Updating .gitignore..."
if ! grep -q ".code-search-index/" .gitignore 2>/dev/null; then
    echo "" >> .gitignore
    echo "# Code Search Index (local, per-machine)" >> .gitignore
    echo ".code-search-index/" >> .gitignore
fi

# 5. Done!
echo ""
echo "✅ Setup complete!"
echo ""
echo "📋 Next Steps:"
echo "1. Restart Claude Code (completely close and reopen)"
echo "2. In Claude Code, type: 'Index this codebase'"
echo "3. Wait for indexing to complete (~20 seconds for small projects)"
echo "4. Start searching: 'Find authentication code'"
echo ""
echo "📁 Files created:"
echo "  ✅ .mcp.json"
echo "  ✅ .mcp-server-wrapper.sh"
echo "  ✅ .gitignore (updated)"
echo ""
echo "💡 Tip: Commit .mcp.json and .mcp-server-wrapper.sh to git"
echo "        DON'T commit .code-search-index/ (already in .gitignore)"
```

**Usage:**
```bash
cd your-project
curl -fsSL https://url-to-this-script.sh | bash
# Or: bash setup-farhan-code-search.sh
```

---

## ✅ Checklist

Before first use:

- [ ] FarhanAliRaza installed in `~/.local/share/claude-context-local/`
- [ ] `.mcp.json` created in project root
- [ ] `.mcp-server-wrapper.sh` created and executable
- [ ] `.gitignore` excludes `.code-search-index/`
- [ ] Claude Code restarted completely
- [ ] Tested indexing: "Index this codebase"
- [ ] Tested search: "Find [something]"
- [ ] Verified index files in `.code-search-index/projects/`

---

## 🎓 Summary

You've configured **FarhanAliRaza/claude-context-local** with:

| Component | Technology | Location |
|-----------|------------|----------|
| **MCP Server** | FarhanAliRaza Python | `~/.local/share/claude-context-local/` |
| **Embeddings** | Google EmbeddingGemma | Cached locally (~1.2 GB) |
| **Vector DB** | FAISS | Project-local index |
| **Storage** | Per-project | `.code-search-index/` |
| **Privacy** | 100% Local | No external APIs |
| **Cost** | Free | Forever |

**Winner:** Same powerful search as cloud services, completely local! 🏆

---

*Generated: 2025-11-27*
*Architecture: FarhanAliRaza/claude-context-local*
*Tested on: lmstudio-bridge-enhanced (1804 chunks, 121 files)*
