# FAQ

## Why does ASG exist?

AI coding agents burn tokens on things that have nothing to do with the task:
re-sending the same giant prompt every turn, re-sending cold history that the
model has already answered, generating unconstrained output that nobody reads,
and retrying failed tool calls. ASG is a **server-side gateway** that removes
that waste before it is billed — without changing how your client speaks.

## What is ASG exactly?

A reverse-proxy gateway that sits between your AI client and the model provider
(a proxy you point `base_url` at). It rewrites requests and streams in-flight to
maximize the provider's prefix cache, fold cold context, constrain outputs, and
repair broken tool calls. Your client sees a normal OpenAI-compatible endpoint.

## Is ASG open source?

No. The source is **closed and proprietary**. This repository is the public
documentation and benchmark record for the design, with no implementation code.
See [LICENSE](../LICENSE).

## How does ASG compare to rtk?

Both reduce token waste, but at opposite layers:

- **rtk** (Rust Token Killer) is a *client-side* CLI that filters a coding
  agent's command output *before* it enters the context window.
- **ASG** is a *server-side* gateway that rewrites the request/response traffic
  *between* the client and the provider.

They are complementary and could be combined: rtk keeps noise out of the prompt;
ASG keeps the prompt maximally cacheable and folds what is already answered.

## Which models / providers are supported?

Any provider behind an OpenAI-compatible chat-completions API, plus
Anthropic-style passthrough for native caching. Verified in production against
DeepSeek, GLM, Qwen, MiniMax, OpenAI, Anthropic, and xAI.

## What do I have to change in my client?

One thing: the `base_url`. Point your client at the ASG endpoint (e.g.
`https://asg.example.com/v1`) and keep the same model names and keys. See
[Quickstart](getting-started/quickstart.md).

## How are savings measured?

`cache hit rate = cached_tokens / (prompt_tokens + cached_tokens)`, and cost is
computed from actual token flows at provider pricing. Details in
[methodology](benchmarks/methodology.md). The headline number is a **cost**
reduction, not a token reduction.

## Is the 86% figure typical?

It's what one production deployment observed. Yours depends on session length,
model mix, and how noisy your clients are. Long sessions with stable prompts
see the biggest wins (per-session hit rates 75–94%).

## Can I try ASG?

Not in public preview at this time. This repository documents the design so you
can evaluate the approach; contact the maintainer for access.

## How is this different from prompt caching?

Prefix caching is the *lever*; ASG is what makes the lever work. Providers only
cache byte-identical prefixes — real client traffic is never byte-identical from
turn to turn. ASG normalizes requests so that the stable prefix is actually
stable, then injects `cache_control` to pin it and warms it asynchronously.

## Does ASG modify model outputs?

Yes, in a constrained way: the Dual-Mode output constraint steers the model
toward shorter, structured output when the request indicates the full verbosity
is not needed. It does not change semantics for requests where full output is
required.

## Is it safe?

The rewrite layers are opt-in, shadow-capable, and fail-open: if a rewrite can't
be applied confidently, traffic passes through untouched. The dangerous cases
(empty tool calls, orphan tool messages) are exactly the ones being repaired.
