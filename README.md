# snyk-bulk

`Dockerfile` entrypoints that replicate `snyk monitor —all-projects` on a per language basis.

__Useful for:__
* flexible testing of monorepos
* discovery-oriented vulnerability scanning
  * contents are not known ahead of time
  * scanning "out-of-band", not directly in the developer build pipeline

## Examples - future plans
build your snyk-bulk image for python3 scanning

`docker build -t snyk-bulk:python3 -f Dockerfile-python .`

snyk scan all python3 projects

`docker run -it --rm --env SNYK_TOKEN -v $(PWD):/project snyk-bulk:python3`

## Current in progress work:

These entrypoints now take commandline arguments that will allow for flexible usage, currently it supports four arguments: 
```
--remote-repo --> the repository name to group all these projects under in the snyk UI (required)
--target --> the folder / local git repo to scan (required)
--json --> folder where the json files should be stored, if not specified, they will be thrown into a mktemp directory and thrown out, with just a summary of pass / fail printed to stdout
--debug --> enables set -x on the entry point bash runtime, and --debug on snyk, shows you as much info as you could possible get
```

An example command for a docker container built with the python file above would look like this:
`docker run -it -e SNYK_TOKEN -v $(pwd):/home/dev mrzarquon/snyk-bulk:python --remote-repo "https://bitbucket.org/cmbarker/myproject" --target "/root/testrepo/"`

Adding `--json /home/dev/json_output` would have the entrypoint save the json to a folder outside of the container, etc. There is a test repository at `/root/testrepo` for an easy purge of cached packages / lockfiles while testing / developing entry points .

## Ecosystem manifest coverage

ecosystem  | manifests           | default base image    | starter Dockerfile |
---------- | ------------------- | --------------------- | ------------------ |
javascript | package-(lock).json<br/>yarn.lock | node:lts-buster-slim  | Dockerfile-javascript |
python     | requirements.txt<br/>Pipfile(.lock)<br/>poetry.lock<br/>setup.py | python:slim-buster | Dockerfile-python |

## Testrepo Content

Testrepo itself is setup as a subtree (not a submodule), tl,dr; subtree lets us just copy files & git history from a repo instead of trying to create a permanent link in time to it. It requires more work on the developer who is updating the subtree folder (testrepo) in the repo, but if you're not modifying testrepo, you can ignore it entirely.

Current testrepo source is: `https://github.com/mrzarquon/nightmare`

How to pull in new testrepo content from the upstream repo:
```
tl,dr; version:
git subtree pull -P testrepo git@github.com:mrzarquon/nightmare.git main
```

1) Add a subtree remote to this repo on your workstation
```
git remote add -f testrepo git@github.com:mrzarquon/nightmare.git
```

2) Pull in the latest from upstream:
```
git subtree pull -P testrepo testrepo main
```

