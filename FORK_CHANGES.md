# Fork changes

`Superior410/universal-modder` is a fork of [`rehan-remade/universal-modder`](https://github.com/rehan-remade/universal-modder),
used as a **pinned dependency** of Universal Mashup Studio (`Superior410/universal-mashup-studio`, ADR 0001).
It is kept as close to upstream as possible so upstream changes can always be pulled.

## Fork-specific changes

| Change | Why | Upstreamable? |
|---|---|---|
| This file | records every fork-only change (brief §137) | no |

No toolkit code differs from upstream. Studio-specific code and documents live in the Studio repository.
Improvements Studio needs from the toolkit (e.g. `um publish check --json`, Steam `buildid` in `um scan`)
are proposed upstream as pull requests rather than patched here.

## History

- 2026-10-07: Epic 0 documents (brief, build order, repository audit, cloud isolation plan, ADRs 0001-0004)
  were drafted here in PRs #25-#30, then moved to the Studio repository.
