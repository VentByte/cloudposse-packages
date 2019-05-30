<!-- This file was automatically generated by the `build-harness`. Make all changes to `README.yaml` and run `make readme` to rebuild this file. -->
[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

# Packages [![TravisCI Build Status](https://travis-ci.org/cloudposse/packages.svg?branch=master)](https://travis-ci.org/cloudposse/packages) [![Codefresh Build Status](https://g.codefresh.io/api/badges/build?repoOwner=cloudposse&repoName=packages&branch=master&pipelineName=apk&accountName=cloudposse&type=cf-1)](https://g.codefresh.io/repositories/cloudposse/packages/builds?filter=trigger:build;branch:master;service:5b234974667ab79287990636~packages) [![Latest Release](https://img.shields.io/github/release/cloudposse/packages.svg)](https://github.com/cloudposse/packages/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)


Cloud Posse distribution of awesome apps.


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps. 
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]




It's 100% Open Source and licensed under the [APACHE2](LICENSE).












## Introduction


Use this repo to easily install releases of popular Open Source apps. We provide a few ways to use it.

1. **Make Based Installer.** This installer works regardless of your OS and distribution. It downloads packages directly from their GitHub source repos and installs them to your `INSTALL_PATH`. 
2. **Alpine Linux Packages.** Use our Alpine repository to install prebuilt packages that use the original source binary (where possible) from the maintainers' official GitHub repo releases.
3. **Docker Image.** Use our docker image as a base-image or as part of a multi-stage docker build. The docker image always distributes the latest linux binaries for `x86_64` architectures.

See examples below for usage.

**Is one of our packages out of date?**

