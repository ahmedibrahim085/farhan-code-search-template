# Setup Verification Checklist

Use this checklist to verify your FarhanAliRaza Code Search setup is complete and working correctly.

## 📋 Pre-Installation Check

- [ ] Python 3.12+ installed (`python3 --version`)
- [ ] Git installed and working (`git --version`)
- [ ] At least 2 GB free disk space (`df -h ~`)
- [ ] Claude Code installed and running

## 🔧 File Structure Verification

Run these commands in your project root:

### 1. Check Configuration Files Exist

```bash
# All three should exist:
ls -la .mcp.json .mcp-server-wrapper.sh .gitignore
```

**Expected output:**
```
-rw-r--r--  .mcp.json
-rwxr-xr-x  .mcp-server-wrapper.sh
-rw-r--r--  .gitignore
```

**Critical check**: `.mcp-server-wrapper.sh` must show `rwx` (executable)

### 2. Verify Wrapper Script Permissions

```bash
test -x .mcp-server-wrapper.sh && echo "✅ Wrapper is executable" || echo "❌ NOT executable - Run: chmod +x .mcp-server-wrapper.sh"
```

### 3. Verify Index Directory in .gitignore

```bash
grep -q ".code-search-index/" .gitignore && echo "✅ Index excluded from git" || echo "❌ NOT in .gitignore"
```

## 🔑 Claude Code Configuration

### 4. Check Claude Code Approval Setting (CRITICAL)

```bash
# This file MUST exist with the approval setting:
mkdir -p .claude
cat .claude/settings.local.json 2>/dev/null || echo "❌ File missing - Create it!"
```

**Expected content:**
```json
{
  "enableAllProjectMcpServers": true
}
```

### 5. Verify Approval Setting

```bash
grep -q '"enableAllProjectMcpServers".*true' .claude/settings.local.json && echo "✅ MCP approval enabled" || echo "❌ Approval NOT enabled - Add enableAllProjectMcpServers: true"
```

**If missing**, create the file:
```bash
mkdir -p .claude
cat > .claude/settings.local.json << 'EOF'
{
  "enableAllProjectMcpServers": true
}
EOF
```

## 🌐 Global Installation Verification

### 6. Check FarhanAliRaza Global Installation

```bash
test -d ~/.local/share/claude-context-local && echo "✅ FarhanAliRaza installed globally" || echo "❌ NOT installed - Run install script"
```

**If missing**, install:
```bash
curl -fsSL https://raw.githubusercontent.com/FarhanAliRaza/claude-context-local/main/scripts/install.sh | bash
```

### 7. Verify MCP Server Files

```bash
test -f ~/.local/share/claude-context-local/mcp_server/server.py && echo "✅ MCP server found" || echo "❌ MCP server missing"
```

### 8. Check uv Package Manager

```bash
command -v uv >/dev/null 2>&1 && echo "✅ uv found at: $(which uv)" || echo "❌ uv NOT found - Install it"
```

**If missing**, install uv:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Then restart your terminal
```

## 🚀 Runtime Verification (After Restart)

### 9. Restart Claude Code

**CRITICAL**: You MUST completely close and reopen Claude Code for changes to take effect.

- [ ] Claude Code completely closed (all windows)
- [ ] Claude Code reopened
- [ ] Project loaded correctly

### 10. Verify MCP Server Loaded

In Claude Code, ask:
```
List available MCP tools
```

**Expected**: You should see tools prefixed with `mcp__code-search__*`, such as:
- `mcp__code-search__search`
- `mcp__code-search__index`
- `mcp__code-search__status`

**If NOT visible**:
1. Check `.claude/settings.local.json` has `enableAllProjectMcpServers: true`
2. Verify `.mcp.json` is in project root
3. Restart Claude Code again
4. Check Claude Code logs for errors

### 11. Test Indexing

In Claude Code:
```
Index this codebase
```

**Expected output** (example):
```
✅ Indexing /path/to/your-project...
✅ Found 45 files
✅ Created 312 code chunks
✅ Indexed in 8 seconds
```

**If errors**:
- Check wrapper script is executable: `ls -la .mcp-server-wrapper.sh`
- Verify global install: `ls ~/.local/share/claude-context-local/`
- Check uv is available: `which uv`

### 12. Verify Index Files Created

```bash
ls .code-search-index/projects/*/index/ 2>/dev/null
```

**Expected files:**
```
chunk_ids.pkl
embeddings.faiss
stats.json
```

### 13. Test Search Functionality

In Claude Code:
```
Find function definitions
```

**Expected**: Returns code chunks with function definitions from your project

**If no results**:
1. Verify indexing completed: "What's the indexing status?"
2. Check index files exist: `ls .code-search-index/projects/*/index/`
3. Re-index: Delete `.code-search-index/` and index again

## ✅ Full System Test

### 14. End-to-End Test

```bash
# 1. Create test file
echo "def test_function(): return 42" > test_example.py

