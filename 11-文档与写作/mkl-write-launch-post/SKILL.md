---
name: "mkl-write-launch-post"
description: "Draft factual project announcements, GitHub launch posts, and development updates for a specified audience or platform. Use to explain shipped work clearly, with evidence and a useful invitation for feedback."
---

# Draft a project announcement

Establish the audience, platform, and state of the work: available release, merged improvement, open PR, or plan. Inspect the supplied revision, demo, and project links before claiming availability. When the medium is unspecified, write a short platform-neutral draft that the user can adapt.

Start with a concrete problem and what the project now lets someone do. Use a small example or observed result when available. Include the minimum context needed to understand a limitation, setup requirement, or preview status. Keep an open PR or planned capability explicitly separate from shipped behavior.

Use the author's supplied tone or samples without inventing personal stories. Numbers about adoption, performance, time saved, stars, or customers need evidence. An aspirational sentence must remain an aspiration. Do not fabricate testimonials, claim affiliations, or frame a static format check as a successful live-agent test.

For a stated length limit, measure the draft with the platform's actual counting rules when available; otherwise state the counting assumption. Preserve the destination URL. Prefer one relevant invitation, such as trying the example or reporting a specific integration issue, over several competing calls to action.

Return a ready-to-review draft. List unresolved factual questions separately, without placeholders disguised as completed claims. Publishing or sending it is a separate action: use existing authorization if present; otherwise leave the draft for review.

## Worked example

Evidence: a merged change adds `--dry-run`; Linux was tested; Windows was not. No time-saving measurements exist.

One suitable draft: "The installer now has a preview mode. Run with `--dry-run` to inspect planned changes before writing files. We tested it on Linux; Windows still needs a check. If you try it on Windows, an issue with the steps and output would help."

Acceptance: availability matches the merged change, limits remain visible, and no adoption or productivity claim is invented. This is an authored example, not a recorded client evaluation.
