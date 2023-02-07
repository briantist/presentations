---
marp: true
theme: gaia
footer: https://github.com/briantist/presentations/
---

# galactory

<!-- _class: lead -->

An `ansible-galaxy` server and upstream proxy that uses Artifactory as its backend.

---

<!-- paginate: true -->

# whoami

Brian Scholer

GitHub: **[@briantist](https://github.com/briantist)**
IRC Libera.chat: **briantist**

- Member, [Ansible Community Steering Committee](https://docs.ansible.com/ansible/latest/community/steering/community_steering_committee.html)
- Maintainer, [`community.hashi_vault` collection](https://github.com/ansible-collections/community.hashi_vault)
- Maintainer/Author, [Ansible docs build GitHub actions](https://github.com/ansible-community/github-docs-build/)
- Contributor, Ansible core, various other collections

---

# About Artifactory

* Local repositories -- stores your Artifacts
* Remote repositories -- internal endpoint for remote repos
* Virtual repositories -- combines local and remote into one endpoint
* Stores many artifact types
  * pypi, apt, yum, docker, maven, nuget, go, etc.
  * [No support for galaxy](https://jfrog.atlassian.net/browse/RTFACT-12695) **:(**

---

# About Galaxy

### "galaxy" means ~3 things
* The `ansible-galaxy` command
* The "galaxy" protocol
* Galaxy servers
  * https://galaxy.ansible.com/ ("Public Galaxy")
  * &lt;Your Galaxy Here&gt; ("Private" galaxy servers)

---

# Private galaxy servers

* [galaxy_ng](https://github.com/ansible/galaxy_ng)
  * Slated to replace the software behind public galaxy
  * Lots of contributors, including those paid to do so
  * Full featured -- UI, search, labels, native authentication, etc.

---

# So why not galaxy_ng?

### About [pulp](https://pulpproject.org/)
* Stores artifacts of different types
* Supports content plugins for extending its repository types
* **galaxy_ng is a pulp plugin**
* Sounds a lot like Artifactory...

###### ...but I already run Artifactory

---

# About galactory

Galactory implments the Ansible Galaxy v2 protocol as a server, including the `/publish/` endpoint.

### Two main features
1. Store and retrieve collections via Artifactory
2. Transparently proxy your requests to an upstream galaxy server (like Public Galaxy) to seamlessly get access to more content (like Artifactory's "vritual" repositories)

---

# About galactory

### Basics

* Flask-based web application
* Collections only
* Just the essentials
  * no UI of its own
  * no collection management
  * no built-in authN/authZ support (passthru to Artifactory)
  * no native TLS/HTTPS support (will probably change)

---

# About galactory

### Direct collection storage

* Uses an Artifactory "Generic" repositry type
* Stores standard collection tarball + metadata (artifact properties)
* Galaxy protocol publish command
* Authentication (optional)
  * Handled in Artifactory
  * galactory auth to Artifactory (configured key)
  * Passthru `ansible-galaxy` token to Artifactory (per-request)

---

# About galactory

### Upstream proxying

* Transparently merges remote (upstream) API results with local (Artifactory) results
* Caches API results in Artifactory
* Retries, use cached results on failure
* Collection tarball from upstream becomes local on first serve
* Exclude namespaces from upstream requests
* **Avoid galaxy outages and 429 throttling errors**

---

# Ways to run

#### Python direct
```
python3 -m pip install galactory
python3 -m galactory --help
```

#### Container

```
docker run --rm ghcr.io/briantist/galactory:latest --help
```
<br />

##### Custom WSGI -- examples coming soon

---

# Ways to run - configuration

```
usage: python -m galactory [-h] [-c CONFIG] [--listen-addr LISTEN_ADDR]
                           [--listen-port LISTEN_PORT] [--server-name SERVER_NAME]
                           [--preferred-url-scheme PREFERRED_URL_SCHEME]
                           --artifactory-path ARTIFACTORY_PATH
                           [--artifactory-api-key ARTIFACTORY_API_KEY] [--use-galaxy-key]
                           [--prefer-configured-key] [--publish-skip-configured-key]
                           [--log-file LOG_FILE]
                           [--log-level {DEBUG,INFO,WARNING,ERROR,CRITICAL}] [--log-headers]
                           [--log-body] [--proxy-upstream PROXY_UPSTREAM]
                           [-npns NO_PROXY_NAMESPACE] [--cache-minutes CACHE_MINUTES]
                           [--cache-read CACHE_READ] [--cache-write CACHE_WRITE]
                           [--use-property-fallback]
                           [--health-check-custom-text HEALTH_CHECK_CUSTOM_TEXT]
```

---

# Ways to run - configuration

`~/.galactory/some.conf` ・ `/etc/galactory.d/some.conf`
```
artifactory-path            = https://artifactory.company.biz/artifactory/ansible-collections

proxy-upstream              = https://galaxy.ansible.com
no-proxy-namespace          = [company]

use-galaxy-key              = true
publish-skip-configured-key = true
prefer-configured-key       = true
```

---

# Ways to run - scenarios

#### On the controller
- Workstations
- CI pipelines
- Can be auth-less if publishing not required
- No service to run and maintain

---

# Ways to run - scenarios

#### Central service
- Single URL for everyone
- Can use credentials for cache/upstream writes while denying publishing
- Can use `ansible-galaxy` token for publishing instead (client provides auth)
- Simplifies `ansible.cfg`

---

# Acknowledgements

- https://github.com/sivel/amanda
  Amanda is the inspiration and code basis for galactory.
- https://github.com/jctanner/galaxy-mirror
  galaxy-mirror is a caching proxy for galaxy.

---

<!-- _class: lead -->

# Demo

---

<!-- _class: lead -->

# Q&A
