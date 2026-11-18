# .buildkite

A Buildkite fixture for Run Before Queue's fork check (TRUNK-24105). It lives on `rbq-*`
branches only, so the PR factory's PRs never get a Buildkite build.

- `pipeline.yml` is uploaded by the pipeline's registered step.
- `before-queue.yml` goes through the Dynamic CI plugin. `rbq-target` is the job set to Run
  Before Queue, and `control` always runs.

## The pipeline

[`rbq-fork-check`](https://buildkite.com/trunk-technologies-inc/rbq-fork-check) in the
`trunk-technologies-inc` org, on the same cluster as `trunk2-pr-checks` (queue `managed`).
Its settings live in Buildkite, not here; this table is the record of them.

| Setting                                         | Value          | Why                                                                                   |
| ----------------------------------------------- | -------------- | ------------------------------------------------------------------------------------- |
| Registered steps                                | see below      | The only copy of the fork guard a fork cannot edit                                    |
| Build pull requests                             | on             | The check reads each PR's own build                                                   |
| Build branches                                  | on             | Off refuses every API build on a branch, Run Before Queue's own included              |
| Filter builds (conditional)                     | `build.pull_request.id != null \|\| build.branch == "rbq-buildkite"` | No push build of a PR branch: one reads as the same repository without `pull_request.repository` |
| Pull request branch filter                      | `rbq-*`        | Factory PRs and `trunk-merge/*` never build                                           |
| Build pull requests from third-party forks      | on, while testing | The fork PR needs a build of its own                                               |
| Prefix third-party fork branch names            | off            | Off is the case where only `pull_request.repository` tells a fork apart               |
| Skip pull request builds for existing commits   | off            | Every PR head gets its own build                                                      |
| Skip / cancel intermediate builds               | off            | A Run Before Queue build must not be skipped by a later push                          |

Registered steps:

```yaml
steps:
  - label: ":pipeline: upload"
    if: build.pull_request.repository.fork != true
    command: buildkite-agent pipeline upload .buildkite/pipeline.yml
    agents:
      queue: managed
```

The user behind trunk-staging-org's Buildkite token needs Build & Read on the pipeline, and
the token needs `write_builds` and `read_pipelines`.
