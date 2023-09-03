# act-fail-example

act fails when the git command is used in the workflow job:

```console
$  act push -W .github/workflows/git.yaml -j git-fail
[Git command/git-fail] 🚀  Start image=ghcr.io/catthehacker/ubuntu:act-latest
[Git command/git-fail]   🐳  docker pull image=ghcr.io/catthehacker/ubuntu:act-latest platform= username= forcePull=true
[Git command/git-fail]   🐳  docker create image=ghcr.io/catthehacker/ubuntu:act-latest platform= entrypoint=["tail" "-f" "/dev/null"] cmd=[]
[Git command/git-fail]   🐳  docker run image=ghcr.io/catthehacker/ubuntu:act-latest platform= entrypoint=["tail" "-f" "/dev/null"] cmd=[]
[Git command/git-fail] ⭐ Run Main actions/checkout@v3
[Git command/git-fail]   🐳  docker cp src=/home/makki/Projects/act-fail-example/. dst=/home/makki/Projects/act-fail-example
[Git command/git-fail]   ✅  Success - Main actions/checkout@v3
[Git command/git-fail] ⭐ Run Main git log -n1
[Git command/git-fail]   🐳  docker exec cmd=[bash --noprofile --norc -e -o pipefail /var/run/act/workflow/1] user= workdir=
| fatal: detected dubious ownership in repository at '/home/makki/Projects/act-fail-example'
| To add an exception for this directory, call:
| 
|       git config --global --add safe.directory /home/makki/Projects/act-fail-example
[Git command/git-fail]   ❌  Failure - Main git log -n1
[Git command/git-fail] exitcode '128': failure
[Git command/git-fail] 🏁  Job failed
Error: Job 'git-fail' failed
```

In act, `action/checkout` does not perform an actual repository checkout; instead, it copies (`docker cp`) local files.
During this process, the local UID/GID of the file or directory is preserved.

For security reasons, git does not operate in repositories where the owner is not the same as the user[^1].
In commonly used act images, such as `ghcr.io/catthehacker/ubuntu:act-latest`, jobs are executed with the root user.
However, the repository copied by `action/checkout` is not owned by the root, which prevents the use of the git command.

To address this, we need to configure `safe.directory`[^2].
In GitHub Actions, git config includes `safe.directory=*`.

## Version info

```console
$ act --version
act version 0.2.50
$ docker images ghcr.io/catthehacker/ubuntu:act-latest 
REPOSITORY                    TAG          IMAGE ID       CREATED      SIZE
ghcr.io/catthehacker/ubuntu   act-latest   084dbe12375b   2 days ago   1.23GB
```

[^1]: https://github.blog/2022-04-12-git-security-vulnerability-announced/#cve-2022-24765
[^2]: https://git-scm.com/docs/git-config/2.35.2#Documentation/git-config.txt-safedirectory
