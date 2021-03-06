# libpod Installation Instructions

## Installing packaged versions of Podman

#### [Arch Linux](https://www.archlinux.org)

```bash
sudo pacman -S podman
```

#### [Fedora](https://www.fedoraproject.org), [CentOS](https://www.centos.org)

```bash
sudo yum -y install podman
```


#### [Fedora-CoreOS](https://coreos.fedoraproject.org), [Fedora SilverBlue](https://silverblue.fedoraproject.org)

Built-in, no need to install

#### [Gentoo](https://www.gentoo.org)

```bash
sudo emerge app-emulation/libpod
```

#### [openSUSE](https://www.opensuse.org)

```bash
sudo zypper install podman
```

#### [openSUSE Kubic](https://kubic.opensuse.org)

Built-in, no need to install

#### [RHEL7](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)

Subscribe, then enable Extras channel and install podman.

```bash
sudo subscription-manager repos --enable=rhel-7-server-extras-rpms
sudo yum -y install podman
```

#### [RHEL8 Beta](https://www.redhat.com/en/blog/powering-its-future-while-preserving-present-introducing-red-hat-enterprise-linux-8-beta?intcmp=701f2000001Cz6OAAS)

```bash
sudo yum module enable -y container-tools:1.0
sudo yum module install -y container-tools:1.0
```

#### [Ubuntu](https://www.ubuntu.com)

```bash
sudo apt-get update -qq
sudo apt-get install -qq -y software-properties-common
sudo add-apt-repository -y ppa:projectatomic/ppa
sudo apt-get update -qq
sudo apt-get -qq -y install podman
```

## Building from scratch

### Prerequisites

#### runc installed

The latest version of `runc` is expected to be installed on the system. It is picked up as the default runtime by podman.

#### conmon installed

The latest version of `conmon` is expected to be installed on the system. Conmon is used to monitor OCI Runtimes.

#### Setup CNI networking

A proper description of setting up CNI networking is given in the [`cni` README](cni/README.md).
But the gist is that you need to have some basic network configurations enabled and
CNI plugins installed on your system.

### Build and Run Dependencies

**Required**

Fedora, CentOS, RHEL, and related distributions:

```bash
yum install -y \
  atomic-registries \
  btrfs-progs-devel \
  conmon \
  containernetworking-cni \
  device-mapper-devel \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  go \
  golang-github-cpuguy83-go-md2man \
  gpgme-devel \
  iptables \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  make \
  ostree-devel \
  pkgconfig \
  runc \
  containers-common
```

Debian, Ubuntu, and related distributions:

```bash
apt-get install -y \
  btrfs-tools \
  git \
  golang-go \
  go-md2man \
  iptables \
  libassuan-dev \
  libdevmapper-dev \
  libglib2.0-dev \
  libc6-dev \
  libgpgme11-dev \
  libgpg-error-dev \
  libprotobuf-dev \
  libprotobuf-c0-dev \
  libseccomp-dev \
  libselinux1-dev \
  pkg-config
```

