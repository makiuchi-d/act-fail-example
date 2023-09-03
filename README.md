# act-fail-example

act fails when the git command is used in the workflow job:

```
$ act push -j git
[Build actions/git] 🚀  Start image=ghcr.io/catthehacker/ubuntu:act-latest
[Build actions/git]   🐳  docker pull image=ghcr.io/catthehacker/ubuntu:act-latest platform= username= forcePull=true
[Build actions/git]   🐳  docker create image=ghcr.io/catthehacker/ubuntu:act-latest platform= entrypoint=["tail" "-f" "/dev/null"] cmd=[]
[Build actions/git]   🐳  docker run image=ghcr.io/catthehacker/ubuntu:act-latest platform= entrypoint=["tail" "-f" "/dev/null"] cmd=[]
[Build actions/git] ⭐ Run Main actions/checkout@v3
[Build actions/git]   🐳  docker cp src=/home/makki/Projects/act-fail-example/. dst=/home/makki/Projects/act-fail-example
[Build actions/git]   ✅  Success - Main actions/checkout@v3
[Build actions/git] ⭐ Run Main git show
[Build actions/git]   🐳  docker exec cmd=[bash --noprofile --norc -e -o pipefail /var/run/act/workflow/1] user= workdir=
| fatal: detected dubious ownership in repository at '/home/makki/Projects/act-fail-example'
| To add an exception for this directory, call:
| 
|       git config --global --add safe.directory /home/makki/Projects/act-fail-example
[Build actions/git]   ❌  Failure - Main git show
[Build actions/git] exitcode '128': failure
[Build actions/git] 🏁  Job failed
Error: Job 'git' failed
```

In act, `action/checkout` does not perform an actual repository checkout; instead, it copies (`docker cp`) local files.
During this process, the local UID/GID of the file or directory is preserved.

For security reasons, git does not operate in repositories where the owner is not the same as the user[^1].
In commonly used act images, such as `ghcr.io/catthehacker/ubuntu:act-latest`, jobs are executed with the root user.
However, the repository copied by `action/checkout` is not owned by the root, which prevents the use of the git command.

To address this, we need to configure 'safe.directory'[^2].
In GitHub Actions, git config includes `safe.directory=*`.

[^1]: https://github.blog/2022-04-12-git-security-vulnerability-announced/#cve-2022-24765
[^2]: https://git-scm.com/docs/git-config/2.35.2#Documentation/git-config.txt-safedirectory
