+++
title = "Introduction"
description = "Quick overview of the Kubizone project"
date = 2023-08-16T13:53:00+02:00
updated = 2023-11-03T16:18:39+01:00
draft = false
weight = 1
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "Quick overview of the Kubizone project"
toc = true
top = false
+++

Kubizone delivers Kubernetes-native Domain Name System (DNS) primitives such as 
[Records](../../custom-resources/record/) & [Zones](../../custom-resources/zone/),
and employs an [Operator](../../operators/kubizone.md) to condense relationships
and delegations between these into easy-to-process arrays of records, which can
be easily pushed to whichever DNS provider(s) you use.

## Purpose
Kubizone aims to be a more flexible and versatile replacement for projects such as
[external-dns](https://github.com/kubernetes-sigs/external-dns).

Instead of building integrations for 3rd party DNS service providers into Kubizone itself
like [external-dns](https://github.com/kubernetes-sigs/external-dns?tab=readme-ov-file#new-providers) does,
Kubizone instead exposes a very simple interface in the form of
[Records](../../custom-resources/record/) and [Zones](../../custom-resources/zone/),
which custom integrations in turn can use to update external DNS provider state.

These custom resources makes Kubizone both a lot simpler and a lot more powerful.
Since records are created directly, instead of via annotations on existing `Service`
and `Ingress` objects, they aren't limited to `A` and `AAAA` records, but can represent
the entire spectrum of possible record types.

## Overview
By default, Kubizone deploys the following [Custom Resources](../../custom-resources/):

1. [Records](../../custom-resources/record/) represents a single DNS Record entry, `A`, `AAAA`, `CNAME`, `TXT`, what have you.

2. [Zones](../../custom-resources/zone/) are collections of records, and can either be apex domains or subdomains.

3. [ZoneFile](../../custom-resources/zonefile/) is used by the [ZoneFile Operator](../../operators/zonefile/) to produce
   [RFC1035](https://www.rfc-editor.org/rfc/rfc1035)-compatible zonefile `ConfigMaps` which can be used with `bind` servers such as `CoreDNS`.

And the following Operators:

1. [Kubizone Operator](../../operators/kubizone/) monitors all [Records](../../custom-resources/record/)
   and [Zones](../../custom-resources/zone/) cluster-wide, keeping track of associations between them,
   and enriching them with all the information necessary for downstream providers to maintain
   external state, such as serials, zone hashes, compiled list of child records, etc.

2. [Zonefile Operator](../../operators/zonefile/) monitors [Zonefiles](../../custom-resources/zonefile/)
   and [Zones](../../custom-resources/zone/), compiling them into [RFC1035](https://www.rfc-editor.org/rfc/rfc1035)-compatible
   `ConfigMaps` which can then either be mounted directly into a `CoreDNS` (or similar) container and served,
   or pushed to an external DNS provider.

   This operator also acts as an example of a downstream provider, whose only interaction with the rest of the Kubizone project
   is as a passive observer of zone changes.

## Developing
If you're interested in building a custom provider integration for Kubizone, please read the section
on [Integrating](integrating.md) and take a look at the [Custom Resources](../../custom-resources/)
as well as the example [route53 integration written in bash](https://github.com/kubi-zone/route53-bash-integration).

Because of the architecture of Kubizone, your controller doesn't need to coordinate or embed itself in
Kubizone to be useful.

Instead, independent providers can simply run -- inside or outside -- of the cluster, watching for zone or
record changes, and acting accordingly.
In fact, this design allows **multiple** providers to co-exist, meaning you can easily mirror
your entire configuration, allowing you to easily fail over to a secondary authority in case of an outage.
