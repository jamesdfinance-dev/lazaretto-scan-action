# Lazaretto Scan (GitHub Action)

Verify npm packages, repos, skills, or files for malicious behavior in CI, before
they ship. It sends each target to the [Lazaretto](https://lazaretto.dev) API,
which fetches it without running it and returns a deterministic verdict
(`malicious`, `flagged`, `clear`, `error`). The action fails the build when the
worst verdict reaches a level you choose.

## Quick start

On every pull request that changes dependencies, scan them, post a sticky comment
with the verdict, and fail the build only if something is malicious:

```yaml
name: Lazaretto scan
on:
  pull_request:
    paths: ['package.json', 'package-lock.json'] # only run when deps change (keeps it cheap)
permissions:
  contents: read
  pull-requests: write # so the action can post its verdict comment
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: jamesdfinance-dev/lazaretto-scan-action@v1
        with:
          api-key: ${{ secrets.LAZARETTO_API_KEY }}
          fail-on: malicious
```

You can also point it at specific targets instead of your package.json:

```yaml
      - uses: jamesdfinance-dev/lazaretto-scan-action@v1
        with:
          api-key: ${{ secrets.LAZARETTO_API_KEY }}
          targets: |
            npm_package:left-pad@1.3.0
            github_repo:some-org/some-skill
```

No key yet? Get one with three free scans, no signup:

```bash
curl -s -X POST https://lazaretto.dev/v1/trial   # returns an api_key
```

Store it as a repo secret (`LAZARETTO_API_KEY`). For more, buy a bundle at
https://lazaretto.dev/#pricing. An agent can also pay per call over x402.

## Pull request comment

On a pull request the action posts one sticky comment (created once, updated on
each run) with a verdict table, so reviewers see the result without opening the
logs. It needs `pull-requests: write` permission (see the quick start). Set
`comment: false` to turn it off.

## Inputs

| input | default | description |
| --- | --- | --- |
| `api-key` | (none) | Lazaretto key with credits. Without it the action reports only and does not fail the build. |
| `targets` | (none) | Newline list of `type:ref`. `type` is `npm_package`, `github_repo`, `clawhub_skill`, or `raw_url`. |
| `package-json` | `package.json` | Scanned for direct deps when `targets` is empty. |
| `fail-on` | `malicious` | Fail the build at this level: `malicious`, `flagged`, or `never`. |
| `comment` | `true` | On a PR, post/update a sticky verdict comment (needs `pull-requests: write`). |
| `github-token` | workflow token | Token used to post the comment. |
| `base-url` | `https://lazaretto.dev` | API base URL. |

## Cost

One full scan uses one credit (a few cents). The action scans each target once
per run, so gate the job on dependency changes (the `paths:` filter in the quick
start) so you pay only when there is something new to check.

Verdicts are signals with evidence, not a warranty. `clear` means no known-bad
match and no rule fired. It is not a statement that the package is risk-free. The
full evidence for each verdict is in the report from the API.

## License

MIT. The Lazaretto service and its detection engine are separate and proprietary.