Debian, Ubuntu, and related distributions will also need to do the following setup:

 * A copy of the development libraries for `ostree`, either in the form of the `libostree-dev` package from the [flatpak](https://launchpad.net/~alexlarsson/+archive/ubuntu/flatpak) PPA, or built [from source](https://github.com/ostreedev/ostree) (more on that [here](https://ostree.readthedocs.io/en/latest/#building)). As of Ubuntu 18.04, `libostree-dev` is available in the main repositories, and the PPA is no longer required.
 * [Add required configuration files](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md#adding-required-configuration-files)
 * Install conmon, CNI plugins and runc
   * [Install conmon](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md#building-and-installing-conmon)
   * [Install CNI plugins](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md#installing-cni-plugins)
   * [runc Installation](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md#installing-runc) - Although installable, the latest runc is not available in the Ubuntu repos. Version 1.0.0-rc4 is the minimal requirement.

**NOTE**

If using an older release or a long-term support release, be careful to double-check that the version of `runc` is new enough (running `runc --version` should produce `spec: 1.0.0`), or else [build](https://github.com/containers/libpod/blob/master/docs/tutorials/podman_tutorial.md#installing-runc) your own.

Be careful to double-check that the version of golang is new enough, version 1.10.x or higher is required.  If needed, golang kits are available at https://golang.org/dl/

**Optional**

Fedora, CentOS, RHEL, and related distributions:

(no optional packages)

Debian, Ubuntu, and related distributions:

```bash
apt-get install -y \
  libapparmor-dev
```

### Get Source Code

As with other Go projects, PODMAN must be cloned into a directory structure like:

```
GOPATH
└── src
    └── github.com
	└── containers
	    └── libpod
```

First, configure a `GOPATH` (if you are using go1.8 or later, this defaults to `~/go`)
and then add $GOPATH/bin to your $PATH environment variable.

```bash
export GOPATH=~/go
mkdir -p $GOPATH
export PATH=$PATH:$GOPATH/bin
```

Next, clone the source code using:

```bash
mkdir -p $GOPATH/src/github.com/containers
cd $_ # or cd $GOPATH/src/github.com/containers
git clone https://github.com/containers/libpod # or your fork
cd libpod
```

### Build

```bash
make install.tools
make
sudo make install
```

Otherwise, if you do not want to build `podman` with seccomp support you can add `BUILDTAGS=""` when running make.

```bash
make BUILDTAGS=""
sudo make install
```

#### Build Tags

`podman` supports optional build tags for compiling support of various features.
To add build tags to the make option the `BUILDTAGS` variable must be set.

```bash
make BUILDTAGS='seccomp apparmor'
```

| Build Tag | Feature                            | Dependency  |
|-----------|------------------------------------|-------------|
| seccomp   | syscall filtering                  | libseccomp  |
| selinux   | selinux process and mount labeling | libselinux  |
| apparmor  | apparmor profile support           | libapparmor |

### Vendoring - Dependency Management

This project is using [vndr](https://github.com/LK4D4/vndr) for managing dependencies, which is a tedious and error-prone task. Doing it manually is likely to cause inconsistencies between the `./vendor` directory (i.e., the downloaded dependencies), the source code that imports those dependencies and the `vendor.conf` configuration file that describes which packages in which version (e.g., a release or git commit) are a dependency.

To ease updating dependencies, we provide the `make vendor` target, which fetches all dependencies mentioned in `vendor.conf`.  `make vendor` whitelists certain packages to prevent the `vndr` tool from removing packages that the test suite (see `./test`) imports.

The CI of this project makes sure that each pull request leaves a clean vendor state behind by first running the aforementioned `make vendor` followed by running `./hack/tree_status.sh` which checks if any file in the git tree has changed.

##### Vendor Troubleshooting

If the CI is complaining about a pull request leaving behind an unclean state, it is very likely right about it. Make sure to run `make vendor` and add all the changes to the commit. Also make sure that your local git tree does not include files not under version control that may reference other go packages. If some dependencies are removed but they should not, for instance, because the CI is needing them, then whitelist those dependencies in the `make vendor` target of the Makefile.  Whitelisting a package will instruct `vndr` to not remove if during its cleanup phase.

## Configuration files

### [registries.conf](https://src.fedoraproject.org/rpms/skopeo/blob/master/f/registries.conf)

#### Man Page: [registries.conf.5](https://github.com/containers/image/blob/master/docs/registries.conf.5.md)

`/etc/containers/registries.conf`

registries.conf is the configuration file which specifies which container registries should be consulted when completing image names which do not include a registry or domain portion.

#### Example from the Fedora `containers-common` package

```
cat /etc/containers/registries.conf
# This is a system-wide configuration file used to
# keep track of registries for various container backends.
# It adheres to TOML format and does not support recursive
# lists of registries.

# The default location for this configuration file is /etc/containers/registries.conf.

# The only valid categories are: 'registries.search', 'registries.insecure',
# and 'registries.block'.

[registries.search]
registries = ['docker.io', 'registry.fedoraproject.org', 'quay.io', 'registry.access.redhat.com', 'registry.centos.org']

# If you need to access insecure registries, add the registry's fully-qualified name.
# An insecure registry is one that does not have a valid SSL certificate or only does HTTP.
[registries.insecure]
registries = []


# If you need to block pull access from a registry, uncomment the section below
# and add the registries fully-qualified name.
#
[registries.block]
registries = []
```

### [mounts.conf](https://src.fedoraproject.org/rpms/skopeo/blob/master/f/mounts.conf)

`/usr/share/containers/mounts.conf` and optionally `/etc/containers/mounts.conf`

The mounts.conf files specify volume mount directories that are automatically mounted inside containers when executing the `podman run` or `podman build` commands.  Container process can then use this content.  The volume mount content does not get committed to the final image.

Usually these directories are used for passing secrets or credentials required by the package software to access remote package repositories.

For example, a mounts.conf with the line "`/usr/share/rhel/secrets:/run/secrets`", the content of `/usr/share/rhel/secrets` directory is mounted on `/run/secrets` inside the container.  This mountpoint allows Red Hat Enterprise Linux subscriptions from the host to be used within the container.

Note this is not a volume mount. The content of the volumes is copied into container storage, not bind mounted directly from the host.

#### Example from the Fedora `containers-common` package:

```
cat /usr/share/containers/mounts.conf
/usr/share/rhel/secrets:/run/secrets
```

### [seccomp.json](https://src.fedoraproject.org/rpms/skopeo/blob/master/f/seccomp.json)

`/usr/share/containers/seccomp.json`

seccomp.json contains the whitelist of seccomp rules to be allowed inside of
containers.  This file is usually provided by the containers-common package.

The link above takes you to the seccomp.json

### [policy.json](https://github.com/containers/skopeo/blob/master/default-policy.json)

`/etc/containers/policy.json`

#### Man Page: [policy.json.5](https://github.com/containers/image/blob/master/docs/policy.json.md)


#### Example from the Fedora `containers-common` package:

```
cat /etc/containers/policy.json
{
    "default": [
	{
	    "type": "insecureAcceptAnything"
	}
    ],
    "transports":
	{
	    "docker-daemon":
		{
		    "": [{"type":"insecureAcceptAnything"}]
		}
	}
}
```
