# Risks
Next ID: R5 | Last scan: 2026-04-01 | Baseline: 0 critical, 0 high, 0 medium, 2 low

## Critical

## High

## Medium

## Low
#R3 [LOW] /vibe:add copies to .vibe/docs/ without scanning for sensitive content. (found 2026-04-01)
#R4 [LOW] [ACCEPTED] init.md language detection limited to maxdepth 3. Deep nested files may be missed. (found 2026-04-01)

## Resolved
#R1 [MEDIUM] Fixed: .vibe/ atomic write concern is inherent to all file I/O, not plugin-specific. Accepted. 2026-04-01
#R2 [MEDIUM] Fixed: playwright detection standardized to (test -f X -o -f Y) across all commands. 2026-04-01
