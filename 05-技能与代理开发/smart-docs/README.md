# Smart Docs - Claude Code Skill

AI-powered comprehensive codebase documentation generator for Claude Code.

## What is This?

This is a **Claude Code Skill** that enables Claude to automatically generate professional technical documentation for any codebase, including:

- 📋 Project overview
- 🏗️ Architecture diagrams (C4 model)
- 🔄 Workflow diagrams (sequence, flowcharts)
- 📊 Mermaid diagrams
- 🔍 Deep dive documentation

## Features

✅ **Zero Dependencies** - No external tools required
✅ **$0 Cost** - Uses Claude Code subscription
✅ **Multi-Language** - Works with any programming language
✅ **Professional Output** - C4 diagrams, sequence diagrams, state diagrams
✅ **Fast** - 10-30 minutes vs hours of manual writing
✅ **Customizable** - Easy to modify templates

## Installation

### Method 1: Automated Install (Recommended)

```bash
cd /path/to/deepwiki-rs
./skills/smart-docs/install.sh
```

### Method 2: Manual Install

```bash
cp -r /path/to/deepwiki-rs/skills/smart-docs ~/.claude/skills/
```

### Method 3: Manual Copy

```bash
# Create directory
mkdir -p ~/.claude/skills/smart-docs

# Copy files
cp skills/smart-docs/SKILL.md ~/.claude/skills/smart-docs/
cp skills/smart-docs/QUICKSTART.md ~/.claude/skills/smart-docs/
```

## Verification

Check if skill is installed:

```bash
ls -la ~/.claude/skills/smart-docs/
```

You should see:
```
SKILL.md
QUICKSTART.md
```

## Usage

### Basic Usage

In Claude Code, navigate to your project and say:

```
Generate comprehensive documentation for this codebase
```

### Advanced Usage

```
Create C4 architecture documentation with:
- System context diagram
- Container architecture
- Component details
- Workflow diagrams
```

### Specific Documentation

```
Document the authentication system in detail
```

```
Generate API documentation with request/response examples
```

## Expected Output

Claude will create:

```
./docs/
├── 1. Project Overview.md
├── 2. Architecture Overview.md
├── 3. Workflow Overview.md
└── 4. Deep Dive/
    └── [modules].md
```

Typical metrics:
- **Lines**: 1,500-3,000 lines of documentation
- **Diagrams**: 10-20 Mermaid diagrams
- **Time**: 10-30 minutes generation time
- **Quality**: ~85-90% of professional documentation

## Examples

See real-world example output:
```bash
# Seller project documentation (2,716 lines, 18 diagrams)
cat /path/to/seller/docs/1.\ Project\ Overview.md
```

## Customization

Edit the skill template:

```bash
code ~/.claude/skills/smart-docs/SKILL.md
```

Changes take effect immediately (no restart needed).

## Troubleshooting

### Skill not found

**Solution**: Restart Claude Code or explicitly invoke:
```
Use the smart-docs skill to document this project
```

### Incomplete documentation

**Solution**: For large codebases, generate in phases:
```
Phase 1: Generate overview
Phase 2: Generate architecture
Phase 3: Generate workflows
Phase 4: Generate deep dives
```

### Wrong tech stack detected

**Solution**: Be explicit in prompt:
```
This is a Laravel + React project. Generate documentation.
```

## Deployment to New Machine

When setting up on a new machine:

```bash
# 1. Clone deepwiki-rs repo
git clone https://github.com/sopaco/deepwiki-rs.git

# 2. Install skill
cd deepwiki-rs
./skills/smart-docs/install.sh

# 3. Done! Use Claude Code as normal
```

## Version Control

This skill is version-controlled in the `deepwiki-rs` repository:

```
deepwiki-rs/
└── skills/
    └── smart-docs/
        ├── SKILL.md           # Main skill file
        ├── QUICKSTART.md      # User guide
        ├── README.md          # This file
        └── install.sh         # Install script
```

To update skill on all machines:
1. Edit files in repo
2. Commit and push
3. On other machines: `git pull && ./skills/smart-docs/install.sh`

## Benefits Over Litho Binary

| Aspect | Smart Docs Skill | Litho Binary |
|--------|------------------|--------------|
| **Cost** | $0 (subscription) | $2-5 per run |
| **Setup** | 30 seconds | 5-10 minutes |
| **Dependencies** | None | Rust, cargo, API keys |
| **Portability** | Copy 1 directory | Build on each machine |
| **Customization** | Edit text file | Modify Rust code |
| **Maintenance** | Very low | Medium |

## FAQ

**Q: Does this replace Litho?**
A: For most use cases, yes. Use this skill for general documentation. Use Litho binary if you need exact C4 diagram formats or specialized preprocessing.

**Q: Can I use this offline?**
A: No, Claude Code requires internet connection.

**Q: How much does it cost?**
A: $0 - included in your Claude Code subscription.

**Q: Does it work with private codebases?**
A: Yes, but check your company policy on AI code analysis.

**Q: Can I customize the output?**
A: Yes, edit `~/.claude/skills/smart-docs/SKILL.md`

**Q: Does it support multiple languages?**
A: Yes, works with any programming language.

## Support

- **Quick Start**: See `QUICKSTART.md`
- **Skill Code**: See `SKILL.md`
- **Issues**: Open issue in deepwiki-rs repo

## License

Same as deepwiki-rs project.

---

**Last Updated**: 2025-11-11
**Version**: 1.0
**Maintainer**: vovanduc
