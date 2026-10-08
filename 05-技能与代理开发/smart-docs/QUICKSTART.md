# Smart Docs Skill - Quick Start Guide

## 🎯 Purpose

Generate comprehensive technical documentation for any codebase automatically using Claude Code.

## ✅ One-Time Setup (Already Done!)

Skill installed at: `~/.claude/skills/smart-docs/SKILL.md`

## 🚀 How to Use

### Step 1: Navigate to Your Project
```bash
cd /path/to/your/project
```

### Step 2: Tell Claude Code

In Claude Code, simply say:

**Basic:**
```
Generate comprehensive documentation for this codebase
```

**Specific:**
```
Create C4 architecture documentation with diagrams
```

**Custom:**
```
Generate documentation including:
- Project overview
- Architecture diagrams
- API reference
- Deployment guide
```

### Step 3: Done!

Claude will automatically:
1. ✅ Scan project structure
2. ✅ Identify technology stack
3. ✅ Analyze architecture patterns
4. ✅ Generate markdown docs with Mermaid diagrams
5. ✅ Create `./docs/` directory with all files

---

## 📂 Expected Output

```
./docs/
├── 1. Project Overview.md
│   - What is the project
│   - Technology stack
│   - Features
│   - Getting started
│
├── 2. Architecture Overview.md
│   - System context (C4 Level 1)
│   - Container architecture (C4 Level 2)
│   - Component architecture (C4 Level 3)
│   - Design decisions
│
├── 3. Workflow Overview.md
│   - Core workflows with sequence diagrams
│   - Data flows
│   - State management
│
└── 4. Deep Dive/
    ├── Module1.md
    ├── Module2.md
    └── ...
```

---

## 💡 Example Prompts

### For Web Applications:
```
Generate full-stack documentation for this Laravel + React application including:
- Multi-tenancy architecture
- API endpoints
- Database schema
- Frontend component structure
```

### For Microservices:
```
Document this microservice including:
- Service architecture
- API contracts
- Message flows
- Deployment diagram
```

### For Libraries/SDKs:
```
Create developer documentation including:
- Getting started
- API reference
- Code examples
- Architecture overview
```

### For Legacy Codebases:
```
Analyze this legacy codebase and create:
- Architecture reverse-engineering
- Module dependencies
- Refactoring recommendations
- Migration guide
```

---

## 🎨 Customization

### Change Output Language

```
Generate documentation in Vietnamese for this project
```

### Custom Documentation Structure

```
Generate documentation with these sections:
1. Executive Summary (business-focused)
2. Technical Architecture
3. Security Analysis
4. Performance Optimization Guide
5. Troubleshooting Guide
```

### Focus on Specific Area

```
Deep dive into the authentication system with:
- Security patterns used
- Authentication flows
- Authorization logic
- Best practices
```

---

## 🔧 Troubleshooting

### Issue: Claude doesn't recognize the skill

**Solution**: Restart Claude Code or explicitly invoke:
```
Use the smart-docs skill to generate documentation
```

### Issue: Documentation incomplete

**Solution**: For large codebases, generate in phases:
```
Phase 1: Generate overview and architecture
Phase 2: Generate workflow documentation
Phase 3: Generate deep dive for module X
```

### Issue: Wrong technology stack detected

**Solution**: Be explicit in prompt:
```
This is a Python Django project with React frontend.
Generate documentation for this stack.
```

---

## 📊 Quality Metrics

Expected quality for different project sizes:

| Project Size | Time | Lines of Docs | Diagrams | Quality |
|--------------|------|---------------|----------|---------|
| **Small** (<100 files) | 5-10 min | 500-1000 | 3-5 | Excellent |
| **Medium** (100-1000 files) | 15-25 min | 1500-2500 | 8-12 | Very Good |
| **Large** (1000-5000 files) | 30-45 min | 3000-5000 | 15-20 | Good |
| **Very Large** (>5000 files) | Multiple passes | Modular | Many | Good |

---

## 💰 Cost

**$0** - Uses your existing Claude Code subscription!

No additional costs:
- ❌ No external API calls
- ❌ No Litho binary needed
- ❌ No proxy server needed
- ✅ 100% included in subscription

---

## 🎓 Best Practices

### 1. Start Small
Test on a small module first to understand output format.

### 2. Be Specific
Tell Claude what you need:
- "Focus on backend API"
- "Include database schema"
- "Add security considerations"

### 3. Iterate
Generate basic docs first, then refine:
```
Regenerate Architecture Overview with more details on the service layer
```

### 4. Review & Edit
Claude generates 85-90% accurate docs. Always review and edit as needed.

### 5. Version Control
Commit generated docs to git:
```bash
git add docs/
git commit -m "Add generated technical documentation"
```

---

## 🔄 Updating Documentation

When codebase changes:

```
Regenerate documentation based on recent code changes
```

Or update specific sections:

```
Update the Architecture Overview to reflect new microservices
```

---

## 📚 Resources

- **Skill File**: `~/.claude/skills/smart-docs/SKILL.md`
- **This Guide**: `~/.claude/skills/smart-docs/QUICKSTART.md`
- **Examples**: See `[path-to-example-project]/docs/` for sample output

---

## 🆘 Need Help?

Ask Claude Code:
```
How do I use the smart-docs skill?
```

```
Show me examples of documentation prompts
```

```
Explain what the smart-docs skill can do
```

---

## 🎉 Success!

You now have an AI-powered documentation generator that:
- ✅ Works with ANY programming language
- ✅ Generates professional C4 diagrams
- ✅ Creates comprehensive technical docs
- ✅ Costs $0 (uses subscription)
- ✅ Takes 10-30 minutes (vs hours manually)

---

**Happy Documenting!** 🚀
