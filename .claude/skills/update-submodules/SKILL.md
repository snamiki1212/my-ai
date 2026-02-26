---
name: update-submodules
description: Update all git submodules under the submodules/ directory by checking out master and pulling the latest changes. Use this skill whenever the user says "/submodule-update", "update submodules", "pull submodules", "sync submodules", or wants to get the latest changes from submodule repositories. Always trigger this skill for any submodule update/pull/sync request.
---

# Submodule Update

Pull the latest changes from the master branch for all git submodules under the `submodules/` directory.

## Steps

1. Find all submodules under `submodules/` directory using `git submodule status` or by listing the `submodules/` directory.

2. For each submodule, run the following commands:
   ```bash
   cd <submodule-path>
   git checkout master
   git pull origin master
   ```

3. Report the result for each submodule: success or failure with error details.

## Example Execution

```bash
# Get list of submodules
git submodule status

# Update each submodule
cd submodules/bluage && git checkout master && git pull origin master
```

## Reporting

After updating, summarize the results in a clear table or list:

```
Submodule Update Results:
- submodules/bluage: ✓ Updated to <commit-hash> (or "Already up to date.")
- submodules/foo:    ✗ Failed — <error message>
```

If any submodule fails, clearly report the error so the user can investigate.

## Notes

- If a submodule uses a branch other than `master` (e.g., `main`), try `main` as a fallback if `master` fails.
- Run updates sequentially to avoid confusing output.
- Always run from the repository root (the directory containing `submodules/`).
