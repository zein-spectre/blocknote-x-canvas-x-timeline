# AI Agent Guidelines

## Rules for AI Working on This Project
1. **Never make unsupported success claims.**
2. **Reproduce problems before fixing them.**
3. **Use one hypothesis at a time.**
4. **Verify every fix.**
5. **Never repeat a failed approach unless new evidence justifies it.**
6. **After every failed debugging attempt, immediately record it in DEBUG_LOG.md.**
7. **After a successful fix, record the root cause, final fix, and verification in DEBUG_LOG.md.**
8. **If debugging reaches 3 consecutive failures, stop making random changes and enter investigation mode.**
9. **If codebase-memory-mcp tool is available, use `search_graph`/`trace_call_path` to explore code structure before grepping manually or reading many files.**