Open up an [issue](https://github.com/cloudposse/packages/issues) or submit a PR (*preferred*). We'll review quickly!

## Usage




### Alpine Repository (recommended)

A public Alpine repository is provided by [Cloud Posse](https://cloudposse.com). The repository is hosted on Amazon S3 and fronted by [CloudFlare's CDN](http://cloudflare.com) with end-to-end TLS. This ensures insane availability with DDoS mitigation and low-cost hosting. Using this alpine repository is ultimately more reliable than depending on [GitHub for availability](https://twitter.com/githubstatus) and provides an easier way to manage dependencies pinned at multiple versions. 

The repository itself is managed using [`alpinist`](https://github.com/cloudposse/alpinist), which takes care of the heavy lifting of building repository indexes. You can self-host your own Alpine repository using this strategy.

### Configure the alpine repository:

#### The Easy Way

We provide a bootstrap script to configure the alpine repository for your version of alpine. 

```
curl -sSL https://apk.cloudposse.com/install.sh | sh
```
__NOTE__: Requires `bash` and `curl` to run:

#### For Docker

Add the following to your `Dockerfile` near the top.
```
# Install the cloudposse alpine repository
ADD https://apk.cloudposse.com/ops@cloudposse.com.rsa.pub /etc/apk/keys/
RUN echo "@cloudposse https://apk.cloudposse.com/3.9/vendor" >> /etc/apk/repositories
```
__NOTE__: we support alpine `3.7`, `3.8`, and `3.9` packages at this time

### Installing Alpine Packages

When adding packages, we recommend using `apk add --update $package` to update the repository index before installing packages.

Simply install any package as normal:
```
apk add gomplate
```

But we recommend that you use version pinning:
```
apk add gomplate==3.0.0-r0
```

And maybe even repository pinning, so you know that you get our versions:
```
apk add gomplate@cloudposse==3.0.0-r0
```

### Makefile Interface

The `Makefile` interface works on OSX and Linux. It's a great way to distribute binaries in an OS-agnostic way which does not depend on a package manager (e.g. no `brew` or `apt-get`). 

This method is ideal for [local development environments](https://docs.cloudposse.com/local-dev-environments/) (which is how we use it) where you need the dependencies installed natively for your OS/architecture, such as installing a package on OSX.

See all available packages:
```
make -C install help
```

Install everything...
```
make -C install all
```

Install specific packages:
```
make -C install aws-vault chamber
```

Install to a specific folder:
```
make -C install aws-vault INSTALL_PATH=/usr/bin
```

Uninstall a specific package
```
make -C uninstall yq
```




## Examples

### Docker Multi-stage Build

Add this to a `Dockerfile` to install packages using a multi-stage build process:
```
FROM cloudposse/packages:latest AS packages

COPY --from=packages /packages/bin/kubectl /usr/local/bin/
```

### Docker with Git Clone

Or... add this to a `Dockerfile` to easily install packages on-demand:
```
RUN git clone --depth=1 -b master https://github.com/cloudposse/packages.git /packages && \
    rm -rf /packages/.git && \
    make -C /packages/install kubectl
```

### Makefile Inclusion

Sometimes it's necessary to install some binary dependencies when building projects. For example, we frequently 
rely on `gomplate` or `helm` to build chart packages.

Here's a stub you can include into a `Makefile` to make it easier to install binary dependencies.

```
export PACKAGES_VERSION ?= master
export PACKAGES_PATH ?= packages/
export INSTALL_PATH ?= $(PACKAGES_PATH)/vendor

## Install packages
packages/install:
        @if [ ! -d $(PACKAGES_PATH) ]; then \
          echo "Installing packages $(PACKAGES_VERSION)..."; \
          rm -rf $(PACKAGES_PATH); \
          git clone --depth=1 -b $(PACKAGES_VERSION) https://github.com/cloudposse/packages.git $(PACKAGES_PATH); \
          rm -rf $(PACKAGES_PATH)/.git; \
        fi

## Install package (e.g. helm, helmfile, kubectl)
packages/install/%: packages/install
        @make -C $(PACKAGES_PATH)/install $(subst packages/install/,,$@)

## Uninstall package (e.g. helm, helmfile, kubectl)
packages/uninstall/%:
        @make -C $(PACKAGES_PATH)/uninstall $(subst packages/uninstall/,,$@)
```



## Makefile Targets
```
assume-role               0.3.2      Easily assume AWS roles in your terminal.
atlantis                  0.4.13     Terraform For Teams
awless                    0.1.11     A Mighty CLI for AWS
aws-iam-authenticator     0.4.0      A tool to use AWS IAM credentials to authenticate to a Kubernetes cluster
aws-okta                  0.19.4     aws-okta allows users to authenticate with AWS using Okta credentials
aws-vault                 4.4.1      A vault for securely storing and accessing AWS credentials in development environments
chamber                   2.3.2      CLI for managing secrets
cli53                     0.8.12     Command line tool for Amazon Route 53
cloudflared               2018.8.0   Argo Tunnel client
codefresh                 0.19.5     Codefresh CLI
ctop                      0.7.1      Top-like interface for container metrics
direnv                    2.18.2     Unclutter your .profile
doctl                     1.15.0     A command line tool for DigitalOcean services
emailcli                  1.0.3      Command line email sending client written in Go.
fargate                   0.2.3      CLI for AWS Fargate
fetch                     0.3.1      fetch makes it easy to download files, folders, and release assets from a specific git commit, branch, or tag of public andssss
figurine                  0.2.2      Print your name in style
fzf                       0.18.0     A command-line fuzzy finder
ghr                       0.12.0     Upload multiple artifacts to GitHub Releases in parallel
github-commenter          0.5.0      Command line utility for creating GitHub comments on Commits, Pull Request Reviews or Issues
github-release            0.7.2      Commandline app to create and edit releases on Github (and upload artifacts)
github-status-updater     0.2.0      Command line utility for updating GitHub commit statuses and enabling required status checks for pull requests
gitleaks                  1.2.0      Audit git repos for secrets 🔑
gometalinter              2.0.11     Concurrently run Go lint tools and normalise their output
gomplate                  3.1.0      A flexible commandline tool for template rendering. Supports lots of local and remote datasources.
goofys                    0.19.0     a high-performance, POSIX-ish Amazon S3 file system written in Go
gosu                      1.10       Simple Go-based setuid+setgid+setgroups+exec
gotop                     1.5.0      A terminal based graphical activity monitor inspired by gtop and vtop
helm                      2.13.1     The Kubernetes Package Manager
helmfile                  0.54.0     Deploy Kubernetes Helm Charts
htmltest                  0.10.1     :white_check_mark: Test generated HTML for problems
hugo                      0.49.2     The world’s fastest framework for building websites.
json2hcl                  0.0.6      Convert JSON to HCL, and vice versa
k6                        0.22.1     A modern load testing tool, using Go and JavaScript - https://k6.io
kops                      1.12.1     Kubernetes Operations (kops) - Production Grade K8s Installation, Upgrades, and Management
kubecron                  1.0.2      Utilities to manage kubernetes cronjobs. Run a CronJob manually for test purposes. Suspend/unsuspend a CronJob
kubectl                   1.14.2     Production-Grade Container Scheduling and Management
kubectx                   0.6.1      Fast way to switch between clusters and namespaces in kubectl – [✩Star] if you're using it!
kubens                    0.6.1      Fast way to switch between clusters and namespaces in kubectl – [✩Star] if you're using it!
lectl                     0.17       Script to check issued certificates by Let's Encrypt on CTL (Certificate Transparency Log) using https://crt.sh
misspell                  0.3.4      Correct commonly misspelled English words in source files
packer                    1.3.1      Packer is a tool for creating identical machine images for multiple platforms from a single source configuration.
pandoc                    2.7.2      Universal markup converter
rakkess                   0.2.0      Review Access - kubectl plugin to show an access matrix for all available resources
rbac-lookup               0.3.0      Find Kubernetes roles and cluster roles bound to any user, service account, or group name.
retry                     3.3.0      ♻️ Functional mechanism based on channels to perform actions repetitively until successful.
scenery                   0.1.4      A Terraform plan output prettifier
shellcheck                0.5.0      ShellCheck, a static analysis tool for shell scripts
shfmt                     2.5.1      A shell parser, formatter and interpreter (POSIX/Bash/mksh)
slack-notifier            0.1.3      Command line utility to send messages with attachments to Slack channels via Incoming Webhooks
sops                      3.2.0      Secrets management stinks, use some sops!
stern                     1.8.0      ⎈ Multi pod and container log tailing for Kubernetes
sudosh                    0.1.4      Shell wrapper to run a login shell with `sudo` as the current user for the purpose of audit logging
teleport                  3.2.4      Privileged access management for elastic infrastructure.
terraform                 0.12.0     Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
terraform-0.11            0.11.14    Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
terraform-0.12            0.12.0     Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
terraform-docs            0.4.5      Generate docs from terraform modules
terragrunt                0.18.3     Terragrunt is a thin wrapper for Terraform that provides extra tools for working with multiple Terraform modules.
terrahelp                 0.6.3      Terrahelp is as a command line utility that provides useful tricks like masking of terraform output.
tfenv                     0.3.0      Transform environment variables for use with Terraform (e.g. `HOSTNAME` ⇨ `TF_VAR_hostname`)
tfmask                    0.2.0      Terraform utility to mask select output from `terraform plan` and `terraform apply`
variant                   0.28.0     Variant is a Universal CLI tool that works like a task runner
venona                    0.20.0     Codefresh runtime-environment agent
yq                        2.1.1      yq is a portable command-line YAML processor
```



## Share the Love 

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/packages)! (it helps us **a lot**) 

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)


## Related Projects

Check out these related projects.

- [build-harness](https://github.com/cloudposse/build-harness) - Collection of Makefiles to facilitate building Golang projects, Dockerfiles, Helm charts, and more
- [geodesic](https://github.com/cloudposse/geodesic) - Geodesic is the fastest way to get up and running with a rock solid, production grade cloud platform built on strictly Open Source tools.



## Help

**Got a question?**

File a GitHub [issue](https://github.com/cloudposse/packages/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## Commercial Support

Work directly with our team of DevOps experts via email, slack, and video conferencing. 

We provide [*commercial support*][commercial_support] for all of our [Open Source][github] projects. As a *Dedicated Support* customer, you have access to our team of subject matter experts at a fraction of the cost of a full-time engineer. 

[![E-Mail](https://img.shields.io/badge/email-hello@cloudposse.com-blue.svg)][email]

- **Questions.** We'll use a Shared Slack channel between your team and ours.
- **Troubleshooting.** We'll help you triage why things aren't working.
- **Code Reviews.** We'll review your Pull Requests and provide constructive feedback.
- **Bug Fixes.** We'll rapidly work to fix any bugs in our projects.
- **Build New Terraform Modules.** We'll [develop original modules][module_development] to provision infrastructure.
- **Cloud Architecture.** We'll assist with your cloud strategy and design.
- **Implementation.** We'll provide hands-on support to implement our reference architectures. 




## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Newsletter

Signup for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover. 

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/packages/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2019 [Cloud Posse, LLC](https://cpco.io/copyright)



## License 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.  

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Igor Rodionov][goruha_avatar]][goruha_homepage]<br/>[Igor Rodionov][goruha_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] |
|---|---|---|

  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://github.com/osterman.png?size=150
  [goruha_homepage]: https://github.com/goruha
  [goruha_avatar]: https://github.com/goruha.png?size=150
  [aknysh_homepage]: https://github.com/aknysh
  [aknysh_avatar]: https://github.com/aknysh.png?size=150



[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs
  [website]: https://cpco.io/homepage
  [github]: https://cpco.io/github
  [jobs]: https://cpco.io/jobs
  [hire]: https://cpco.io/hire
  [slack]: https://cpco.io/slack
  [linkedin]: https://cpco.io/linkedin
  [twitter]: https://cpco.io/twitter
  [testimonial]: https://cpco.io/leave-testimonial
  [newsletter]: https://cpco.io/newsletter
  [email]: https://cpco.io/email
  [commercial_support]: https://cpco.io/commercial-support
  [we_love_open_source]: https://cpco.io/we-love-open-source
  [module_development]: https://cpco.io/module-development
  [terraform_modules]: https://cpco.io/terraform-modules
  [readme_header_img]: https://cloudposse.com/readme/header/img?repo=cloudposse/packages
  [readme_header_link]: https://cloudposse.com/readme/header/link?repo=cloudposse/packages
  [readme_footer_img]: https://cloudposse.com/readme/footer/img?repo=cloudposse/packages
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?repo=cloudposse/packages
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img?repo=cloudposse/packages
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?repo=cloudposse/packages
  [share_twitter]: https://twitter.com/intent/tweet/?text=Packages&url=https://github.com/cloudposse/packages
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=Packages&url=https://github.com/cloudposse/packages
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/packages
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/packages
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/packages
  [share_email]: mailto:?subject=Packages&body=https://github.com/cloudposse/packages
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/packages?pixel&cs=github&cm=readme&an=packages