# 2. In Claude Code, re-index:
#    "Index this codebase"

# 3. In Claude Code, search:
#    "Find test_function"

# 4. Verify it returns test_example.py

# 5. Clean up
rm test_example.py
```

## 🎯 Checklist Summary

Use this quick reference before reporting issues:

```bash
# Run all checks at once:
echo "=== File Structure ==="
ls -la .mcp.json .mcp-server-wrapper.sh .gitignore 2>/dev/null

echo -e "\n=== Wrapper Executable ==="
test -x .mcp-server-wrapper.sh && echo "✅ Yes" || echo "❌ No"

echo -e "\n=== Gitignore Entry ==="
grep -q ".code-search-index/" .gitignore && echo "✅ Present" || echo "❌ Missing"

echo -e "\n=== Claude Code Approval ==="
test -f .claude/settings.local.json && echo "✅ File exists" || echo "❌ File missing"
grep -q '"enableAllProjectMcpServers".*true' .claude/settings.local.json 2>/dev/null && echo "✅ Approval enabled" || echo "❌ Approval disabled"

echo -e "\n=== Global Installation ==="
test -d ~/.local/share/claude-context-local && echo "✅ Installed" || echo "❌ Not installed"

echo -e "\n=== uv Package Manager ==="
command -v uv >/dev/null 2>&1 && echo "✅ Found: $(which uv)" || echo "❌ Not found"

echo -e "\n=== Index Directory ==="
test -d .code-search-index && echo "✅ Created" || echo "⚠️  Not yet indexed"
```

## 🐛 Common Issues and Fixes

### Issue: "Nothing was configured or installed"

**Diagnosis**: Missing Claude Code approval setting

**Fix**:
```bash
mkdir -p .claude
cat > .claude/settings.local.json << 'EOF'
{
  "enableAllProjectMcpServers": true
}
EOF
# Then restart Claude Code
```

### Issue: "MCP server not loading"

**Diagnosis checklist**:
1. Is `.mcp.json` in project root? → `ls .mcp.json`
2. Is wrapper executable? → `ls -la .mcp-server-wrapper.sh`
3. Is approval enabled? → `cat .claude/settings.local.json`
4. Did you restart Claude Code completely?

### Issue: "Permission denied on wrapper script"

**Fix**:
```bash
chmod +x .mcp-server-wrapper.sh
```

### Issue: "Search returns no results"

**Fix**:
```bash
# Delete and rebuild index
rm -rf .code-search-index/
# Then in Claude Code: "Index this codebase"
```

## 📊 Expected Performance Benchmarks

Use these to verify normal operation:

| Metric | Small Project (<100 files) | Medium Project (1000 files) |
|--------|----------------------------|------------------------------|
| **Indexing Time** | 10-20 seconds | 1-2 minutes |
| **Index Storage** | ~5 MB | ~50 MB |
| **Search Latency** | 50-100ms | 100-200ms |
| **Chunks Created** | ~500-1500 | ~5000-15000 |

## ✨ Success Criteria

You've successfully set up FarhanAliRaza Code Search when:

- ✅ All file structure checks pass
- ✅ `.claude/settings.local.json` has `enableAllProjectMcpServers: true`
- ✅ Global installation verified
- ✅ MCP tools visible in Claude Code (`mcp__code-search__*`)
- ✅ Indexing completes without errors
- ✅ Index files created in `.code-search-index/`
- ✅ Search returns relevant results

**When all checked**: You're ready to use semantic code search! 🎉

## 📝 Reporting Issues

If setup fails after following this checklist:

1. Run the "Checklist Summary" script above
2. Copy the output
3. Note which step failed
4. Include Claude Code version
5. Report issue with all details

---

**Last Updated**: 2025-11-27
**Tested On**: macOS, Linux (Ubuntu/Debian)
**Claude Code Version**: Latest
