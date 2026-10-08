# rich_rust Integration Plan for dcg

## Executive Summary

This document outlines a comprehensive plan to integrate [`rich_rust`](https://github.com/Dicklesworthstone/rich_rust) into the dcg (Destructive Command Guard) codebase to achieve premium, stylish terminal output for human observers while maintaining full compatibility with AI coding agents.

**Critical Constraint:** Agent-facing output (JSON on stdout) must remain completely untouched. All rich_rust enhancements apply exclusively to human-facing output (stderr and human-readable CLI commands).

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Dependency Integration](#2-dependency-integration)
3. [Console Abstraction Layer](#3-console-abstraction-layer)
4. [Module-by-Module Integration](#4-module-by-module-integration)
5. [Feature Mapping](#5-feature-mapping)
6. [Migration Strategy](#6-migration-strategy)
7. [Agent Safety Measures](#7-agent-safety-measures)
8. [Performance Considerations](#8-performance-considerations)
9. [Testing Strategy](#9-testing-strategy)
10. [Rollout Phases](#10-rollout-phases)

---

## 1. Architecture Overview

### Current dcg Output Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         dcg CLI / Hook Mode                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  src/hook.rs │    │  src/cli.rs  │    │ src/output/mod.rs    │  │
│  │  (JSON→stdout)│    │  (commands)  │    │ (TTY detection)      │  │
│  └──────┬───────┘    └──────┬───────┘    └──────────┬───────────┘  │
│         │                   │                       │               │
│         │                   │                       ▼               │
│         │                   │            ┌──────────────────────┐   │
│         │                   │            │  src/output/theme.rs │   │
│         │                   │            │  (colors, borders)   │   │
│         │                   │            └──────────┬───────────┘   │
│         │                   │                       │               │
│         │                   ▼                       ▼               │
│         │    ┌────────────────────────────────────────────────┐    │
│         │    │  src/output/{denial,test,tables,progress}.rs   │    │
│         │    │  (DenialBox, TestResultBox, Tables, Progress)  │    │
│         │    │  Uses: comfy-table, ratatui, colored, indicatif│    │
│         │    └────────────────────────────────────────────────┘    │
│         │                                                           │
│         ▼                                                           │
│   ┌──────────┐                               ┌───────────────┐     │
│   │  STDOUT  │  ← JSON only (agents)         │    STDERR     │     │
│   │  (pure)  │                               │  (colorful)   │     │
│   └──────────┘                               └───────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Target Architecture with rich_rust

```
┌─────────────────────────────────────────────────────────────────────┐
│                         dcg CLI / Hook Mode                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  src/hook.rs │    │  src/cli.rs  │    │ src/output/mod.rs    │  │
│  │  (JSON→stdout)│    │  (commands)  │    │ (rich_rust Console)  │  │
│  └──────┬───────┘    └──────┬───────┘    └──────────┬───────────┘  │
│         │                   │                       │               │
│         │                   │                       ▼               │
│         │                   │            ┌──────────────────────┐   │
│         │                   │            │  src/output/theme.rs │   │
│         │                   │            │  (rich_rust Theme)   │   │
│         │                   │            └──────────┬───────────┘   │
│         │                   │                       │               │
│         │                   ▼                       ▼               │
│         │    ┌────────────────────────────────────────────────┐    │
│         │    │  src/output/{denial,test,tables,progress}.rs   │    │
│         │    │  rich_rust: Panel, Table, Rule, ProgressBar    │    │
│         │    │  Markup syntax: [bold red]text[/]              │    │
│         │    └────────────────────────────────────────────────┘    │
│         │                                                           │
│         ▼                                                           │
│   ┌──────────┐                               ┌───────────────┐     │
│   │  STDOUT  │  ← JSON only (UNCHANGED)      │    STDERR     │     │
│   │  (pure)  │                               │  (premium UI) │     │
│   └──────────┘                               └───────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Dependency Integration

### 2.1 Cargo.toml Changes

```toml
# Add to [dependencies]
rich_rust = { version = "0.1", features = ["full"] }

# Remove after migration (or keep as fallback)
# colored = "2.1"          # Replaced by rich_rust Style
# comfy-table = "7.2"      # Replaced by rich_rust Table
# indicatif = "0.17"       # Replaced by rich_rust ProgressBar
# ratatui = { ... }        # Keep for now, gradual migration
```

### 2.2 Feature Flag Strategy

Create a feature flag for gradual rollout:

```toml
[features]
default = []
rich-output = ["dep:rich_rust"]
legacy-output = []  # Keep old rendering for compatibility testing
```

### 2.3 Conditional Compilation

```rust
#[cfg(feature = "rich-output")]
use rich_rust::prelude::*;

#[cfg(not(feature = "rich-output"))]
use crate::output::legacy::*;
```

---

## 3. Console Abstraction Layer

### 3.1 Create `src/output/console.rs`

This module wraps rich_rust's Console with dcg-specific defaults and stderr routing:

```rust
//! Console abstraction for dcg output.
//!
//! Provides a unified interface for all human-facing output, automatically
//! routing to stderr and detecting terminal capabilities.

use rich_rust::prelude::*;
use std::io::{self, Write};
use std::sync::OnceLock;

/// Global console instance for human-facing output.
static CONSOLE: OnceLock<DcgConsole> = OnceLock::new();

/// dcg-specific console wrapper.
pub struct DcgConsole {
    inner: Console,
    force_plain: bool,
}

impl DcgConsole {
    /// Create a new console that writes to stderr.
    pub fn new() -> Self {
        let inner = Console::new()
            .with_stderr()  // Critical: all output goes to stderr
            .with_force_terminal(false);  // Respect TTY detection

        Self {
            inner,
            force_plain: false,
        }
    }

    /// Create a plain-text console (no colors, no unicode).
    pub fn plain() -> Self {
        let inner = Console::new()
            .with_stderr()
            .with_no_color();

        Self {
            inner,
            force_plain: true,
        }
    }

    /// Print styled text using markup syntax.
    pub fn print(&self, text: &str) {
        if self.force_plain {
            self.inner.print_plain(text);
        } else {
            self.inner.print(text);
        }
    }

    /// Print a renderable (Panel, Table, Rule, etc.).
    pub fn print_renderable<R: Renderable>(&self, renderable: &R) {
        self.inner.print_renderable(renderable);
    }

    /// Print a horizontal rule with optional title.
    pub fn rule(&self, title: Option<&str>) {
        self.inner.rule(title);
    }

    /// Get terminal width.
    pub fn width(&self) -> usize {
        self.inner.width()
    }
}

/// Get the global console instance.
pub fn console() -> &'static DcgConsole {
    CONSOLE.get_or_init(|| {
        if crate::output::should_use_rich_output() {
            DcgConsole::new()
        } else {
            DcgConsole::plain()
        }
    })
}

/// Initialize the console with explicit settings.
pub fn init_console(force_plain: bool) {
    let _ = CONSOLE.set(if force_plain {
        DcgConsole::plain()
    } else {
        DcgConsole::new()
    });
}
```

---

## 4. Module-by-Module Integration

### 4.1 `src/output/denial.rs` - DenialBox → rich_rust Panel

**Current Implementation:** Manual ANSI escape codes, custom box drawing

**Target Implementation:** rich_rust Panel with styled content

```rust
use rich_rust::prelude::*;

pub struct DenialBox {
    command: String,
    reason: String,
    pack: Option<String>,
    pattern: Option<String>,
    explanation: Option<String>,
    severity: Option<Severity>,
    allow_once_code: Option<String>,
}

impl DenialBox {
    pub fn render(&self, theme: &DcgTheme) -> String {
        let console = Console::new().with_stderr();

        // Build styled content using markup
        let mut content = Text::new();

        // Header with severity coloring
        let severity_style = match self.severity {
            Some(Severity::Critical) => "[bold bright_red]",
            Some(Severity::High) => "[bold red]",
            Some(Severity::Medium) => "[bold yellow]",
            Some(Severity::Low) => "[bold blue]",
            None => "[bold]",
        };

        content.push_line(format!("{severity_style}🛑 COMMAND BLOCKED[/]"));
        content.push_line("");

        // Command with highlighting
        content.push_line(format!("[dim]Command:[/]  [bold]{cmd}[/]", cmd = self.command));

        if let Some(reason) = &self.reason {
            content.push_line(format!("[dim]Reason:[/]   {reason}"));
        }

        if let Some(pack) = &self.pack {
            content.push_line(format!("[dim]Pack:[/]     [cyan]{pack}[/]"));
        }

        if let Some(pattern) = &self.pattern {
            content.push_line(format!("[dim]Pattern:[/]  [magenta]{pattern}[/]"));
        }

        if let Some(explanation) = &self.explanation {
            content.push_line("");
            content.push_line(format!("[italic]{explanation}[/]"));
        }

        // Allow-once code section
        if let Some(code) = &self.allow_once_code {
            content.push_line("");
            content.push_line("[dim]─────────────────────────────────────[/]");
            content.push_line(format!(
                "[yellow]To allow once:[/] [bold]dcg allow-once {code}[/]"
            ));
        }

        // Create panel with severity-appropriate border
        let box_style = match self.severity {
            Some(Severity::Critical) => BoxStyle::double(),
            Some(Severity::High) => BoxStyle::heavy(),
            _ => BoxStyle::rounded(),
        };

        let border_color = match self.severity {
            Some(Severity::Critical) => "bright_red",
            Some(Severity::High) => "red",
            Some(Severity::Medium) => "yellow",
            Some(Severity::Low) => "blue",
            None => "red",
        };

        Panel::from_text(content.to_string())
            .title("[bold] DCG [/]")
            .border_style(Style::parse(border_color).unwrap_or_default())
            .box_style(box_style)
            .padding((1, 2))
            .to_string()
    }
}
```

### 4.2 `src/output/test.rs` - TestResultBox → rich_rust Panel

**Current Implementation:** Similar manual rendering

**Target Implementation:**

```rust
use rich_rust::prelude::*;

impl TestResultBox {
    pub fn render(&self, theme: &DcgTheme) -> String {
        let (title, border_style, header_color) = match &self.result {
            TestOutcome::Blocked { .. } => (
                " WOULD BE BLOCKED ",
                BoxStyle::heavy(),
                "bold bright_red",
            ),
            TestOutcome::Allowed { .. } => (
                " WOULD BE ALLOWED ",
                BoxStyle::rounded(),
                "bold bright_green",
            ),
        };

        let mut content = Text::new();

        // Command line
        content.push_line(format!(
            "[dim]Command:[/]     [bold]{cmd}[/]",
            cmd = self.command
        ));

        // Result-specific content
        match &self.result {
            TestOutcome::Blocked { pattern_id, pack_id, severity, reason, confidence } => {
                if let Some(pattern) = pattern_id {
                    content.push_line(format!("[dim]Pattern:[/]     [magenta]{pattern}[/]"));
                }
                if let Some(pack) = pack_id {
                    let sev = severity.map(|s| format!(" [dim]({})[/]", severity_label(s)))
                        .unwrap_or_default();
                    content.push_line(format!("[dim]Pack:[/]        [cyan]{pack}[/]{sev}"));
                }
                if let Some(conf) = confidence {
                    let bar = render_confidence_bar(*conf);
                    content.push_line(format!("[dim]Confidence:[/]  {bar} {conf:.0}%", conf = conf * 100.0));
                }
                content.push_line(format!("[dim]Reason:[/]      {reason}"));
            }
            TestOutcome::Allowed { reason } => {
                let reason_text = match reason {
                    AllowedReason::NoPatternMatch => "No pattern matches".to_string(),
                    AllowedReason::AllowlistMatch { entry, layer } => {
                        format!("Allowlist: [italic]\"{entry}\"[/] ({layer})")
                    }
                    AllowedReason::BudgetExhausted => {
                        "[yellow]Budget exhausted (fail-open)[/]".to_string()
                    }
                };
                content.push_line(format!("[dim]Reason:[/]      {reason_text}"));
            }
        }

        Panel::from_text(content.to_string())
            .title(format!("[{header_color}]{title}[/]"))
            .box_style(border_style)
            .border_style(Style::parse(header_color).unwrap_or_default())
            .padding((1, 2))
            .to_string()
    }
}

/// Render a visual confidence bar using Unicode blocks
fn render_confidence_bar(confidence: f64) -> String {
    let filled = (confidence * 10.0).round() as usize;
    let empty = 10 - filled;

    let color = if confidence >= 0.8 {
        "red"
    } else if confidence >= 0.5 {
        "yellow"
    } else {
        "green"
    };

    format!(
        "[{color}]{}[/][dim]{}[/]",
        "█".repeat(filled),
        "░".repeat(empty)
    )
}
```

### 4.3 `src/output/tables.rs` - comfy-table → rich_rust Table

**Current Implementation:** comfy-table with ratatui colors

**Target Implementation:**

```rust
use rich_rust::prelude::*;

/// Table renderer for scan results.
pub struct ScanResultsTable {
    rows: Vec<ScanResultRow>,
    theme: Option<DcgTheme>,
}

impl ScanResultsTable {
    pub fn render(&self) -> String {
        if self.rows.is_empty() {
            return "[dim]No findings.[/]".to_string();
        }

        let mut table = Table::new()
            .title("[bold]Scan Results[/]")
            .with_column(Column::new("File").style(Style::new().cyan()))
            .with_column(Column::new("Line").justify(JustifyMethod::Right))
            .with_column(Column::new("Severity").justify(JustifyMethod::Center))
            .with_column(Column::new("Pattern"));

        for row in &self.rows {
            let severity_cell = match row.severity {
                Severity::Critical => "[bold bright_red]CRIT[/]",
                Severity::High => "[red]HIGH[/]",
                Severity::Medium => "[yellow]MED[/]",
                Severity::Low => "[blue]LOW[/]",
            };

            table.add_row_cells([
                &row.file,
                &row.line.to_string(),
                severity_cell,
                &row.pattern_id,
            ]);
        }

        table.to_string()
    }
}

/// Pack listing table.
pub struct PackListTable {
    rows: Vec<PackRow>,
}

impl PackListTable {
    pub fn render(&self) -> String {
        if self.rows.is_empty() {
            return "[dim]No packs available.[/]".to_string();
        }

        let mut table = Table::new()
            .title("[bold]Available Packs[/]")
            .with_column(Column::new("Pack ID").style(Style::new().cyan()))
            .with_column(Column::new("Name"))
            .with_column(Column::new("Destructive").justify(JustifyMethod::Right))
            .with_column(Column::new("Safe").justify(JustifyMethod::Right))
            .with_column(Column::new("Status").justify(JustifyMethod::Center));

        for row in &self.rows {
            let status = if row.enabled {
                "[green]●[/] enabled"
            } else {
                "[dim]○[/] disabled"
            };

            table.add_row_cells([
                &row.id,
                &row.name,
                &row.destructive_count.to_string(),
                &row.safe_count.to_string(),
                status,
            ]);
        }

        table.to_string()
    }
}
```

### 4.4 `src/output/progress.rs` - indicatif → rich_rust ProgressBar

```rust
use rich_rust::prelude::*;

/// Progress display for long-running operations.
pub struct ScanProgress {
    total: usize,
    current: usize,
    current_file: String,
}

impl ScanProgress {
    pub fn render(&self) -> String {
        let bar = ProgressBar::new()
            .completed(self.current)
            .total(self.total)
            .width(40)
            .style(Style::new().cyan());

        let pct = (self.current as f64 / self.total as f64 * 100.0) as usize;

        format!(
            "{bar}  [bold]{pct}%[/]  [dim]{file}[/]",
            bar = bar.to_string(),
            pct = pct,
            file = truncate(&self.current_file, 30)
        )
    }
}
```

### 4.5 `src/hook.rs` - print_colorful_warning Integration

**Critical:** This function outputs to stderr for human consumption while the JSON verdict goes to stdout.

```rust
use crate::output::console::console;
use rich_rust::prelude::*;

/// Print a colorful warning to stderr about a blocked command.
pub fn print_colorful_warning(
    command: &str,
    reason: &str,
    pack: Option<&str>,
    pattern: Option<&str>,
    explanation: Option<&str>,
    allow_once_code: Option<&str>,
    matched_span: Option<&MatchSpan>,
    pattern_suggestions: &[PatternSuggestion],
) {
    let console = console();

    // Build the denial panel
    let denial = DenialBox {
        command: command.to_string(),
        reason: reason.to_string(),
        pack: pack.map(String::from),
        pattern: pattern.map(String::from),
        explanation: explanation.map(String::from),
        severity: determine_severity(pack, pattern),
        allow_once_code: allow_once_code.map(String::from),
    };

    // Render to stderr via Console
    eprintln!("{}", denial.render(&DcgTheme::auto()));

    // Pattern suggestions as a secondary panel
    if !pattern_suggestions.is_empty() {
        let suggestions = render_suggestions_panel(pattern_suggestions);
        eprintln!("{}", suggestions);
    }
}

fn render_suggestions_panel(suggestions: &[PatternSuggestion]) -> String {
    let mut content = Text::new();

    for (i, suggestion) in suggestions.iter().enumerate() {
        content.push_line(format!(
            "[bold cyan]{i}.[/] {desc}",
            i = i + 1,
            desc = suggestion.description
        ));
        if let Some(example) = &suggestion.example {
            content.push_line(format!("   [dim]Example:[/] [italic]{example}[/]"));
        }
    }

    Panel::from_text(content.to_string())
        .title("[yellow bold] 💡 Suggestions [/]")
        .box_style(BoxStyle::rounded())
        .border_style(Style::new().yellow())
        .to_string()
}
```

### 4.6 `src/cli.rs` - Command Output Formatting

#### doctor command

```rust
fn run_doctor(fix: bool, format: DoctorFormat) -> Result<()> {
    let console = console();

    console.rule(Some("[bold] dcg doctor [/]"));
    console.print("");

    let checks = vec![
        ("Configuration", check_config()),
        ("Hook Registration", check_hook()),
        ("Pack Loading", check_packs()),
        ("Permissions", check_permissions()),
    ];

    let mut all_passed = true;

    for (name, result) in checks {
        let (icon, status, color) = match result {
            CheckResult::Pass => ("✓", "PASS", "green"),
            CheckResult::Warn(msg) => {
                all_passed = false;
                ("⚠", &format!("WARN: {msg}"), "yellow")
            }
            CheckResult::Fail(msg) => {
                all_passed = false;
                ("✗", &format!("FAIL: {msg}"), "red")
            }
        };

        console.print(&format!(
            "  [{color}]{icon}[/] {name:<20} [{color}]{status}[/]"
        ));
    }

    console.print("");

    if all_passed {
        console.print("[bold green]All checks passed![/]");
    } else {
        console.print("[yellow]Some issues detected.[/] Run with --fix to attempt repairs.");
    }

    Ok(())
}
```

#### packs command

```rust
fn run_list_packs(enabled: bool, format: PacksFormat) -> Result<()> {
    match format {
        PacksFormat::Json => {
            // JSON output to stdout (agent-safe)
            let packs = collect_packs(enabled);
            println!("{}", serde_json::to_string_pretty(&packs)?);
        }
        PacksFormat::Pretty => {
            let console = console();

            console.rule(Some("[bold] Available Packs [/]"));
            console.print("");

            let rows = collect_pack_rows(enabled);
            let table = PackListTable::new(rows);

            console.print(&table.render());

            // Summary footer
            let total = rows.len();
            let enabled_count = rows.iter().filter(|r| r.enabled).count();

            console.print("");
            console.print(&format!(
                "[dim]Total:[/] {total} packs ({enabled_count} enabled, {} disabled)",
                total - enabled_count
            ));
        }
    }

    Ok(())
}
```

#### explain command

```rust
fn run_explain(command: &str, format: ExplainFormat) -> Result<()> {
    match format {
        ExplainFormat::Json => {
            // JSON to stdout
            let trace = explain_command(command)?;
            println!("{}", serde_json::to_string_pretty(&trace)?);
        }
        ExplainFormat::Pretty => {
            let console = console();

            console.rule(Some("[bold] Decision Trace [/]"));

            let trace = explain_command(command)?;

            // Render as a tree structure
            let mut tree = TreeNode::new(format!(
                "[bold]Command:[/] {}",
                truncate(command, 60)
            ));

            // Add decision stages
            let keyword_node = TreeNode::new(format!(
                "[{}]Keyword Gate[/]: {}",
                if trace.keyword_matched { "green" } else { "dim" },
                if trace.keyword_matched { "matched" } else { "no match" }
            ));
            tree.add_child(keyword_node);

            if let Some(pack_eval) = &trace.pack_evaluation {
                let mut pack_node = TreeNode::new(format!(
                    "[cyan]Pack Evaluation[/]: {}",
                    pack_eval.pack_id
                ));

                for pattern in &pack_eval.patterns_checked {
                    let (icon, color) = if pattern.matched {
                        ("●", "red")
                    } else {
                        ("○", "dim")
                    };
                    pack_node.add_child(TreeNode::new(format!(
                        "[{color}]{icon}[/] {}",
                        pattern.name
                    )));
                }

                tree.add_child(pack_node);
            }

            // Final decision
            let decision_node = TreeNode::new(format!(
                "[bold {}]Decision:[/] {}",
                if trace.decision == "deny" { "red" } else { "green" },
                trace.decision.to_uppercase()
            ));
            tree.add_child(decision_node);

            let tree_widget = Tree::new(tree);
            console.print_renderable(&tree_widget);
        }
    }

    Ok(())
}
```

---

## 5. Feature Mapping

### 5.1 rich_rust Feature → dcg Usage

| rich_rust Feature | dcg Usage Area | Priority |
|-------------------|----------------|----------|
| `Console` | Global output coordinator | P0 |
| `Panel` | DenialBox, TestResultBox, help panels | P0 |
| `Table` | Pack listings, scan results, stats | P0 |
| `Rule` | Section dividers in CLI output | P1 |
| `Style` + Markup | All colored text (replaces `colored`) | P0 |
| `Tree` | explain command, decision traces | P1 |
| `ProgressBar` | scan command, long operations | P2 |
| `Columns` | Side-by-side comparisons | P2 |
| `Syntax` (feature) | Heredoc highlighting in explain | P3 |
| `Markdown` (feature) | Help text rendering | P3 |

### 5.2 Markup Syntax Reference

```rust
// Basic styles
"[bold]Bold text[/]"
"[italic]Italic text[/]"
"[underline]Underlined[/]"
"[dim]Muted text[/]"

// Colors
"[red]Red text[/]"
"[green]Green text[/]"
"[yellow]Warning[/]"
"[cyan]Info[/]"
"[magenta]Pattern name[/]"

// Combined
"[bold red]Critical error[/]"
"[dim italic]Explanatory text[/]"

// Backgrounds
"[on red]Highlighted[/]"
"[white on red] BLOCKED [/]"

// Hex/RGB (if terminal supports)
"[#ff6600]Orange text[/]"
"[rgb(100,150,200)]Custom color[/]"
```

---

## 6. Migration Strategy

### 6.1 Phase 1: Foundation (Week 1)

1. Add `rich_rust` dependency with `full` feature
2. Create `src/output/console.rs` abstraction
3. Create `src/output/rich_theme.rs` mapping dcg themes to rich_rust
4. Add `--legacy-output` flag for fallback

### 6.2 Phase 2: Core Components (Week 2)

1. Migrate `denial.rs` to use rich_rust Panel
2. Migrate `test.rs` to use rich_rust Panel
3. Update `hook.rs` `print_colorful_warning` function
4. Ensure all stderr output uses Console

### 6.3 Phase 3: Tables & Progress (Week 3)

1. Migrate `tables.rs` from comfy-table to rich_rust Table
2. Update `progress.rs` to use rich_rust ProgressBar
3. Update CLI commands: `packs`, `stats`, `scan`

### 6.4 Phase 4: Advanced Features (Week 4)

1. Add Tree rendering for `explain` command
2. Add Syntax highlighting for heredoc content
3. Add Markdown rendering for `--help` output
4. Performance optimization pass

### 6.5 Phase 5: Cleanup (Week 5)

1. Remove unused dependencies (comfy-table, indicatif if fully replaced)
2. Remove legacy output code paths
3. Update documentation
4. Final testing across terminal types

### 6.6 Phase 7 Cleanup Gate Status

The Phase 7 cleanup work is now split into separate beads so each risky cleanup
can be verified independently. The umbrella gate (`bd-3jl0`) should be treated as
the handoff point from the validated rich-output implementation to those child
cleanup tasks, not as permission to batch dependency removal, docs rewrites, and
feature-default changes into one commit.

| Bead | Cleanup Surface | Status |
|------|-----------------|--------|
| `bd-391u` | Remove `comfy-table` only after markdown/table fallbacks are accounted for | Follow-up |
| `bd-v7lj` | Remove unused legacy output paths while preserving non-rich builds | Follow-up |
| `bd-3fs5` | Refresh README rich-output examples and screenshots | Follow-up |
| `bd-c0di` | Document `DCG_NO_RICH`, `NO_COLOR`, `DCG_NO_COLOR`, `TERM=dumb`, and `CI` behavior | Follow-up |
| `bd-2uor` | Add agent-compatibility notes for stdout JSON and stderr human output | Follow-up |
| `bd-15p0` | Consider making `rich-output` default after the cleanup and docs beads land | Follow-up |

This keeps Phase 7 aligned with the safety rule for dcg output: agent-facing JSON
contracts stay stable while human-facing rich output can improve behind explicit
feature and environment controls.

---

## 7. Agent Safety Measures

### 7.1 Output Channel Separation

**Rule:** JSON output to stdout MUST remain pure JSON. All rich output goes to stderr.

```rust
// CORRECT: Human output to stderr
eprintln!("{}", panel.render());

// CORRECT: Agent output to stdout
println!("{}", serde_json::to_string(&verdict)?);

// WRONG: Never mix
println!("{}", panel.render()); // ❌ Don't send rich output to stdout
```

### 7.2 Format Detection

```rust
impl Cli {
    /// Determine if we're in agent mode (JSON output expected).
    fn is_agent_mode(&self) -> bool {
        // Check for JSON format flag
        if self.json_output() {
            return true;
        }

        // Check for stdin from Claude Code hook
        if std::env::var("CLAUDE_CODE_HOOK").is_ok() {
            return true;
        }

        // Check for piped stdout (non-TTY)
        if !std::io::stdout().is_terminal() {
            return true;
        }

        false
    }
}
```

### 7.3 Conditional Output

```rust
fn output_result(result: &EvaluationResult, cli: &Cli) {
    if cli.is_agent_mode() {
        // Pure JSON to stdout
        let json = serde_json::to_string(result).unwrap();
        println!("{json}");
    } else {
        // Rich output to stderr
        let panel = TestResultBox::from_evaluation(&result.command, result);
        eprintln!("{}", panel.render(&DcgTheme::auto()));
    }
}
```

### 7.4 Environment Variable Controls

```rust
// Respect NO_COLOR (https://no-color.org/)
if std::env::var("NO_COLOR").is_ok() {
    console = Console::plain();
}

// Respect CI environments
if std::env::var("CI").is_ok() {
    console = Console::plain();
}

// dcg-specific disable
if std::env::var("DCG_NO_RICH").is_ok() {
    console = Console::plain();
}
```

---

## 8. Performance Considerations

### 8.1 Hook Mode Latency Budget

dcg operates under strict latency constraints:

| Tier | Budget | Current | Target |
|------|--------|---------|--------|
| Instant | <5ms | ✓ | ✓ |
| Fast | <15ms | ✓ | ✓ |
| Normal | <50ms | ✓ | ✓ |

**rich_rust Impact Assessment:**

- Console initialization: ~1ms (one-time)
- Panel rendering: <1ms per panel
- Table rendering: <2ms for typical sizes
- Markup parsing: <0.5ms per string

**Mitigation:**

1. Lazy initialization of Console
2. Pre-compute static panels at startup
3. Cache terminal width detection
4. Avoid rendering if output is suppressed

### 8.2 Memory Considerations

rich_rust uses heap allocation for:
- Segment vectors (per-line)
- Style structs (small, stack-optimizable)
- String building

For hook mode where memory matters:
- Reuse Console instance (already planned)
- Avoid repeated allocations in tight loops
- Consider `smallvec` for small segment lists

### 8.3 Benchmarking

Add benchmark comparing old vs new rendering:

```rust
// benches/output_perf.rs
use criterion::{criterion_group, criterion_main, Criterion};

fn bench_denial_box(c: &mut Criterion) {
    let denial = DenialBox::new(/* test data */);

    c.bench_function("denial_box_legacy", |b| {
        b.iter(|| denial.render_legacy(&theme))
    });

    c.bench_function("denial_box_rich", |b| {
        b.iter(|| denial.render_rich(&theme))
    });
}
```

---

## 9. Testing Strategy

### 9.1 Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn denial_box_renders_with_all_fields() {
        let denial = DenialBox {
            command: "rm -rf /".to_string(),
            reason: "Recursive deletion of root".to_string(),
            pack: Some("core.filesystem".to_string()),
            pattern: Some("rm-rf-root".to_string()),
            explanation: Some("This would destroy the system".to_string()),
            severity: Some(Severity::Critical),
            allow_once_code: Some("abc123".to_string()),
        };

        let output = denial.render(&DcgTheme::default());

        assert!(output.contains("COMMAND BLOCKED"));
        assert!(output.contains("rm -rf /"));
        assert!(output.contains("core.filesystem"));
        assert!(output.contains("dcg allow-once abc123"));
    }

    #[test]
    fn denial_box_respects_no_color() {
        let denial = DenialBox::minimal();
        let theme = DcgTheme::no_color();

        let output = denial.render(&theme);

        // No ANSI escape codes
        assert!(!output.contains("\x1b["));
    }

    #[test]
    fn table_handles_empty_input() {
        let table = ScanResultsTable::new(vec![]);
        let output = table.render();

        assert!(output.contains("No findings"));
    }
}
```

### 9.2 Integration Tests

```rust
#[test]
fn hook_mode_outputs_json_to_stdout() {
    let output = Command::new("dcg")
        .args(["hook"])
        .write_stdin(r#"{"tool_name":"Bash","tool_input":{"command":"rm -rf /"}}"#)
        .output()
        .unwrap();

    // stdout should be valid JSON
    let stdout = String::from_utf8_lossy(&output.stdout);
    let json: serde_json::Value = serde_json::from_str(&stdout).unwrap();

    assert_eq!(json["decision"], "deny");

    // stderr should have the colorful warning (for humans)
    let stderr = String::from_utf8_lossy(&output.stderr);
    assert!(stderr.contains("BLOCKED") || stderr.contains("blocked"));
}
```

### 9.3 Visual Regression Tests

Create reference screenshots for different terminal types:
- iTerm2 (macOS)
- Windows Terminal
- GNOME Terminal (Linux)
- xterm (minimal)

### 9.4 Accessibility Tests

```rust
#[test]
fn output_works_with_screen_reader() {
    // Test with NO_COLOR set
    std::env::set_var("NO_COLOR", "1");

    let denial = DenialBox::minimal();
    let output = denial.render(&DcgTheme::auto());

    // Should be readable without color codes
    assert!(!output.contains("\x1b["));
    assert!(output.contains("BLOCKED")); // Key information preserved
}
```

---

## 10. Rollout Phases

### 10.1 Alpha (Internal Testing)

- Feature flag: `rich-output` (opt-in)
- Duration: 1 week
- Criteria: No regressions in existing tests

### 10.2 Beta (Opt-In Users)

- Feature flag: build with `--features rich-output`
- Runtime controls: rich rendering is used only when the output is TTY-safe and
  the disabling environment variables are absent
- Disable rich formatting with `DCG_NO_RICH=1`, `NO_COLOR=1`, `DCG_NO_COLOR=1`,
  `TERM=dumb`, `CI=1`, or the global `--legacy-output` / `DCG_LEGACY_OUTPUT=1`
  fallback
- Collect feedback via GitHub issues before changing default features

### 10.3 General Availability

- Default-enabled rich output remains gated by `bd-15p0`
- Keep the legacy fallback until non-rich builds and stdout/stderr separation
  have dedicated verification on the final default-feature configuration
- Remove legacy code only through `bd-v7lj`, after confirming no markdown,
  scan, or agent-output contract still depends on it

---

## Appendix A: Color Palette Mapping

| dcg Theme Color | rich_rust Equivalent | Hex |
|-----------------|---------------------|-----|
| `error_color` | `bright_red` | #FF5555 |
| `warning_color` | `yellow` | #F1FA8C |
| `success_color` | `bright_green` | #50FA7B |
| `info_color` | `cyan` | #8BE9FD |
| `muted_color` | `dim` | #6272A4 |
| `critical_severity` | `bold bright_red` | #FF5555 |
| `high_severity` | `red` | #FF6E6E |
| `medium_severity` | `yellow` | #F1FA8C |
| `low_severity` | `blue` | #8BE9FD |

---

## Appendix B: Box Style Reference

| dcg BorderStyle | rich_rust BoxStyle | Characters |
|-----------------|-------------------|------------|
| `Unicode` | `rounded()` | ╭─╮ │ ╰─╯ |
| `Ascii` | `ascii()` | +-+ | +-+ |
| `None` | `minimal()` | (no borders) |
| (critical) | `double()` | ╔═╗ ║ ╚═╝ |
| (high) | `heavy()` | ┏━┓ ┃ ┗━┛ |

---

## Appendix C: Dependency Comparison

### Before Migration

```toml
colored = "2.1"           # Basic ANSI colors
comfy-table = "7.2"       # Table rendering
indicatif = "0.17"        # Progress bars
ratatui = "0.30"          # TUI colors (partial use)
console = "0.15"          # Terminal detection
```

### After Migration

```toml
rich_rust = { version = "0.1", features = ["full"] }
# ratatui removed (colors now via rich_rust)
# colored removed (markup syntax)
# comfy-table removed (rich_rust Table)
# indicatif removed (rich_rust ProgressBar)
console = "0.15"          # Keep for TTY detection (or use rich_rust's)
```

**Binary size impact:** Estimated neutral to slight reduction (removing multiple crates, adding one comprehensive one).

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-01-19 | Claude | Initial comprehensive plan |
