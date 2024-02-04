+++
title = "Advanced Use Cases"
description = "Covers some more advanced use cases for Kubizone"
date = 2023-11-06T20:26:16+01:00
updated = 2023-11-06T20:39:54+01:00
draft = false
weight = 4
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "More advanced use cases for Kubizone."
toc = true
top = false
+++

# Using zoneRef to decouple Records and Zones from a domains.
Using partial domain names and zone References (`zoneRef`) it is possible to define a
[`Record`](../custom-resources/record.md) or even an entire [`Zone`](../custom-resources/zone.md)
without knowing what the parent Zone's domain is ahead of time.

The fully qualified domain name of the record or zone will be automatically deduced,
and if you at some point in the future want to change the parent Zone's domain
name, you won't have to modify the `domainName` of all your child records or zones along
with it.

As an example, consider the following Zone:

```yaml
apiVersion: kubi.zone/v1alpha1
kind: Zone
metadata:
  name: my-zone
spec:
  domainName: example.org.
  delegations:
  - records:
    - pattern: "*"
```

And this child record:

```yaml
apiVersion: kubi.zone/v1alpha1
kind: Record
metadata:
  name: www-record
spec:
  domainName: www
  zoneRef:
    name: my-zone
  type: A
  rdata: "192.168.0.2"
```

With this setup `www.example.org.` will resolve to `192.168.0.2` as expected.

If we now edit our `my-zone` and change the top-level domain to `.com`, like so:

```yaml
spec:
  domainName: example.com.
```

Then the change will be automatically reflected in our record, and `www.example.com`
will now resolve correctly instead.

If we had instead defined our Record in terms of the expected fully qualified
domain name of `www.example.org.`, then the Record would simply be orphaned during
the change, since the parent domain would no longer match.

This method might also be useful in some cases where a lot of records have to be
shared between Zones, and can therefore just be duplicated, and have their `zoneRef`
changed, or in cases where the Zones are in separate namespaces, simply copied.


# Delegating across namespaces

Kubizone allows you to delegate responsibilities for subdomains across namespace
boundaries. This means that domain control can be managed with high granularity,
by narrowly scoping the types and domains which a Zone should adopt.

For example, this zone delegates responsibility for all all `*.dev.example.org`
records to the `development` namespace, and similarly allows adoption of DNS
records matching `*.prod.example.org` from the `production` namespace.

```yaml
---
apiVersion: kubi.zone/v1alpha1
kind: Zone
metadata:
  name: example-org
spec:
  domainName: example.org.
  delegations:
    - namespaces: ["development"]
      records:
      - pattern: "*.dev"
    - namespaces: ["production"]
      records:
      - pattern: "*.prod"
```

These rules ensure that under no circumstances will any Records matching `*.prod.example.org`
deployed into the `development` namespace be allowed to appear within this parent
zone's entries, and vice versa.

This record *will* be adopted by the Zone:

```yaml
---
apiVersion: kubi.zone/v1alpha1
kind: Record
metadata:
  name: www-dev-example-org
  namespace: development
spec:
  domainName: www.dev.example.org.
  type: A
  rdata: "192.168.0.2"
```

Whereas this record won't be adopted by the zone, because it's a .prod record in the
development namespace, which violates the delegation rules configured for the www-example zone.

```yaml
---
apiVersion: kubi.zone/v1alpha1
kind: Record
metadata:
  name: www-prod-example-org
  namespace: development
spec:
  domainName: www.prod.example.org.
  type: A
  rdata: "192.168.0.2"
```

These delegation rules can be much more specific than that, allowing only certain record
types to be adopted from specific namespaces. You might for instance have a namespace which
hosts all the incoming mailing infrastructure which is managed by an entirely separate team.
To allow them maximum freedom in doing their work, you can delegate the entire `MX` record
type, as well as all `A` and `AAAA` records for the `mail.example.org.` domain to their namespace:

```yaml
---
apiVersion: kubi.zone/v1alpha1
kind: Zone
metadata:
  name: example-org
spec:
  domainName: example.org.
  delegations:
    - namespaces: ["mailing"]
      records:
      - pattern: "*.mail"
        types: ["A", "AAAA"]
      - pattern: "*"
        types: ["MX"]
```