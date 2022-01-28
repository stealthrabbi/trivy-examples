# trivy-examples

Examples of tinkering with Trivy vulnerability scanner

Always consult the [latest Trivy documentation](https://aquasecurity.github.io/trivy/). These notes are meant to be a
general guide, but may be out of date with the latest Trivy. At the time of writing, 0.22.0 was the latest.

Trivy can be used to scan docker images (either your own, or public ones) on your local system. It can be run either as a docker image, or installed natively on an O/S.

Trivy can be integrated in to your Continuous Integration (CI) process. THis is advantageous because vulnerabilities can be
determiend prior to pushing a release package to an artifact repository.

# Setup

## Option 1 - Use trivy docker image

1. [Install Docker](https://docs.docker.com/get-docker/) for your O/S.
1. Pull the latest trivy docker image
   ```
   docker pull aquasec/trivy:latest
   ```

## Option 2 - Install natively on O/S

1. [Install trivy](https://aquasecurity.github.io/trivy/v0.22.0/installation/) for your O/S. Recommed using the Install script.

# Scanning docker images via trivy

Note, these are being run from windows, with the trivy cache being mounted to `C:\temp-trivy`. Change this to whatever directory you want to use to cache the trivy database.

These examples show running trivy both as a docker container (option 1), and natively (option 2).

Various [commmand line interface options](https://aquasecurity.github.io/trivy/v0.22.0/getting-started/cli/image/) exist, such as specifying the exit code to use if there are findings.

## hello-world example - should reveal no errors

```
docker run --rm -v C:\temp-trivy:/root/.cache/ aquasec/trivy:latest image hello-world
```

```
trivy image hello-world
```

## Node Alpine - 3 findings

```
docker run --rm -v C:\temp-trivy:/root/.cache/ aquasec/trivy:latest image node:14-alpine
```

```
trivy image hello-world node:14-alpine
```

# Scanning configuration / Infrastructre as Code (IaC) files

With docker image, the folder for scanning must be mounted as a path inside the container. This example uses the `./configs/` directory in this repository.

```
docker run --rm -v $PWD/configs\:/root/configs/ -v C:\temp-trivy:/root/.cache/ aquasec/trivy:latest fs --security-checks vuln,config /root/configs
```

```
trivy fs --security-checks vuln,config ./configs
```

# Integration with CI

See Advanced configuration for all examples.

- [GitLab CI Integration](https://aquasecurity.github.io/trivy/v0.22.0/advanced/integrations/gitlab-ci/)

# Vulnerability filtering

Sometimes there is no reasonable solution to a vulnerability. A vulnerability can be assessed and suppressed if deemed acceptable by the community.

THe CLI allows specifying a [trivyignore file](https://aquasecurity.github.io/trivy/v0.22.0/vulnerability/examples/filter/) (with a default location).

```
docker run --rm -v $PWD/.trivyignore:/.trivyignore -v $PWD/configs:/root/configs/ -v C:\temp-trivy:/root/.cache/ aquasec/trivy:latest fs --security-checks vuln,config /root/configs
```

for non-docker usage, the .trivyignore file is already in the correct location if running trivy from the root directory. Try editing the file to filter / unfilter things.

# Usage notes

If scanning your built docker image, you may come across vulnerabilities that do not seem to be
caused by your Dockerfile / software. Trivy tries to separate out vulnerabilities from the base image
and contents added by your image. However, depending on the base image, it may be hard to distinguish that. FOr example,
`alpine-node` contains vulnerable node packages, but these show up in the node packages section of the vuln report, not in the
base image report.

So, consider scanning the base image of your custom containers, in addition to the final image.
