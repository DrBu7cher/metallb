---
title: Release Notes
weight: 7
---

## Version 0.4.0 (not released yet)

[Documentation for this release](https://master--metallb.netlify.com)

Action required if upgrading from 0.3.x:

- MetalLB's use of Kubernetes labels has changed slightly to conform
  to Kubernetes best practices. If you were using a label match on
  `app: controller` or `app: speaker` Kubernetes labels to find
  MetalLB objects, you should now match on a combination of `app:
  metallb`, `component: controller` or `component: speaker`, depending
  on what objects you want to select.
- RBAC rules have changed, and now allow the MetalLB speaker to list
  and watch Node objects. If you are not installing MetalLB via the
  provided manifest, you will need to make this change by hand.
- If you want to switch to using Helm to manage your MetalLB
  installation, you must first uninstall the manifest-based version,
  with `kubectl delete -f metallb.yaml`.

New features:

- Initial IPv6 support! The `ndp` protocol allows v6 Kubernetes
  clusters to advertise their services using
  the
  [Neighbor Discovery Protocol](https://en.wikipedia.org/wiki/Neighbor_Discovery_Protocol),
  IPv6's analog to ARP. If you have an IPv6 Kubernetes cluster, please
  try it out
  and [file bugs](https://github.com/google/metallb/issues/new)!
- BGP peers now have
  a
  [node selector]({{% relref "configuration/_index.md" %}}#limiting-peers-to-certain-nodes). You
  can use this to integrate MetalLB into more complex cluster network
  topologies.
- MetalLB now has
  a
  [Helm chart](https://github.com/google/metallb/tree/master/helm/metallb). If
  you use [Helm](https://helm.sh) on your cluster, this should make it
  easier to track and manage your MetalLB installation. The chart will
  be submitted for inclusion in the main Helm stable repository
  shortly after the release is finalized. Use of Helm is optional,
  installing the manifest directly is still fully supported.

Other improvements:
- MetalLB
  now
  [backs off on failing BGP connections](https://github.com/google/metallb/issues/84),
  to avoid flooding logs with failures
- ARP mode should be a little
  more
  [interoperable with clients](https://github.com/google/metallb/issues/172),
  and failover should be a little faster, thanks to tweaks to its
  advertisement logic.
- ARP and NDP modes export [Prometheus](https://prometheus.io) metrics
  for requests received, responses sent, and failover-related
  transmissions. This brings them up to "monitoring parity" with BGP
  mode.
- Binary internals were refactored to share more common code. This
  should reduce the amount of visual noise in the logs.

This release includes contributions from David Anderson, driftavalli,
Matt Layher, John Marcou, Paweł Prażak, and Hugo Slabbert. Thanks to
all of them for making MetalLB better!

## Version 0.3.1

[Documentation for this release](https://metallb.universe.tf)

Fixes a couple
of [embarrassing bugs](https://github.com/google/metallb/issues/142)
that sneaked into 0.3.

Bugfixes:

- Revert to using `apps/v1beta2` instead of `apps/v1` for MetalLB's
  Deployment and Daemonset, to remain compatible with Kubernetes 1.8.
- Create the `metallb-system` namespace when installing
  `test-bgp-router`.
- Disable BIRD in `test-bgp-router`. Bird got updated to 2.0, and the
  integration with `test-bgp-router` needs some reworking.

## Version 0.3.0

[Documentation for this release](https://v0-3-0--metallb.netlify.com)

Action required if upgrading from 0.2.x:

- The `bgp-speaker` DaemonSet has been renamed to just
  `speaker`. Before applying the manifest for 0.3.0, delete the old
  daemonset with `kubectl delete -n metallb-system
  ds/bgp-speaker`. This will take down your load-balancers until you
  deploy the new DaemonSet.
- The
  [configuration file format](https://raw.githubusercontent.com/google/metallb/master/manifests/example-config.yaml) has
  changed in a few backwards-incompatible ways. You need to update
  your ConfigMap by hand:
  - Each `address-pool` must now have a `protocol` field, to select
    between ARP and BGP mode. For your existing configurations, add
    `protocol: bgp` to each address pool definition.
  - The `advertisements` field of `address-pool` has been renamed to
    `bgp-advertisements`, and is now optional. If you don't need any
    special advertisement settings, you can remove the section
    entirely, and MetalLB will use a reasonable default.
  - The `communities` section has been renamed to `bgp-communities`.

New features:

- MetalLB now supports ARP advertisement, enabled by setting
  `protocol: arp` on an address pool. ARP mode does not require any
  special network equipment, and minimal configuration. You can follow
  the [ARP mode tutorial]({{% relref "tutorial/arp.md" %}}) to get
  started. There is also a page about ARP
  mode's [behavior and tradeoffs]({{% relref "concepts/arp.md" %}}),
  and documentation
  on [configuring ARP mode]({{% relref "configuration/_index.md" %}}).
- The container images are
  now
  [multi-architecture images](https://blog.docker.com/2017/11/multi-arch-all-the-things/). MetalLB
  now supports running on all supported Kubernetes architectures:
  amd64, arm, arm64, ppc64le, and s390x.
- You can
  now
  [disable automatic address allocation]({{% relref "configuration/_index.md" %}}#controlling-automatic-address-allocation) on
  address pools, if you want to have manual control over the use of
  some addresses.
- MetalLB pods now come
  with
  [Prometheus scrape annotations](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml). If
  you've configured your Prometheus-on-Kubernetes to automatically
  discover monitorable pods, MetalLB will be discovered and scraped
  automatically. For more advanced monitoring needs,
  the
  [Prometheus Operator](https://coreos.com/operators/prometheus/docs/latest/user-guides/getting-started.html) supports
  more flexible monitoring configurations in a Kubernetes-native way.
- We've documented how
  to
  [Integrate with the Romana networking system]({{% relref "configuration/romana.md" %}}),
  so that you can use MetalLB alongside Romana's BGP route publishing.
- The website got a makeover, to accommodate the growing amount of
  documentation in a discoverable way.

This release includes contributions from David Anderson, Charles
Eckman, Miek Gieben, Matt Layher, Xavier Naveira, Marcus Söderberg,
Kouhei Ueno. Thanks to all of them for making MetalLB better!

## Version 0.2.1

[Documentation for this release](https://v0-2-1--metallb.netlify.com)

Notable fixes:

- MetalLB unable to start because Kubernetes cannot verify that
  "nobody" is a non-root
  user ([#85](https://github.com/google/metallb/issues/85))

## Version 0.2.0

[Documentation for this release](https://v0-2-0--metallb.netlify.com)

Major themes for this version are: improved BGP interoperability,
vastly increased test coverage, and improved documentation structure
and accessibility.

Notable features:
 
- This website! It replaces a loose set of markdown files, and
  hopefully makes MetalLB more accessible.
- The BGP speaker now speaks Multiprotocol BGP
  ([RFC 4760](https://tools.ietf.org/html/rfc4760)). While we still
  only support IPv4 service addresses, speaking Multiprotocol BGP is a
  requirement to successfully interoperate with several popular BGP
  stacks. In particular, this makes MetalLB compatible
  with [Quagga](http://www.nongnu.org/quagga/) and Ubiquiti's
  EdgeRouter and Unifi product lines.
- The development workflow with Minikube now works with Docker for
  Mac, allowing mac users to hack on MetalLB. See
  the [hacking documentation]({{% relref "community/_index.md" %}})
  for the required additional setup.

Notable fixes:

- Handle multiple BGP peers properly. Previously, bgp-speaker
  mistakenly made all its connections to the last defined peer,
  ignoring the others.
- Fix a startup race condition where MetalLB might never allocate an
  IP for some services.
- Test coverage is above 90% for almost all packages, up from ~0%
  previously.
- Fix yaml indentation in the MetalLB manifests.

## Version 0.1.0

[Documentation for this release](https://github.com/google/metallb/tree/v0.1)

This was the first tagged version of MetalLB. Its changelog is
effectively "MetalLB now exists, where previously it did not."
