# DevOps-Demo

This is a Jenkins server setup for running Jenkins on a local machine. It will run a pipeline that executes a Python script.

## Demo production server

> **Warning**
> The demo server has been shut down, as it was costing me money and resources and the demo period was over. It can be re-established on demand if needed.

For demonstration purposes, this Jenkins server is working and available at [jenkins.zero-ml.com](https://jenkins.zero-ml.com). Contact me to get a username and password if you wish to inspect it.

## Requirements

1. Tested this only on Linux, in a x86_64 architecture. You should be able to run it on other architectures, but I can't guarantee it.
2. You need to have Docker & Docker Compose installed on your machine.
3. Clone this repository to your machine and `cd` into it.

## Local Setup

To setup this server locally, accessible at [localhost:8080](http://localhost:8080), run the following commands:

```bash
docker-compose --env-file=example.env up -d
```

After that, you can view the Python build pipeline by accessing [localhost:8080/job/example-app-python](http://localhost:8080/job/example-app-python) (Login details are in [example.env](example.env)). No need to click anything, the pipeline is pre-configured to scan the repository every 3 minutes, just sit back and watch.

## Security

- The local server does not use TLS/SSL certificates(The production server does). The local server is accessible through HTTP on port 8080 (The production server is accessible through HTTPS on port 443).
- The server is pre-configured with matrix-based security. You can login with the credentials in [example.env](example.env).
- The server image is built from the official Jenkins image, but provides only a frozen subset of plugins. This is to prevent the server from being compromised by malicious plugins, or compromised plugin updates. See the [master's Dockerfile](master/Dockerfile).
- The host does not expose the docker socket to the Jenkins master container. Instead, this is done via `socat`. See the [docker-compose.yml](master/docker-compose.yml) file for more details. Ideally, the Jenkins master container should not have access to the docker socket at all, but this is not possible in a setting where running on local machines is needed.
- The Jenkins server does not run its own builds as a node. Instead, it provisions temporary docker containers as nodes, as requested by the pipeline. This is much safer and does not expose the server to arbitrary malicious code in the builds.
- The configured job runs on every branch, including pull requests from forks. However, the job explicitly doesn't trust code from forks, which will make them run in groovy sandbox mode. This is to prevent malicious code from being executed on the server. See [here](master/init.groovy.d/jobs.groovy#L23) for more details.

## Maintainability

- The Jenkins server is built from a Dockerfile, with hardly any changes to the original, other than preloading it with plugins. This makes it very easy to maintain and update the server, after testing the updates in a staging environment.
- Likewise, the plugins themselves are all in the minimal set of plugins that come with the typical installation of Jenkins. The only exception to this is the Docker plugin.
- The server is easily customized through various `groovy` scripts that run on every server restart. This can be used to easily insert new jobs, security settings, users and more. See the [master's init.groovy.d](master/init.groovy.d) folder for more details.
- It's easy to configure the server as a proper production server, see the production branch in this repository to see how I do it in [jenkins.zero-ml.com](http://jenkins.zero-ml.com).
- Since the server is deployed via a single `docker-compose` command, it's easy to deploy it on any machine, including production servers. Moreover, it also means that any server failures are easily recoverable, as the server is almost stateless. The only data loss that can occur is the loss of the pipeline build history, which is not a big deal (and if it is, the data volume can easily be backed up to an external location).

## Storage

| Image | Description | Size |
| --- | --- | --- |
| ofersadan85/jenkins-docker-master | Jenkins master image, alpine based with plugins installed | 541 MB |
| ofersadan85/jenkins-agent-python-docker | Python agent image, alpine based with python installed | 461 MB |
| alpine/socat | socat image, alpine based | 8.5 MB |

As you can see, storage requirements are not very high, thanks to the use of Alpine images. This can probably be further optimized by removing some of the default plugins, but that would make the server less usable for other things. I haven't looked into it thoroughly yet.

## Known Issues

- The primary user, defined in [example.env](example.env), is read by Jenkins as "ambiguous". This is **NOT** a security risk, as this is only a risk in servers that allow new users to sign up. This is not the case here, as the server is pre-configured with a single user. Furthermore, this is easily fixed in the UI with a single click. If I had more time, I would have fixed this, but so far I didn't 😊.
