SUBDIRS =
#SUBDIRS += src

top_docs = README COPYING AUTHORS TODO NOTES

EXTRA_DIST =
EXTRA_DIST += $(top_docs)


# stage1 - 340M
build-es/.RUCKUS-DEBOOTSTRAP:
	debootstrap --arch=amd64 focal $(dir $@) http://archive.ubuntu.com/ubuntu/
	touch $@

build-%/proc/self:
	mount proc -t proc $(dir $@)

build-%/sys/block:
	mount sysfs -t sysfs $(dir $@)

build-%/dev/block:
	mount --rbind /dev $(dir $@)
	mount --make-rslave $(dir $@)


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
# - get chrooted
# - setup apt sources
#deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
#deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
#deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
#deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
#
# - do stage 2 apt install
#   - hold a couple packages? (doesn't seem to work very consistently...)
#     - apt-mark hold snapd wireguard-dkms
#   - ubuntu-server? (this does pull in snapd, says +425M)
#   - hand-made list? (w/ --no-install-recommends)
#pkgs=
#pkgs+=" alsa-ucm-conf alsa-topology-conf at bc cpio curl dirmngr"
#pkgs+=" dbus-user-session dconf-gsettings-backend dconf-service dmeventd dmidecode dosfstools ethtool finalrd fuse gawk gdisk gettext-base gir1.2-packagekitglib-1.0 git git-man glib-networking glib-networking-common"
#pkgs+=" glib-networking-services gnupg gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm groff-base gsettings-desktop-schemas iso-codes krb5-locales"
#pkgs+=" linux-base lshw lvm2 lz4"
#pkgs+=" mdadm ntfs-3g openssh-client packagekit packagekit-tools parted patch pci.ids pciutils perl perl-modules-5.30 pinentry-curses plymouth plymouth-theme-ubuntu-text policykit-1 publicsuffix"
#pkgs+=" software-properties-common sound-theme-freedesktop squashfs-tools thin-provisioning-tools"
#pkgs+=" tpm-udev udisks2 unattended-upgrades usb.ids usbutils vim vim-runtime xauth"
#
# left out these python packages
#pkgs+=" python-apt-common python3-apport python3-apt python3-attr python3-automat python3-blinker python3-certifi python3-chardet python3-click python3-colorama python3-configobj python3-constantly python3-cryptography python3-debconf python3-debian python3-distro python3-distro-info"
#pkgs+=" python3-distupgrade python3-entrypoints python3-gdbm python3-hamcrest python3-httplib2 python3-hyperlink python3-idna python3-incremental python3-jwt python3-keyring python3-launchpadlib python3-lazr.restfulclient python3-lazr.uri python3-newt python3-oauthlib python3-openssl"
#pkgs+=" python3-pexpect python3-problem-report python3-ptyprocess python3-pyasn1 python3-pyasn1-modules python3-requests python3-requests-unixsocket python3-secretstorage python3-service-identity python3-simplejson python3-software-properties python3-systemd python3-twisted"
#pkgs+=" python3-twisted-bin python3-update-manager python3-urllib3 python3-wadllib python3-zope.interface"
#
#
#   - iptables crda
#   - network-manager
#   - iptables-persistent (yes for saving rules)
#   - tshark (yes for non-root capture, dpkg-reconfigure later?)
#   - openssh-server ntp sntp ncurses-term
#   - dnsmasq
#   - cifs-utils smbclient samba attr samba-dsdb-modules samba-vfs-modules
#   - keyutils winbind libnss-winbind libpam-winbind
#   - acl
#   - bind9-host bind9-dnsutils net-tools
#   - vlan bridge-utils ifenslave
#   - postfix (no config, use dpkg-reconfigre to tweak later)
#   - mailutils
#   - zsh gkrellmd lm-sensors fancontrol read-edid i2c-tools
#   - git-email
#   - emacs-nox emacs-common-non-dfsg
#   - aspell aspell-en
#   - autofs nfs-common
#   - ftp ncftp
#   - wireguard-tools (--no-install-recommends to avoid grub junk - or apt-mark hold wireguard-dkms wireguard-modules)
#   - hdparm
#   - logwatch libdate-manip-perl
#   - lsof
#   - rfkill rsync
#   - info
#   - traceroute
#   - tree
#   - zip unzip unrar
#   - xattr
#   - zstd
#   - krb5-user (specified default realm, server (krb5), and pw server (krb5pw), use dpkg-reconfigure later?)
#   - libpam-krb5
#   - mosh
#   - wget
#
# DU: 1.1G (after apt clean)
#
#   - compilation toolchain (~300M)
#     - build-essential manpages-dev (190M)
#     - autoconf automake autopoint m4 libtool gettext pkg-config bison flex (13M)
#     - cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc (62M)
#     - yajl-tools
#
# DU: 1.4G (after apt clean)
#
#
build-es-mount-vfs: build-es/proc/self build-es/sys/block build-es/dev/block

# FIXME: apt error "Is /dev/pts mounted?"
#
build-es/.RUCKUS-ES-APT-PREP: build-es/.RUCKUS-DEBOOTSTRAP
	$(MAKE) build-es-mount-vfs
	echo "deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse" > build-es/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse" >> build-es/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse" >> build-es/etc/apt/sources.list
	echo "deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse" >> build-es/etc/apt/sources.list
	chroot $(dir $@) /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH apt update
	chroot $(dir $@) /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH apt upgrade -y --autoremove
	touch $@

