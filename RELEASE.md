# Release Criteria

Before tagging anything public, confirm all items below.

## Spec release criteria

- [ ] Coherent end-to-end example exists
- [ ] Terminology consistent throughout (Claim / Evidence / Falsifier / Confidence)
- [ ] Internal links and cross-references resolve
- [ ] At least one worked example someone else could follow without prior context

## Spec tag checklist

- [ ] Spec has a coherent narrative (intro → concepts → formats)
- [ ] Terminology consistent
- [ ] At least one end-to-end worked example
- [ ] Examples match the spec
- [ ] Version updated (front-matter + any references)
- [ ] Tag message describes what changed

## Tagging

```bash
git tag -a v0.1.0 -m "RAVE Specification v0.1.0 — initial public release"
git push public v0.1.0
```

Then create a GitHub Release from the tag at https://github.com/ravegraph-io/rave-spec/releases/new.
