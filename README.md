# GrowthBook Code References with GitHub Actions (standalone utility)

This GitHub Action is a utility that can be used for finding references to feature flags in your code, both for reference and for code cleanup.

## Configuration

Create a new Actions workflow in your selected GitHub repository (e.g. `code-references.yml`) in the `.github/workflows` directory of your repository. Under "Edit new file", paste the following code:

```yaml
name: Find feature flag code references
on:
    pull_request:
        branches:
            - main
concurrency:
    group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
    cancel-in-progress: true
jobs:
    growthBookCodeReferences:
        name: GrowthBook Code References
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v4
              with:
                  # This value must be set if the lookback configuration option is
                  # defined for find-code-refs. Read more:
                  # https://github.com/growthbook/gb-find-code-refs#searching-for-unused-flags-extinctions
                  fetch-depth: 11
            - name: Fetch flags
              run: |
                  curl -H "Authorization: Bearer ${{ secrets.GB_API_TOKEN }}" "${{ secrets.GB_API_HOST }}/api/v1/feature-keys" -o flags.json
            - name: GrowthBook Code References
              uses: growthbook/coderefs-standalone-action@2.11.5-13
              with:
                  flagsPath: flags.json
                  dir: ${{ github.workspace }}
                  # head_ref only available on pull request eventss, ref_name for push events
                  branch: ${{ github.head_ref || github.ref_name }}
                  repoName: ${{ github.repository }}
            - name: Upload coderefs.json
              run: |
                  if test -f "coderefs.json"; then
                    curl -XPOST -H "Authorization: Bearer ${{ secrets.GB_API_TOKEN }}" -H "Content-Type: application/json" -d @coderefs.json "${{ secrets.GB_API_HOST }}/api/v1/code-refs"
                  else
                    echo "No coderefs.json file found."
                  fi
```

<!-- action-docs-inputs -->

## Inputs

| parameter    | description                                                                                                                                                                                                                                                                                                                                                                                                                     | required | default         |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | --------------- |
| flagsPath    | Path to JSON file containing an array of feature key strings.                                                                                                                                                                                                                                                                                                                                                                   | `true`   |                 |
| dir          | Path to directory containing codebase to be scanned.                                                                                                                                                                                                                                                                                                                                                                            | `true`   |                 |
| branch       | Name of the branch.                                                                                                                                                                                                                                                                                                                                                                                                             | `true`   |                 |
| outFile      | Set the filename of the output JSON containing all code references. Defaults to 'coderefs.json'.                                                                                                                                                                                                                                                                                                                                | `false`  | 'coderefs.json' |
| repoName     | Define the repo name so that it is included in the output JSON.                                                                                                                                                                                                                                                                                                                                                                 | `false`  |                 |
| allowTags    | Enable storing references for tags. Lists the tag as a branch.                                                                                                                                                                                                                                                                                                                                                                  | `false`  | false           |
| contextLines | The number of context lines above and below a code reference for the job to send to GrowthBook. By default, the flag finder will not send any context lines to GrowthBook. If < 0, it will send no source code to GrowthBook. If 0, it will send only the lines containing flag references. If > 0, it will send that number of context lines above and below the flag reference. You may provide a maximum of 5 context lines. | `false`  | 2               |
| debug        | Enable verbose debug logging.                                                                                                                                                                                                                                                                                                                                                                                                   | `false`  | false           |
| lookback     | Set the number of commits to search in history for whether you removed a feature flag from code. You may set to 0 to disable this feature. Setting this option to a high value will increase search time.                                                                                                                                                                                                                       | `false`  | 10              |

<!-- action-docs-inputs -->
