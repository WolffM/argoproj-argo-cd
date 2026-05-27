## Steps to reproduce
1. From the repository root, run `go test ./util/git -run Test_nativeGitClient_Checkout_SubmoduleDisabledStillCleansState -count=1`.
2. The test creates a repository with a `.gitmodules` file, checks out a revision with submodules disabled, writes an untracked `stale.txt`, and checks out the same revision again with `cleanState=false`.
3. This mirrors reposerver reuse of a local clone across multiple revision operations when submodules are disabled.

## Observed
Before the fix, the second checkout did not run `git clean -ffdx` when `submoduleEnabled=false` and `cleanState=false`. The untracked `stale.txt` remained in the repo working tree, demonstrating state leakage between operations. In real reposerver usage, this stale state can pollute later render operations and cause inconsistent or unknown application status outcomes.

## Expected
Repositories that contain `.gitmodules` should still be cleaned between checkout operations, even when submodules are disabled. Running checkout repeatedly across revisions should not preserve stale untracked files or nested git artifacts. The second checkout should remove `stale.txt` and leave a deterministic working tree for the requested revision.
