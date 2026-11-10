# bundle-code

Copy a folder to a destination, skipping paths listed in an ignore file, and write
`commit_metadata.json` (git commit and GitHub Actions info).

## Install

    pip install bundle-code

## Usage

    bundle-code SRC DST [IGNORE_FILE] [--metadata-file NAME]

`.git` is always skipped. Also runnable as `python -m bundle_code`.
# bundle-code
