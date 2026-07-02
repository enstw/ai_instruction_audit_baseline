# Skill: verify

Verify that a code change actually does what it's supposed to by exercising it end-to-end and observing behavior — drive the affected flow, not just tests or typecheck. Run before committing nontrivial changes.

## Do NOT invoke
- On a diff that only touches tests, docs, or other code with no runtime surface to drive (a change to product source always has one) — there's nothing to observe
