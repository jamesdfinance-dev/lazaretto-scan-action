# Lazaretto Scan (GitHub Action)

Verify npm packages, repos, skills, or files for malicious behavior in CI, before
they ship. It sends each target to the [Lazaretto](https://lazaretto.dev) API,
which fetches it without running it and returns a deterministic verdict
(`malicious`, `flagged`, `clear`, `error`). The action fails the build when the
worst verdict reaches a level you choose.

## Quick start

```yaml
- uses: jamesdfinance-dev/lazaretto-scan-action@v1
  with:
    api-key: ${{ secrets.LAZARETTO_API_KEY }}
    targets: |
      npm_package:left-pad@1.3.0
      github_repo:some-org/some-skill
    fail-on: malicious
```

No key yet? Get one with three free scans, no signup:

```bash
curl -s -X POST https://lazaretto.dev/v1/trial   # returns an api_key
```

Store it as a repo secret (`LAZARETTO_API_KEY`). For more, buy a bundle at
https://lazaretto.dev/#pricing. An agent can also pay per call over x402.

## Scan your dependencies

Leave `targets` empty and it scans the direct dependencies in `package.json`:

```yaml
- uses: jamesdfinance-dev/lazaretto-scan-action@v1
  with:
    api-key: ${{ secrets.LAZARETTO_API_KEY }}
    fail-on: flagged
```

## Inputs

| input | default | description |
| --- | --- | --- |
| `api-key` | (none) | Lazaretto key with credits. Without it the action reports only and does not fail the build. |
| `targets` | (none) | Newline list of `type:ref`. `type` is `npm_package`, `github_repo`, `clawhub_skill`, or `raw_url`. |
| `package-json` | `package.json` | Scanned for direct deps when `targets` is empty. |
| `fail-on` | `malicious` | Fail the build at this level: `malicious`, `flagged`, or `never`. |
| `base-url` | `https://lazaretto.dev` | API base URL. |

## Cost

One full scan uses one credit (a few cents). The action scans each target once
per run, so scanning a large dependency tree on every push adds up. Start with an
explicit `targets` list or `fail-on: flagged` on the packages you care about.

Verdicts are signals with evidence, not a warranty. `clear` means no known-bad
match and no rule fired. It is not a statement that the package is risk-free. The
full evidence for each verdict is in the report from the API.

## License

MIT. The Lazaretto service and its detection engine are separate and proprietary.
