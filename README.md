# OpenStack Charm Guide

[![tags][image-badge-cg]][image-link-openstack-tags]

The [OpenStack Charm Guide][cg] is the main source of information for the
[OpenStack Charms][openstack-charms].

## Building

To build the guide run this simple command:

    tox

The resulting HTML files will be found in the `doc/build/html` directory. These
can be opened individually with a web browser or hosted by a local web server.

## Contributing

This repository is under version control and is managed via the [OpenStack
Gerrit system][gerrit-openstack] (see the [OpenDev Developer’s
Guide][opendev-dev-guide]). The [Documentation contributions][cg-doc-contrib]
page outlines how to contribute to this project.

## Bugs

Documentation issues can be filed on [Launchpad][lp-bugs-cg].

<!-- LINKS -->

[image-badge-cg]: https://governance.openstack.org/tc/badges/charm-guide.svg
[image-link-openstack-tags]: http://governance.openstack.org/tc/reference/tags/index.html
[cg]: https://docs.openstack.org/charm-guide
[openstack-charms]: https://launchpad.net/openstack-charms
[lp-bugs-cg]: https://bugs.launchpad.net/charm-guide/+filebug
[gerrit-openstack]: https://review.openstack.org
[opendev-dev-guide]: https://docs.openstack.org/infra/manual/developers.html
[cg-doc-contrib]: https://docs.openstack.org/charm-guide/latest/community/doc-contrib/index.html
