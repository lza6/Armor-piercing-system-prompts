# Dispatch Patterns & Decision Logic

## PR Type → Skill Mapping

| PR Type | Primary Skill | Secondary | Notes |
|---------|--------------|-----------|-------|
| model | backend-developer | test-engineer | |
| api | api-developer | test-engineer | |
| frontend | frontend-developer | test-engineer | |
| integration | backend-developer | api-developer + test-engineer | |
| tests | test-engineer | — | |
| bugfix | debugger | test-engineer | Start with investigation |
| optimization | performance-profiler | python-optimizer | Add monitoring-specialist for metrics |
| database | supabase-expert | performance-profiler | |
| reporting | data-analyst | excel-reporter | |
| monitoring | monitoring-specialist | performance-profiler | |

## Standard Workflows

### Standard PR Flow
1. `architect` — Design if needed
2. Implementation skill — Based on PR type
3. `test-engineer` — Write tests
4. `reviewer` — Self-review before submission

### Bug Fix Flow
1. `debugger` — Root cause analysis
2. Implementation skill — Fix
3. `test-engineer` — Regression test
4. `reviewer` — Review fix

### Feature Planning Flow
1. `planner` — Opus planning mode → FEATURE_PLAN.md
2. `architect` — Design if complex

### Performance Optimization Flow
1. `performance-profiler` — Identify bottlenecks
2. `python-optimizer` — Optimize code
3. `supabase-expert` — Optimize queries
4. `monitoring-specialist` — Setup monitoring

### Data Analysis Flow
1. `data-analyst` — Statistical analysis
2. `excel-reporter` — Generate reports

## Complexity-Based Routing

| Complexity | Estimated LOC | Skills | Estimated Time |
|-----------|---------------|--------|---------------|
| Simple | < 50 lines, single file | implementation + reviewer | < 30 min |
| Moderate | 50-150 lines, 2-3 files | architect + implementation + test + reviewer | 30-90 min |
| Complex | 150-300 lines, multiple files | planner + architect + implementation + test + reviewer | 90-180 min |

## T0 Selection Logic

1. Check PR type → Select primary skill
2. Assess complexity → Add architect if complex
3. Check domain → Route to specialist
4. Add test-engineer for all implementation PRs
5. Add reviewer for final check

**Overrides**:
- Hotfix → Skip architect, minimal testing
- Refactor → Add extra reviewer focus
- Security → Add security-engineer review

## Common Skill Name Mistakes

| Wrong | Correct |
|-------|---------|
| performance-engineer | `performance-profiler` |
| qa-engineer | `quality-engineer` |
| tester | `test-engineer` |
| developer | `backend-developer` |
| frontend-dev | `frontend-developer` |
| database-expert | `supabase-expert` |
| profiler | `performance-profiler` |
| optimizer | `python-optimizer` |
