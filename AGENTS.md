# Repository workflow

## Sync the parent after submodule pointer automation

After pushing to `main` in `frontend` or `backend`, wait for the pointer automation to update the parent repository's `origin/main`, then switch the parent repository to `main` and fast-forward it with `git pull --ff-only origin main`.

Run this sync only after a push to `main`. Pushes to other branches do not trigger the pointer automation and must not trigger this workflow.
