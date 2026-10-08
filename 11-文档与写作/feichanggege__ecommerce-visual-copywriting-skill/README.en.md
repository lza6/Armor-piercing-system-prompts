<div align="center">

# E-commerce Visual Copywriting Skill

[中文](README.md) · **English**

> Turn product inputs into executable main-image and detail-page storyboards, on-image copy, design notes, and image-generation prompts.

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-ecommerce--visual--copywriting-blueviolet)](SKILL.md)
[![skills.sh](https://skills.sh/b/feichanggege/ecommerce-visual-copywriting-skill)](https://skills.sh/feichanggege/ecommerce-visual-copywriting-skill)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Verify](https://img.shields.io/badge/verify-python%20tools%2Fverify--skill.py-2ea44f)](tools/verify-skill.py)

**Given product information, qualification boundaries, platform requirements, and visual references, this skill produces a visual strategy brief and storyboard first, then final scenes, on-image copy, design notes, and prompts after confirmation.**

[Showcase](#showcase) · [Quick Start](#quick-start) · [Triggers](#triggers) · [Deliverables](#deliverables) · [Safety Boundaries](#safety-boundaries) · [Verification](#verification)

</div>

---

## Showcase

These images show the intended execution style: visual interception, multi-angle product views, scene-based detail modules, and texture-led selling points.

<div align="center">
  <table>
    <tr>
      <td align="center" width="50%">
        <a href="assets/showcase/musang-king-durian-main.jpg">
          <img src="assets/showcase/thumbs/musang-king-durian-main-thumb.jpg" alt="Musang King durian hero image" width="360" height="360">
        </a><br>
        <strong>Premium food hero image</strong>
      </td>
      <td align="center" width="50%">
        <a href="assets/showcase/figure-multi-angle.png">
          <img src="assets/showcase/thumbs/figure-multi-angle-thumb.jpg" alt="Figure multi-angle product view" width="360" height="360">
        </a><br>
        <strong>Multi-angle product view</strong>
      </td>
    </tr>
    <tr>
      <td align="center" width="50%">
        <a href="assets/showcase/figure-desktop-scene.png">
          <img src="assets/showcase/thumbs/figure-desktop-scene-thumb.jpg" alt="Desktop display scene" width="360" height="360">
        </a><br>
        <strong>Desktop scene detail</strong>
      </td>
      <td align="center" width="50%">
        <a href="assets/showcase/lumina-pendant-lamp.png">
          <img src="assets/showcase/thumbs/lumina-pendant-lamp-thumb.jpg" alt="Glass pendant lamp texture image" width="360" height="360">
        </a><br>
        <strong>Home lighting texture expression</strong>
      </td>
    </tr>
  </table>
</div>

> Showcase images demonstrate visual output formats. They are not legal opinions, platform approval guarantees, or proof of commercial claims.

---

## Problem

Most AI-generated e-commerce copy fails in three ways:

- It jumps into final copy before direction is validated.
- It sounds aggressive but risks ad-law, health-claim, or platform-review issues.
- It gives copy only, without scene direction, visual hierarchy, material lighting, disclaimers, or designer handoff notes.

This skill turns e-commerce visual copywriting into a gated SOP: identify the conversion driver, lock the Campaign Style Lock, produce main-image and detail-page storyboards, then move into final copy, scene notes, design notes, and prompts only after user confirmation.

## Quick Start

```bash
npx skills add feichanggege/ecommerce-visual-copywriting-skill
```

Then tell your Agent:

```text
Create an e-commerce main-image and detail-page visual plan for this product. Start with the visual strategy brief and storyboard; wait for my confirmation before final copy and image-generation prompts.
```

### Manual Install

```bash
git clone https://github.com/feichanggege/ecommerce-visual-copywriting-skill.git
mkdir -p ~/.codex/skills/ecommerce-visual-copywriting
cp -r ecommerce-visual-copywriting-skill/SKILL.md ecommerce-visual-copywriting-skill/SKILL.en.md ecommerce-visual-copywriting-skill/references ecommerce-visual-copywriting-skill/examples ~/.codex/skills/ecommerce-visual-copywriting/
```

Windows PowerShell:

```powershell
git clone https://github.com/feichanggege/ecommerce-visual-copywriting-skill.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills\ecommerce-visual-copywriting" | Out-Null
Copy-Item -Recurse ecommerce-visual-copywriting-skill\SKILL.md,ecommerce-visual-copywriting-skill\SKILL.en.md,ecommerce-visual-copywriting-skill\references,ecommerce-visual-copywriting-skill\examples "$env:USERPROFILE\.codex\skills\ecommerce-visual-copywriting\"
```

## Language Switching

- Default: Chinese via `SKILL.md`.
- English companion: [README.en.md](README.en.md) and [SKILL.en.md](SKILL.en.md).
- Runtime request: say "Output in English" or "Use bilingual output."

## Triggers

You can say:

- "Create Taobao main images and a detail page for this product."
- "Give me the main-image storyboard first, not final copy."
- "Turn these selling points into e-commerce visual copy."
- "Create a Douyin Shop visual script."
- "Check whether this detail page has ad-law risks."
- "My designer needs scene direction + on-image copy + design notes."

## Deliverables

| Scenario | Deliverable | Gate |
|---|---|---|
| Visual strategy | Conversion driver, style lock, benefit table, compliance boundary | Confirm direction first |
| Main images | Five image tasks, composition, on-image copy, design notes | Main image copy stays short |
| Detail page | Opening, benefits, material/craft, scenes, proof, FAQ, CTA | Follows a conversion path |
| Image prompts | Subject, composition, lighting, material, background, restrictions | Inherits the Campaign Style Lock |
| Compliance review | Risk terms, replacement direction, disclaimers | No fabricated proof |
| Self review | Compliance, benefit translation, first focus, haptic/media expression, narrative continuity | Any dimension below 80 must be rewritten |

## How It Differs From Generic AI Copy

| Dimension | Generic AI copy | This skill |
|---|---|---|
| Sequence | Jumps to final output | Strategy, storyboard, then execution |
| Output | A block of ad copy | Scene + on-image copy + design notes + prompt |
| Main-image logic | Stacked selling points | Five images with distinct visual tasks |
| Detail-page logic | Information pile | Attention, desire, trust, action |
| Compliance | Often mixes health or absolute claims | Category-specific boundaries |
| Quality gate | Ends when it sounds smooth | Five independent review dimensions |

## Safety Boundaries

This skill will:

- Work from user-provided product information, qualifications, report numbers, and platform requirements.
- Mark uncertain qualifications as missing.
- Use different boundaries for ordinary food, health food, fitness equipment, and other categories.
- Pause at the strategy and storyboard stages before final execution.

This skill will not:

- Replace lawyers, platform reviewers, or regulators.
- Fabricate reports, patents, approval numbers, sales, reviews, or functional proof.
- Promote treatment, healthcare, or body-function improvement claims without qualification.
- Automatically publish, list, message a designer, or edit a store backend.

## Repository Structure

```text
SKILL.md                         # Default Chinese workflow
SKILL.en.md                      # English companion workflow
README.md                        # Chinese homepage, default entry
README.en.md                     # English README
references/compliance-rules.md   # Category compliance rules and replacements
examples/README.md               # Reusable input/output examples
assets/showcase/                 # Showcase images
assets/showcase-output.svg       # Structured output card
tools/verify-skill.py            # Pre-release structure and privacy checks
docs/skill-polishing-report.md   # Luban polishing notes and benchmarks
```

## Verification

Run:

```bash
python tools/verify-skill.py
```

The script checks:

- `SKILL.md` frontmatter, triggers, pauses, and five-part self review.
- `SKILL.en.md` existence and English workflow content.
- README language switching, showcase, safety boundaries, and verification notes.
- Showcase images, examples, compliance rules, and marketplace metadata.
- Common token, cookie, private-path, or credential leaks in text files.

## License

[MIT](LICENSE)

---

<div align="center">

*Validate the visual strategy before chasing conversion.*

</div>
