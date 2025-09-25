---
applyTo: '**'
---
# GitHub Copilot Context Instructions

## Context Exclusion Rules

### **EXCLUDE from ALL responses:**
- `docs/implementation-guidelines.md` - User reference file, never use as context

### **Reasoning:**
Implementation guidelines contain prompts FOR Copilot, not project requirements. User keeps file open for copying prompts but content should NOT influence responses.

### **Rule:**
When `docs/implementation-guidelines.md` is active, treat as if it doesn't exist for context. Focus only on user's actual request and existing project code.

---