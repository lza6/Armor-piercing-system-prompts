# Roadmap

This roadmap keeps the project direction clear for users, contributors, and maintainers. The goal is to make `ecommerce-visual-copywriting-skill` a practical, reusable AI agent skill for e-commerce visual copywriting, compliance review, and product-detail-page planning.

## Current Scope

The project currently provides:

- A 6-step workflow for e-commerce main image and product detail page copywriting.
- A structured output format: visual scene, in-image copy, and design guidance.
- A compliance rule library for common product categories.
- A self-review scoring mechanism that requires outputs to reach a minimum quality bar before delivery.
- Installation and usage guidance for AI tools that support custom skills or prompt instructions.

## Near-Term Priorities

### v1.1: Examples and Evaluation Cases

- Add more input/output examples for common product categories.
- Build a small regression case set for checking whether future rule changes still produce compliant output.
- Add bad-example vs improved-example comparisons.
- Document expected output standards for designers, operators, and AI agent users.

### v1.2: Compliance Rule Expansion

- Expand platform-specific review notes for Taobao, Tmall, JD, Pinduoduo, and Douyin Shop.
- Add more category-specific rule sections, especially for food, supplements, fitness products, personal care, and household goods.
- Improve the banned-word replacement table with safer expression patterns.
- Add source notes where compliance rules depend on public platform policies or legal requirements.

### v1.3: Contributor Workflow

- Add issue templates for bug reports, compliance-rule suggestions, and example requests.
- Add pull request templates with a review checklist.
- Create a repeatable review process for changes to compliance rules.
- Publish release notes for meaningful changes to the skill workflow or rule library.

### v1.4: Quality and Automation

- Add lightweight validation scripts for file structure, required sections, and broken links.
- Add sample prompts for testing skill behavior across different AI tools.
- Explore automated checks for overlong in-image copy, missing disclaimers, and risky claims.
- Maintain a small changelog of compliance rule updates.

## Ongoing Maintenance

The project will continue to prioritize:

- Practical usefulness for real e-commerce operators and designers.
- Clear boundaries between copywriting suggestions and legal/compliance advice.
- Verifiable product claims instead of unsupported marketing claims.
- Concise, mobile-readable copy that can realistically fit into product images.
- Transparent review of issues and pull requests from the community.

## How AI Assistance Is Used

AI tools may be used to help maintain this repository, especially for:

- Triage of issues and pull requests.
- Drafting and reviewing documentation improvements.
- Generating example cases for manual review.
- Testing whether rule changes create risky or non-compliant outputs.
- Preparing changelogs and release notes.

All substantive rule changes should still be reviewed by a human maintainer before release.

## Contribution Areas

Good first contributions include:

- Adding real-world example inputs and expected outputs.
- Reporting unclear wording in the skill workflow.
- Suggesting safer replacement expressions for risky advertising claims.
- Adding platform-specific review notes with sources.
- Improving English descriptions so non-Chinese users can understand the project.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for contribution guidelines.
