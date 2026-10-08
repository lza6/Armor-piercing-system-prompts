# Quick Design Workflow

Rapid design creation with minimal planning overhead.

## Prerequisites
- Invoke the `ui-ux-designer` subagent first for design intelligence

## Workflow Steps

### 1. Start Design Process
Use `ui-ux-designer` subagent directly:
- Skip extensive planning
- Move to implementation quickly
- Make design decisions on-the-fly

### 2. Implement
- Default to HTML/CSS/JS if unspecified
- Focus on core functionality
- Maintain quality despite speed

### 3. Generate Assets
Use Claude's built-in multimodal vision:
- Generate required visuals
- Verify quality quickly
- Use imagemagick CLI or browser Canvas API for adjustments

### 4. Report & Approve
- Summarize changes briefly
- Request user approval
- Update the project's UI/UX context doc (if present) if the approved pattern should become durable project knowledge

## When to Use
- Simple components
- Prototypes and MVPs
- Time-constrained projects
- Iterative exploration
- Single-page designs

## Quality Shortcuts
While moving fast, maintain:
- Semantic HTML
- CSS variables for consistency
- Basic accessibility
- Clean code structure

## Related
- `workflow-immersive.md` - For comprehensive designs
- `technical-overview.md` - Quick reference
