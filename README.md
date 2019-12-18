# HAProxy Exporter for Prometheus

Fork of https://github.com/prometheus/haproxy_exporter

This is a simple server that scrapes HAProxy stats and exports them via HTTP for
Prometheus consumption.

**\*Note:** since HAProxy 2.0.0, the official source includes a Prometheus exporter module that can be built into your binary with a single flag during build time and offers an exporter-free Prometheus endpoint. More information [down below](#official-prometheus-exporter).*

## Getting Started

To run it:

```bash
./haproxy_exporter [flags]
```

Help on flags:

```bash
./haproxy_exporter --help
```

For more information check the [source code documentation][gdocs].

[gdocs]: https://godoc.org/github.com/FundingCircle/haproxy_exporter

## Usage

### HTTP stats URL

Specify custom URLs for the HAProxy stats port using the `--haproxy.scrape-uri`
flag. For example, if you have set `stats uri /baz`,

```bash
haproxy_exporter --haproxy.scrape-uri="http://localhost:5000/baz?stats;csv"
```

Or to scrape a remote host:

```bash
haproxy_exporter --haproxy.scrape-uri="http://haproxy.example.com/haproxy?stats;csv"
```

Note that the `;csv` is mandatory (and needs to be quoted).

If your stats port is protected by [basic auth][], add the credentials to the
scrape URL:

```bash
haproxy_exporter  --haproxy.scrape-uri="http://user:pass@haproxy.example.com/haproxy?stats;csv"
```

You can also scrape HTTPS URLs. Certificate validation is enabled by default, but
you can disable it using the `--no-haproxy.ssl-verify` flag:

```bash
haproxy_exporter --no-haproxy.ssl-verify --haproxy.scrape-uri="https://haproxy.example.com/haproxy?stats;csv"
```

[basic auth]: https://cbonte.github.io/haproxy-dconv/configuration-1.6.html#4-stats%20auth

### Unix Sockets

As alternative to localhost HTTP a stats socket can be used. Enable the stats
socket in HAProxy with for example:


    stats socket /run/haproxy/admin.sock mode 660 level admin


The scrape URL uses the 'unix:' scheme:

```bash
haproxy_exporter --haproxy.scrape-uri=unix:/run/haproxy/admin.sock
```

### Building

```bash
make build
```

### Testing

[![CircleCI](https://circleci.com/gh/FundingCircle/haproxy_exporter/tree/master.svg?style=shield)][circleci]

```bash
make test
```

[circleci]: https://circleci.com/gh/FundingCircle/haproxy_exporter

### Releases

https://github.com/FundingCircle/haproxy_exporter/releases

0. Bump the VERSION file and merge to master
1. Create a tag and then create a release
2. Publish binaries:

```
git checkout <your-tag>

promu crossbuild
promu crossbuild tarballs

" You'll need
" brew install github-release
" export GITHUB_TOKEN=<your token from https://github.com/settings/tokens>

promu release -v .tarballs/
```

## License

Apache License 2.0, see [prometheus/haproxy_exporter/LICENSE](https://github.com/prometheus/haproxy_exporter/blob/master/LICENSE), [FundingCircle/haproxy_exporter/LICENSE](https://github.com/FundingCircle/haproxy_exporter/blob/master/LICENSE).

## Alternatives

### Official Repo

https://github.com/prometheus/haproxy_exporter

### Official Prometheus exporter

As of 2.0.0, HAProxy includes a Prometheus exporter module that can be built into your binary during build time.

To build with the official Prometheus exporter module, `make` with the following `EXTRA_OBJS` flag:

```bash
make TARGET=linux-glibc EXTRA_OBJS="contrib/prometheus-exporter/service-prometheus.o"
```

Once built, you can enable and configure the Prometheus endpoint from your `haproxy.cfg` file as a typical frontend:

```haproxy
frontend stats
    bind *:8404
    http-request use-service prometheus-exporter if { path /metrics }
    stats enable
    stats uri /stats
    stats refresh 10s
```

For more information, see [this official blog post](https://www.haproxy.com/blog/haproxy-exposes-a-prometheus-metrics-endpoint/).