# base list (pared down from ubuntu-server)
ES_PKGS =
ES_PKGS += alsa-ucm-conf alsa-topology-conf at bc cpio curl dirmngr
ES_PKGS += dbus-user-session dconf-gsettings-backend dconf-service dmeventd dmidecode dosfstools ethtool finalrd fuse gawk gdisk gettext-base gir1.2-packagekitglib-1.0 git git-man glib-networking glib-networking-common
ES_PKGS += glib-networking-services gnupg gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm groff-base gsettings-desktop-schemas iso-codes krb5-locales
ES_PKGS += linux-base lshw lvm2 lz4
ES_PKGS += mdadm ntfs-3g openssh-client packagekit packagekit-tools parted patch pci.ids pciutils perl perl-modules-5.30 pinentry-curses plymouth plymouth-theme-ubuntu-text policykit-1 publicsuffix
ES_PKGS += software-properties-common sound-theme-freedesktop squashfs-tools thin-provisioning-tools
ES_PKGS += tpm-udev udisks2 unattended-upgrades usb.ids usbutils vim vim-runtime xauth
# additional things we want
ES_PKGS += iptables crda
ES_PKGS += network-manager
ES_PKGS += iptables-persistent #(yes for saving rules, dpkg-reconfigure later?)
ES_PKGS += tshark #(yes for non-root capture, dpkg-reconfigure wireshark-common later?)
ES_PKGS += openssh-server ntp sntp ncurses-term
ES_PKGS += dnsmasq
ES_PKGS += cifs-utils smbclient samba attr samba-dsdb-modules samba-vfs-modules
ES_PKGS += keyutils winbind libnss-winbind libpam-winbind
ES_PKGS += acl
ES_PKGS += bind9-host bind9-dnsutils net-tools
ES_PKGS += vlan bridge-utils ifenslave
ES_PKGS += postfix #(no config, use dpkg-reconfigre to tweak later)
ES_PKGS += mailutils
ES_PKGS += zsh gkrellmd lm-sensors fancontrol read-edid i2c-tools
ES_PKGS += git-email
ES_PKGS += emacs-nox emacs-common-non-dfsg
ES_PKGS += aspell aspell-en
ES_PKGS += autofs nfs-common
ES_PKGS += ftp ncftp
ES_PKGS += wireguard-tools #(--no-install-recommends to avoid grub junk - or apt-mark hold wireguard-dkms wireguard-modules)
ES_PKGS += hdparm
ES_PKGS += logwatch libdate-manip-perl
ES_PKGS += lsof
ES_PKGS += rfkill rsync
ES_PKGS += info
ES_PKGS += traceroute
ES_PKGS += tree
ES_PKGS += zip unzip unrar
ES_PKGS += xattr
ES_PKGS += zstd
ES_PKGS += krb5-user #(specified default realm, server (krb5), and pw server (krb5pw), use dpkg-reconfigure later?)
ES_PKGS += libpam-krb5
ES_PKGS += mosh
ES_PKGS += wget
# compilation toolchain (~300M)
ES_PKGS += build-essential manpages-dev # (190M)
ES_PKGS += autoconf automake autopoint m4 libtool gettext pkg-config bison flex # (13M)
ES_PKGS += cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc # (62M)
ES_PKGS += yajl-tools

# FIXME: this is still interactive... i thought -y fixed that...?
#
#        looks like i need to create a config.dat to feed into apt via
#        DEBIAN_FRONTEND=noninteractive DEBCONF_DB_FALLBACK=Pipe
#
build-es/.RUCKUS-ES-APT: build-es/.RUCKUS-ES-APT-PREP
	$(MAKE) build-es-mount-vfs
	chroot $(dir $@) /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH apt install -y --no-install-recommends $(ES_PKGS)
	touch $@

# - get 3rd party stuff
#   - telegraf
build-es/.RUCKUS-ES-3RDPARTY: build-es/.RUCKUS-ES-APT
	$(MAKE) build-es-mount-vfs
	touch $@

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
build-es/.RUCKUS-ES-COMPILE: build-es/.RUCKUS-ES-3RDPARTY
	$(MAKE) build-es-mount-vfs
	touch $@

#build-es/.RUCKUS-ES: build-es/.RUCKUS-ES-COMPILE
#	touch $@


# apt clean
#
build-es/.RUCKUS-ES-FINAL: build-es/.RUCKUS-ES-COMPILE
	$(MAKE) build-es-mount-vfs
	touch $@

es.sqsh: build-es/.RUCKUS-ES-FINAL
	umount -R $(dir $^)/{proc,sys,dev}
	mksquashfs $(dir $^) $@ -comp lz4 -Xhc -noappend -wildcards -e .RUCKUS*


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



# Things to reconfigure before 1st boot, via chroot
#
# - hostname, mailname
# - dbus-uuidgen > /etc/machine-id
#
# - dpkg-reconfigure
#   - postfix (get the defaults seeded, run noninteractively to get fqdn right)
#   - tzdata
#   - locales
#   - krb5-user? (or remove krb5.conf until user manually does this or installs the file?)
#
