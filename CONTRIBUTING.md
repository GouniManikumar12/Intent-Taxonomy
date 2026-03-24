# Contributing to Agentic Intent Taxonomy

Thanks for helping improve the Agentic Intent Taxonomy. This repository is intentionally small: the README carries the normative overview, the schema defines the machine-readable contract, and the examples provide concrete validation targets.

## Getting Started

1. Fork the repository and create a focused branch.
2. Make your changes in the relevant location:
   - `README.md` for normative wording and examples
   - `agentic-intent-taxonomy.schema.json` for machine-readable contract changes
   - `examples/` for canonical fixtures
3. Check that schema, examples, and README stay aligned.
4. Open a pull request that explains the motivation and proposed behavior change.

## Contribution Guidelines

- Keep normative language explicit and implementation-neutral.
- Update examples when you add or change schema behavior.
- Preserve the separation between `model_output` and `system_decision`.
- Call out any compatibility implications for implementers.
- Prefer additive changes in public drafts unless a breaking change is clearly justified.

## Pull Request Checklist

- Schema changes are reflected in examples and documentation.
- New fields or enum values are accompanied by clear semantics.
- Any breaking changes are called out explicitly in the pull request.

## Communication

Use GitHub issues and pull requests for design discussion. For security-sensitive concerns, follow the process in `SECURITY.md`.
