SUBDIRS =
#SUBDIRS += src

top_docs = README COPYING AUTHORS TODO NOTES

EXTRA_DIST =
EXTRA_DIST += $(top_docs)


build-es/.RUCKUS-DEBOOTSTRAP:
	debootstrap --arch=amd64 focal $(dir $@) http://archive.ubuntu.com/ubuntu/
	touch $@

# ES (base)
#
# This is going to be used for most of our domUs.  Should include
# what's needed for srv-router, srv-net, srv-auth.
#
# FIXME: we could split srv-auth's extra stuff into ES-auth if we
#        really want... but it strikes me as overkill.  It's really
#        just the difference between including the samba package or
#        not (~45M).  That would also make our base image unable to export
#        files via smbd, but that might be fine.
#
# FIXME: had to "export PATH" to get certain things happy
#        post-chroot... not sure (e.g., emacs dired, dpkg-reconfigure)
#
# - get chrooted
# - setup apt sources
#deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
#deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
#deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
#deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
#
# - do stage 2 apt install
#   - hold a couple packages
#     - apt-mark hold snapd wireguard-dkms
#   - ubuntu-server (this does pull in snapd)
#   - network-manager
#   - iptables-persistent (yes for saving rules)
#   - tshark (yes for non-root capture)
#   - openssh-server ntp
#   - dnsmasq
#   - cifs-utils smbclient
#   - keyutils winbind libnss-winbind libpam-winbind
#   - attr acl
#   - bind9-host bind9-dnsutils net-tools
#   - samba
#   - vlan bridge-utils ifenslave
#   - postfix (no config, use dpkg-reconfigre to tweak later)
#   - mailutils
#   - zsh gkrellmd lm-sensors fancontrol read-edid i2c-tools
#   - git git-man git-email
#   - emacs-nox emacs-common-non-dfsg liblcms2-utils
#   - aspell
#   - autofs
#   - ftp ncftp
#   - wireguard (--no-install-recommends to avoid grub junk - or apt-mark hold wireguard-dkms wireguard-modules)
#   - hdparm
#   - logwatch
#   - lsof
#   - rfkill rsync
#   - info texinfo
#   - traceroute
#   - tree
#   - zip unzip unrar
#   - xattr
#   - zstd
#   - krb5-user libpam-krb5 (specified default realm, server, and pw server, use dpkg-reconfigure later?)
#   - compilation toolchain (~350M)
#     - build-essential manpages-dev (190M)
#     - autoconf automake autopoint m4 libtool gettext pkg-config bison flex (14M)
#     - cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc (125M)
#     - yajl-tools
#   - mosh
#
# LEFT OFF HERE: 1.9G
#
# FIXME: bah, i've already got snapd?  where'd it come from?
#
# - get 3rd party stuff
#   - telegraf
# - get ready to compile stuff (if not available via ppa)
#   - openzfs
#   - ruckusrd
#   - pingenator
#   - awesome-powers (just clone this in /scrap)
# - additional apt packages (that would have installed ubuntu deps)
#   - zfs-auto-snapshot (or make it an awesome-powers submodule?)
#   - sanoid (or make it an awesome-powers submodule?)
#
# FIXME: should I be compiling these things in here, or compiling them
#        somewhere else and installing them via another ppa?
#
#        given that I don't *have* them compiled yet, and I'm
#        bootstrapping a new version of Ubuntu, so I don't even have a
#        build environment prepped yet... i guess we have to build
#        them in here.
#
#        but maybe for subsequent builds, we could check availability
#        via apt?  wget debs from somewhere?  keep debs in this source
#        tree and use if found?
#
build-es/.RUCKUS-ES: build-es/.RUCKUS-DEBOOTSTRAP
	mount proc -t proc $(dir $@)/proc
	mount sysfs -t sysfs $(dir $@)/sys
	mount --bind /dev $(dir $@)/dev
	echo chroot $(dir $@) /usr/bin/env -i HOME=/root TERM=\$TERM PATH=\$PATH/bin/bash --login
	touch $@

# FIXME: temporary PHONY for testing deps... remove when done
.PHONY: es.sqsh
es.sqsh: build-es/.RUCKUS-ES
	echo mksquashfs $(dir $^) $@ -comp lz4 -Xhc -noappend -wildcards -e .RUCKUS*


# ES-file
#
# Really basic file-server... we use ES-wine at CAH so we can run
# AVIMarkServer via wine on the file server.
#
# FIXME: Is there really anything special we need for a file server
#        that we couldn't just stick in the base ES image?  If so,
#        build an es-file.sqsh between es.sqsh and es-web.sqsh


# ES-web
#
# - via apt
#   - apache2
#   - influxdb (upstream ppa)
#   - grafana (upstream ppa)
#   - unifi (wget the .deb?)


# ES-wine
#
# - enable 32bit libs across the board
# - apt
#   - gcc-multilib (?) (85M)
#   - wine (via upstream ppa)
# 
#es-wine.sqsh: builde-es-wine/.DONE
#	mksquashfs ...


# VMM (Xen Dom0)
#
# - compile things
#   - xen
# - apt
#   - efibootmgr
#   - syslinux extlinux isolinux
#   - ntfs3g?
#   - fwupd
#   - fuse?
#   - dkms
#   - smartmontools
#   - drbd9 (should I wait until here and use upstream ppa?)
# - compile/apt
#   - ruckusrd (if we haven't already)


# Workstation
#
# - via apt
#   - ubuntu-desktop (maybe exclude a few pkgs?)
#   - wireshark gkrellm emacs gitk git-gui
#   - firefox
#   - libreoffice
#   - teamviewer (because it won't really work until I have graphical boot?)
#   - audacity audacious rhythmbox sox ogg123 vorbis-tools etc
#   - jackd
#   - gimp inkscape
#   - vlc (upstream ppa?)
#   - dvd player stuff (maybe?)
#   - special font packages (maybe?)
#   - rdesktop (and more?)
# - 3rd party stuff
#   - google-chrome google-earth (wget the .deb?)
#   - ardour (wget the download)
#   - qgis (upstream ppa)
#   - steam (upstream ppa)
#   - openshot (wget upstream appimage)
#   - zoom (wget .deb)
#   - teams (wget .deb)
# - remove packages that got pulled in via ubuntu-desktop?
#   - thunderbird
#   - stupid autoupdate gui notification thing
#   - snapd?



# Things to reconfigure upon 1st boot
#
# - dpkg-reconfigure
#   - postfix
#   - tzdata
#   - locales
#   - tshark? wireshark?  make sure group exists and setuid set
#   - netfilter-persistent? make sure empty rules files exist
#   - krb5-user or libpam-krb5?
