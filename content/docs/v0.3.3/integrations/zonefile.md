+++
title = "Zonefile"
description = 'The Zonefile integration.'
date = 2023-08-16T13:53:00+02:00
updated = 2023-11-03T16:18:39+01:00
draft = false
weight = 2
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Distills Zones into RFC-1035 compatible text representations of zones.'
toc = true
top = false
+++

The Zonefile integration takes [Zone](../custom-resources/zone.md) resources and produces
[RFC1035](https://datatracker.ietf.org/doc/html/rfc1035#section-5)-compatible text representations
of their collective records as ConfigMaps. These Zonefile configmaps can then in turn be used to serve DNS requests
for a domain.

You can push the contents of the `ConfigMap` to an extenral provider who supports the format, or serve the zonefile
directly from within the cluster by mounting the `ConfigMap` in a CoreDNS pod, and exposing it as a service.

Elaborating on this, you can then treat the CoreDNS service as the master, and set up transfers to externally managed DNS servers.

Or if you're feeling adventurous: expose CoreDNS as a `hostNetwork: true` pod on port `53`, and treat your Kubernetes nodes as authoritative for your entire domain.
