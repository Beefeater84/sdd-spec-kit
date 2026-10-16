# Staging is the one Vercel preview with a fixed URL

Status: accepted
Date: 2026-10-16
Issue: PR #54
Affects: -; README, preprod setup on Vercel

## Context

The kit targets projects deployed to Vercel. Vercel builds a preview for every branch, but each feature branch gets its own URL, which appears with the branch and goes away with it. Many things can only be checked on a real site with a known address: payments, sign-in (OAuth redirect URLs, cookie domains), webhooks, and testing from a phone. These need the address registered in third-party services and the environment variables set for it in Vercel. Doing that again for every feature branch does not work.

## Decision

- The development branch is the permanent branch `staging` (see `45-stable-release-branch.md`). On Vercel it is the one preview branch with a fixed URL that never changes.
- That URL is set up once: environment variables in Vercel for the `staging` preview, and the address registered where a confirmed address is required (payment providers, OAuth apps, webhooks).
- Features that need a real address (payments, sign-in, anything checked from a phone) are checked on the `staging` URL after their PR is merged into `staging`, before the release PR into `main`.

## Consequences

- The name `staging` and its URL stay fixed; renaming the branch means redoing the Vercel and third-party setup.
- Feature-branch previews stay useful for UI checks, but are not used for checks that need a confirmed address.
- `main` stays the production branch; only released code reaches it.
