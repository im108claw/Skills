---
name: software-adoption-evaluation
description: "Use when evaluating and installing desktop software safely."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [software, installation, compatibility, pricing, verification]
---

# Software Adoption Evaluation

Use this skill when the user asks to research an app, confirm whether it is free, install it, set it up, or make it the default application. The goal is to separate product claims from installability and only execute after both are proven.

## Required sequence

1. **Identify the exact product and intended capability**
   - Distinguish the base app from paid AI, cloud, collaboration, or automation features.
   - Translate “free” into a concrete claim: free download, free browsing, free external-agent connection, free trial, or free built-in service.
2. **Verify current commercial terms from primary sources**
   - Prefer the vendor’s pricing/comparison page, dated announcement, FAQ, and Terms.
   - Record the verification date because plans can change.
   - Treat third-party news as discovery context, then follow it to the vendor source.
   - Never infer that the whole product is free merely because one connector or feature became free.
3. **Verify platform compatibility against the actual target host**
   - Measure OS, version, CPU architecture, and current default application.
   - Check the vendor’s explicit supported-platform matrix and installer artifacts.
   - “ARM supported” is not enough: Windows ARM, macOS ARM, and Linux ARM are different targets.
4. **Build a capability matrix before installation**
   - At minimum: base app, bundled tools, external integration, built-in cloud/AI features, account requirement, subscription requirement, supported OS/architecture.
   - Mark each item as free, paid, unavailable, or unknown.
5. **Apply safety gates**
   - Do not click Subscribe, payment, permission, password, or 2FA UI without explicit authorization.
   - If authentication is intentionally deferred, install only as far as the user requested and stop before sign-in.
   - If the target host is unsupported, do not force-install, emulate, or claim completion. Identify a compatible target instead.
6. **Install only from a vendor-controlled source**
   - Prefer an official package repository or the vendor’s direct download endpoint.
   - Inspect redirect destination, filename, content type, signature/checksum availability, and architecture before execution.
7. **Verify the artifact, not just the installer exit**
   - Confirm the executable launches, version is readable, and expected free capability works without activating a paid plan.
   - For default-app changes, verify the OS default after setup; do not equate “installed” with “made default.”
8. **Report in decision order**
   - Conclusion: installable now or blocked.
   - Free/paid boundary.
   - Compatibility and target host.
   - What was actually installed and verified.
   - Remaining human-only step such as account sign-in.

## Pitfalls

- A newly announced “free” integration may coexist with a paid built-in AI subscription.
- Product FAQs can lag behind newly updated pricing pages. When current first-party pages conflict, prefer the newer dated announcement plus the live comparison table, and explicitly note the stale page.
- Download pages may silently choose an installer using browser user-agent rather than the real host. Inspect the direct endpoint instead of trusting the landing page.
- Do not convert a blocked installation into an indefinite promise without recording the compatible target and the exact unblock condition.
- Host availability is transient. Re-check the chosen remote machine immediately before installation.

## References

- `references/opera-neon-2026-08.md` — dated example of separating a free browser/external-agent tier from a paid built-in AI tier and checking OS/architecture.
