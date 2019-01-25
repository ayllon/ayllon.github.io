---
title: Distributing software
---

Lately I have been looking at ways of distributing the beta version
of a piece of software written in C++. Basically, should we ship
`rpms` and/or `debs`? Some sort of self-contained package? I have to take
into account that a couple of the dependencies are not available on the
usual repositories, as they are not publicly released either. Some other
dependencies are available in some repositories (Fedora), but not in others
(EPEL, Ubuntu, Debian).

There are a few "new" ways of shipping software with everything self-contained.
Docker containers would be one example. However, this being a command-line tool
for end-users, I do not think that would be the best way.

But, is it me, or it is even a bigger mess now to distribute software?
There used to be a couple of ways (binaries, I mean), but now these containerized
solutions are making everything more complicated. Now it is not deb or rpm, but
rather deb, rpm, docker, Singularity, Flatpak, Snap, AppImage...? Oh my...

I went ahead and tried to actually package in all, or most of those solutions,
to get a hang of them, and a better ground to compare. The summary can be seen
on this table:

| Packaging   | System       | Root | Single file | Centralized     | Easy to use |
|-------------|--------------|------|-------------|-----------------|-------------|
| RPM/copr    | Fedora       | Yes  | No          | Yes (copr)      | Trivial     |
| RPM/epel    | Fedora       | Yes  | No          | Yes (epel)      | Trivial     |
| Deb         | Debian       | Yes  | No          | Yes (ppa)       | Trivial     |
| Docker      | Any          | No   | Yes         | Yes (DockerHub) | Hard        |
| Singularity | Linux        | No   | Yes         | No              | Trivial     |
| Flatpak     | Linux        | No   | Yes         | Yes (Flathub)   | Medium      |
| Snap        | Linux        | Yes  | Yes         | Yes (Store)     | Easy        |
| AppImage    | Linux        | No   | Yes         | No              | Trivial     |
| Homebrew    | Linux/MacOSX | No   | -           | Yes (GitHub)    | Medium      |

## Details

### [RPM/copr](http://copr.fedorainfracloud.org/)

* Works for Fedora and CentOS.
* Requires root for enabling the repo, and to install the RPMS.

```sh
sudo dnf copr enable "user/project"
```

* [Copr](http://copr.fedorainfracloud.org/). RPMS are pushed there and anyone
  can easily install and upgrade.
* Not subject to the distribution standards (no reviewing)

### [RPM/epel/fedora](https://fedoraproject.org/wiki/EPEL)

* Works for Fedora and CentOS.
* Requires root for installing (and to enable EPEL on CentOS/RHEL if not
  already there).
* Subject to the distribution standards, requires going through
  [peer review](https://fedoraproject.org/wiki/Packaging:ReviewGuidelines)
  before it is accepted.

### Deb

* Debian/Ubuntu.
* Requires root for installation and for enabling a PPA.
* For Ubuntu we have [PPA](https://launchpad.net/ubuntu/+ppas), but I have not
  found anything similar for Debian.

### [Docker](https://www.docker.com/get-started)

* Docker images can be run on Windows and MacOSX too using a virtual machine
  behind the scenes (VirtualBox/HyperKit).
* Root is not required, but the user needs to be allowed to run Docker
  containers.
* [Dockerhub](https://hub.docker.com/)
* Usage is not as straight forward (IMHO), since it requires, besides installing
  Docker, configuring properly user, volumes, permissions, etc.

### [Singularity](https://www.sylabs.io/docs/)

* Software is distributed as an self-executable containerized image
* Simpler to use than Docker, but with the same flexibility
* A runtime is required, but no daemon is involved (as is the case with Docker)
* It can be trivially built from Docker images

### [Flatpak](https://www.flatpak.org/)

* Any Linux. It is installed by default on Fedora. Requires manual installation
  on other distributions, but it widely available.
* Once Flatpak is installed, there is no need for `root` access to install
  applications, as they can be installed on the user `$HOME` directory.
* It can be distributed as a single `.flatpak` file embedding all dependencies.
* [Flathub](https://flathub.org/home) is the central repository, but there are
  [requirements](https://github.com/flathub/flathub/wiki/App-Requirements):
  i.e only desktop applications with a graphical interface.
* The runtime is shared with other flatpak applications, so if the user is
  already using [flatpak apps](https://flathub.org/apps/collection/popular) the
  impact is lower.
* For the usability, the user would need to add `~/.local/share/flatpak/exports/bin`
  to the `$PATH`, but, once done, the tool can be executed by its full qualified
  name: i.e `ch.unige.astro.sextractorxx`
* An alias can make the execution transparent (either point at that script, or
  to `flatpak run`).
* Manifest files are fairly straight-forward.

### [Snap](https://snapcraft.io/)

* Installed by default in Ubuntu. Support on other distributions seem to be
[so-so](https://kamikazow.wordpress.com/2018/06/08/adoption-of-flatpak-vs-snap-2018-edition/).
* Root is required to install and to build.
* As with Flatpak, the artifact is a single file.
* [Snapcraft store](https://snapcraft.io/store)
* It requires root to build because it installs the dependencies locally, which
  I really dislike.
* I find it trickier than Flatpak

### [AppImage](https://appimage.org)

* Works in any Linux, but one needs to be careful and build with the
  oldest-new-enough platform we can find, as newer versions of libc and similar
  are likely to be backwards-compatible, but if we compile with a modern system,
  older platforms [may not be able to run the binary](https://github.com/AppImage/AppImageKit/wiki/Creating-AppImages#binaries-compiled-on-old-enough-base-system).
* libfuse has to be installed, but normally it is on basically any modern Linux
  system. May not be available on some Docker images.
* No root required, no runtime (besides libfuse).
* [Torvalds likes it](https://plus.google.com/+LinusTorvalds/posts/WyrATKUnmrS)⸮
* Very very easy to use.

```bash
wget https://.../MyApp-x86_64.AppImage
chmod a+x MyApp-x86_64.AppImage
./MyApp-x86_64.AppImage --help
```

Handling the Python environment within the image is not trivial. We can not rely
on the system Python version, as packages may be missing and, besides, there are
ABI [incompatibility between Python versions](https://docs.python.org/3/c-api/stable.html).

### [Homebrew](https://brew.sh/)/[Linuxbrew](http://linuxbrew.sh/)

* Works both on MacOSX and [Linux](htts://linuxbrew.sh/)
* Does not require root.
* Custom "Tap" can be provided via [Github repos](https://github.com/).
* It builds from sources, unless pre-built binaries are provided. This could be
  potentially brittle.
* Reasonably low maintenance once everything is setup.

* [Example Homebrew tap](https://github.com/ayllon/homebrew-obsge)

The same manifests can be used both in Linux and MacOSX. However, Linux binaries
can not be specified on the same manifest file, as the MacOSX version will not
recognize it (pity). The other way around is fine.


## My conclusions
For Linux only, AppImage seems to be the most flexible option. Native formats
as RPM or DEB are not easily portable between distributions, so they require
a non trivial amount of maintenance.

If Fedora/CentOS/RHEL is good enough, copr can fit the bill.

Homebrew is a good option for MacOSX users. Guaranteeing that the software
will compile in any computer at any given time may prove to be complicated.
For instance, my tap was originally working, and when I wrote this document the
linking of boost-python was broken 😒
