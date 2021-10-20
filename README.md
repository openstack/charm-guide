# OpenStack Charm Guide

[![tags][image-badge-cg]][image-link-openstack-tags]

The [OpenStack Charm Guide][cg] is the main source of information for the
the [OpenStack Charms][openstack-charms].

## Building

To build the guide run this simple command:

    tox

The resulting HTML files will be found in the `doc/build/html` directory. These
can be opened individually with a web browser or hosted by a local web server.

## Contributing

Documentation issues can be filed on [Launchpad][lp-bugs-cg].

This repository is under version control and is managed via the [OpenStack
Gerrit system][gerrit-openstack] (see the [OpenDev Developer’s
Guide][opendev-dev-guide]). For specific guidance on working with the
documentation hosted on [docs.openstack.org][link] please read the [OpenStack
Documentation Contributor Guide][openstack-doc-guide].

<!-- LINKS -->

[image-badge-cg]: https://governance.openstack.org/tc/badges/charm-guide.svg
[image-link-openstack-tags]: http://governance.openstack.org/tc/reference/tags/index.html
[cg]: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide
[openstack-charms]: https://launchpad.net/openstack-charms
[lp-bugs-cg]: https://bugs.launchpad.net/charm-guide/+filebug
[gerrit-openstack]: https://review.openstack.org
[opendev-dev-guide]: https://docs.openstack.org/infra/manual/developers.html
[openstack-doc-guide]: https://docs.openstack.org/doc-contrib-guide/index.html
[link]: https://docs.openstack.org
