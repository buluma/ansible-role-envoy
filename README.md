# [Ansible role envoy](#envoy)

Install and configure Envoy on your linux system.

|GitHub|GitLab|Downloads|Version|Issues|Pull Requests|
|------|------|-------|-------|------|-------------|
|[![github](https://github.com/buluma/ansible-role-envoy/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-envoy/actions)|[![gitlab](https://gitlab.com/shadowwalker/ansible-role-envoy/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-envoy)|[![downloads](https://img.shields.io/ansible/role/d/4699)](https://galaxy.ansible.com/buluma/envoy)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-envoy.svg)](https://github.com/buluma/ansible-role-envoy/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-envoy.svg)](https://github.com/buluma/ansible-role-envoy/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-envoy.svg)](https://github.com/buluma/ansible-role-envoy/pulls/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-envoy/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    envoy_systemd: true
    envoy_config:
      admin:
        address:
          socket_address: {address: 127.0.0.1, port_value: 9901}
      static_resources:
        listeners:
          - name: listener_0
            address:
              socket_address: {address: 127.0.0.1, port_value: 10000}
        filter_chains:
          - filters:
              - name: envoy.filters.network.http_connection_manager
                typed_config:
                  "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                stat_prefix: ingress_http
                codec_type: AUTO
                route_config:
                  name: local_route
                virtual_hosts:
                  - name: local_service
                    domains: ["*"]
                    routes:
                      - match: {prefix: "/"}
                        route: {cluster: some_service}
                http_filters:
                  - name: envoy.filters.http.router
                    typed_config:
                      "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
        clusters:
          - name: some_service
            connect_timeout: 0.25s
            type: STATIC
            lb_policy: ROUND_ROBIN
            load_assignment:
              cluster_name: some_service
              endpoints:
                - lb_endpoints:
                    - endpoint:
                        address:
                          socket_address:
                            address: 127.0.0.1
                            port_value: 1234

  roles:
    - role: "buluma.envoy"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-envoy/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  roles:
    - role: buluma.bootstrap
    - role: buluma.setuptools
    # - role: buluma.openssl
    - role: buluma.ca_certificates
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-envoy/blob/master/defaults/main.yml):

```yaml
---
envoy_bin_path: "/usr/local/bin"

# Disabled if installed as sidecar servie for example consul usage
# https://learn.hashicorp.com/tutorials/consul/consul-terraform-sync-intro?in=consul/network-infrastructure-automation#install-envoy-optional
envoy_enable_conf: false
envoy_conf_path: "/etc/envoy"

envoy_user: "envoy"
envoy_group: "envoy"
# wether envoy user should be a system user or not
envoy_system_user: true

# Download
# https://archive.tetratelabs.io/envoy/envoy-versions.json
envoy_version: "1.22.2"
envoy_arch: "linux-amd64"
# optional
# clear if unused
envoy_checksum: "sha256:fbd2460189f330a6b1e6b4ff79b4604dff1847091d4abbdda2c6b3894fadb396"
envoy_download_url: "https://archive.tetratelabs.io/envoy/download/v{{ envoy_version }}/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_download_path: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_unarchive_dest: "/tmp"
envoy_unarchive_creates: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}/bin/envoy"

# Create a systemd servicefile
envoy_systemd: false
envoy_systemd_enabled: true

# will be used with `to_nice_yaml`
envoy_config: []
# admin:
#   address:
#     socket_address: { address: 127.0.0.1, port_value: 9901 }
# static_resources:
#   listeners:
#   - name: listener_0
#     address:
#       socket_address: { address: 127.0.0.1, port_value: 10000 }
#     filter_chains:
#     - filters:
#       - name: envoy.filters.network.http_connection_manager
#         typed_config:
#           "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
#           stat_prefix: ingress_http
#           codec_type: AUTO
#           route_config:
#             name: local_route
#             virtual_hosts:
#             - name: local_service
#               domains: ["*"]
#               routes:
#               - match: { prefix: "/" }
#                 route: { cluster: some_service }
#           http_filters:
#           - name: envoy.filters.http.router
#             typed_config:
#               "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
#   clusters:
#   - name: some_service
#     connect_timeout: 0.25s
#     type: STATIC
#     lb_policy: ROUND_ROBIN
#     load_assignment:
#       cluster_name: some_service
#       endpoints:
#       - lb_endpoints:
#         - endpoint:
#             address:
#               socket_address:
#                 address: 127.0.0.1
#                 port_value: 1234
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-envoy/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Build Status GitHub](https://github.com/buluma/ansible-role-bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-bootstrap/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.setuptools](https://galaxy.ansible.com/buluma/setuptools)|[![Build Status GitHub](https://github.com/buluma/ansible-role-setuptools/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-setuptools/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-setuptools/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-setuptools)|
|[buluma.openssl](https://galaxy.ansible.com/buluma/openssl)|[![Build Status GitHub](https://github.com/buluma/ansible-role-openssl/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-openssl/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-openssl/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-openssl)|
|[buluma.ca_certificates](https://galaxy.ansible.com/buluma/ca_certificates)|[![Build Status GitHub](https://github.com/buluma/ansible-role-ca_certificates/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-ca_certificates/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-ca_certificates/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-ca_certificates)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-envoy/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/repository/docker/buluma/enterpriselinux/general)|all|
|[Amazon](https://hub.docker.com/repository/docker/buluma/amazonlinux/general)|Candidate|
|[Fedora](https://hub.docker.com/repository/docker/buluma/fedora/general)|all|
|[Ubuntu](https://hub.docker.com/repository/docker/buluma/ubuntu/general)|all|
|[Debian](https://hub.docker.com/repository/docker/buluma/debian/general)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-envoy/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-envoy/blob/master/CHANGELOG.md)

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-envoy/blob/master/LICENSE).

## [Author Information](#author-information)

[buluma](https://buluma.github.io/)

Please consider [sponsoring me](https://github.com/sponsors/buluma).

### [Special Thanks](#special-thanks)

Template inspired by [Robert de Bock](https://github.com/robertdebock)
