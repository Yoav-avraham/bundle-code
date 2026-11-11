# bundle-code

Copy a project folder to a destination, skipping the paths you list in an ignore file, and
stamp it with a `commit_metadata.json` describing the git commit and GitHub Actions run it
came from. Optionally zip the result.

Pure standard library, no dependencies. Python 3.8+.

## Install

    pip install bundle-code
    pip install git+<repo-url>      # or straight from git

## Usage

    bundle-code SRC DST [--ignore IGNORE_FILE] [--metadata-file NAME]
                [--pack] [--pack-name NAME]

Also runnable as `python -m bundle_code`.

| Argument | Description |
| --- | --- |
| `SRC` | Folder to copy. |
| `DST` | Destination folder (created if missing; merged into if it exists). |
| `--ignore FILE` | Optional ignore file (see below). |
| `--metadata-file NAME` | Metadata file name inside `DST`. Default: `commit_metadata.json`. |
| `--pack` | Also zip `DST` into `<DST>.zip`. |
| `--pack-name NAME` | Zip name without `.zip`. Defaults to the destination folder name. |

### Example

    bundle-code ./my-app ./dist/my-app --ignore .bundleignore --pack

## Ignore file

One entry per line. Blank lines and lines starting with `#` are skipped. A trailing `/` is
ignored. An entry matches a path if it equals the file or folder name, equals the path
relative to `SRC` (for example `docs/drafts`), or matches it as a glob (for example `*.pyc`).

    # .bundleignore
    node_modules
    __pycache__
    *.log
    docs/drafts

`.git` is always skipped. A missing ignore file only prints a warning.

## Commit metadata

`DST/commit_metadata.json` is written after the copy:

```json
{
  "commit": { "sha": "...", "short_sha": "...", "author_name": "...", "author_email": "...",
              "commit_date": "...", "subject": "...", "message": "...",
              "pr_head_sha": null, "url": "https://github.com/org/repo/commit/..." },
  "ref": { "ref": "refs/heads/main", "ref_name": "main", "ref_type": "branch" },
  "repository": "org/repo",
  "actor": "...",
  "event_name": "push",
  "workflow": "...",
  "run": { "id": "...", "number": "...", "attempt": "...", "url": "..." },
  "generated_at": "2025-01-01T00:00:00+00:00"
}
```

- Commit details come from `git log -1` in `SRC`. If `SRC` is not a git repo, `sha` falls back
  to `GITHUB_SHA`.
- Run, ref, repository and actor come from the standard `GITHUB_*` environment variables, so
  they are `null` outside GitHub Actions.
- On `pull_request` events, `pr_head_sha` is read from the file at `GITHUB_EVENT_PATH`.

## GitHub Actions example

```yaml
- uses: actions/checkout@v4
- run: pip install bundle-code
- run: bundle-code . build/app --ignore .bundleignore --pack --pack-name app-${{ github.sha }}
```

Pairs well with [bundle-deps](../bundle-deps), which bundles the dependencies for offline use.
