# .buildkite

A Buildkite fixture for Run Before Queue's fork check (TRUNK-24105). It lives on `rbq-*`
branches only: the `rbq-fork-check` pipeline builds pull requests from `rbq-*` branches and
nothing else, so the PR factory's PRs never get a Buildkite build.

- `pipeline.yml` is uploaded by the pipeline's registered step, which skips fork builds.
- `before-queue.yml` goes through the Dynamic CI plugin. `rbq-target` is the job set to Run
  Before Queue, and `control` always runs.
