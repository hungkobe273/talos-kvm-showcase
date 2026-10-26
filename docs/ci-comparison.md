# Three CI systems on one app

The same Go demo app is built by three systems from the same commit.

| | GitHub Actions | Jenkins | GitLab CE |
|---|---|---|---|
| Hosting | GitHub-hosted runners | Self-hosted, Docker on a Compute Engine VM | Self-hosted Omnibus on a VM, Docker-executor runner on the Jenkins VM |
| Pipeline file | workflow YAML | `Jenkinsfile` (Groovy, sandboxed) | `.gitlab-ci.yml` |
| Trigger | push | SCM polling every 5 min | push (repo mirrored to GitLab) |
| Output | image `sha-*`, bumps the GitOps repo | image `jenkins-sha-*` | image `gitlab-sha-*` |
| Deploys | yes (only deployer) | no | no |

## Notes from setting them up

- Jenkins: keep the Groovy sandbox on; private repos need a read-only credential; Go VCS stamping fails on
  mounted workspaces with different ownership (`GOFLAGS=-buildvcs=false`).
- GitLab: disable Auto DevOps before the first push; `image` is a reserved keyword and cannot be a job name;
  watch for trailing `\r` when pasting tokens from Windows.
- Only one system should write to the GitOps repo, otherwise two pipelines race to bump the same tag.
