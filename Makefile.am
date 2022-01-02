SUBDIRS =
#SUBDIRS += src

top_docs = README COPYING AUTHORS TODO NOTES

EXTRA_DIST =
EXTRA_DIST += $(top_docs)

# The overall process is:
#
# - Build layer in sysroot
# - Create .sqsh of upper layer
# - Clean out sysroot
# - Prepare a new upper layer
# - Overlay-mount layers on sysroot for next image stage
#
# FIXME: consider using tmpfs for build dir(s).  makes cleaning them
#        out pretty easy, as you just unmount... but we'd be requiring
#        a bit more RAM (probably an extra 2G or so to be safe).
#
CHROOT=chroot sysroot /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH


# stage1 - 340M
sysroot/.RUCKUS-DEBOOTSTRAP:
	debootstrap --arch=amd64 focal $(dir $@) http://archive.ubuntu.com/ubuntu/
	touch $@

sysroot/proc/self:
	mount proc -t proc $(dir $@)

sysroot/sys/block:
	mount sysfs -t sysfs $(dir $@)

sysroot/dev/block:
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
sysroot-mount-vfs: sysroot/proc/self sysroot/sys/block sysroot/dev/block

sysroot/.RUCKUS-ES-APT-PREP: sysroot/.RUCKUS-DEBOOTSTRAP
	$(MAKE) sysroot-mount-vfs
	echo "deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse" > sysroot/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	echo "deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	$(CHROOT) apt update
	$(CHROOT) apt upgrade -y --autoremove
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
ES_PKGS += man-db wamerican whois
# compilation toolchain (~300M)
ES_PKGS += build-essential manpages-dev # (190M)
ES_PKGS += autoconf automake autopoint m4 libtool gettext pkg-config bison flex # (13M)
ES_PKGS += cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc # (62M)
ES_PKGS += yajl-tools

# NOTE: We want apt install to be completely noninteractive with
#       pre-seeded values for all debconf questions.  My initial idea
#       was to run through interactively and then just copy the
#       resulting debconf config.dat into subsequent runs, but reading
#       online kinda implies that that's the "wrong" thing to do, and
#       that I should use either DEBCONF_DB_FALLBACK or
#       DEBCONF_DB_OVERRIDE variables (set to either Pipe or File) to
#       supply values...  both of those methods were providing strange
#       errors, though...  so I tried just copying config.dat in
#       directly as I'd originally planned and it worked fine... so
#       that's what I'm doing.
#
APT_INSTALL=DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends
sysroot/.RUCKUS-ES-APT: sysroot/.RUCKUS-ES-APT-PREP
	$(MAKE) sysroot-mount-vfs
	cp debconf-config.dat sysroot/var/cache/debconf/config.dat
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(ES_PKGS)
	touch $@

sysroot/etc/apt/sources.list.d/sourceruckus.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://sourceruckus.github.io/ppa/sourceruckus.key | $(CHROOT) apt-key add -
	echo "deb https://sourceruckus.github.io/ppa focal main" > $@

sysroot/etc/apt/sources.list.d/influxdb.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://repos.influxdata.com/influxdb.key | $(CHROOT) apt-key add -
	echo "deb https://repos.influxdata.com/ubuntu focal stable" > $@

# NOTE: We do this instead of using add-apt-repository to make things
#       a bit more consistent in here and to avoid multiple apt update
#       calls.
sysroot/etc/apt/sources.list.d/linbit-drbd9.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) gpgconf --launch gpg-agent
	$(CHROOT) gpg --ignore-time-conflict --no-options --no-default-keyring \
	    --secret-keyring /etc/apt/secring.gpg \
	    --trustdb-name /etc/apt/trustdb.gpg \
	    --keyring /etc/apt/trusted.gpg \
	    --keyserver keyserver.ubuntu.com \
	    --recv CC1B5A793C04BB3905AD837734893610CEAA9512
	$(CHROOT) gpgconf --kill all
	echo "deb http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal main" > $@

ES_PKGS_3RD =
ES_PKGS_3RD += telegraf #(135M! didn't realize it was that large)
ES_PKGS_3RD += drbd-utils
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/sourceruckus.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/influxdb.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/linbit-drbd9.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) apt update
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(ES_PKGS_3RD)
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
sysroot/.RUCKUS-ES-COMPILE: sysroot/.RUCKUS-ES-3RDPARTY
	$(MAKE) sysroot-mount-vfs
	touch $@


# FIXME: add some systemctl enable/disable calls in here, and maybe
#        systemctl set-default multi-user.target
#
sysroot/.RUCKUS-ES-FINAL: sysroot/.RUCKUS-ES-COMPILE
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) apt clean
	$(CHROOT) useradd -c "Local Administrator" -m -s /usr/bin/zsh localadmin
	touch $@

es.sqsh: sysroot/.RUCKUS-ES-FINAL
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
VMM_PKGS =
VMM_PKGS += groff ghostscript imagemagick libpaper1 netpbm psutils

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



# Things for installer to do, via chroot
#
# - prompt for top layer
# - hostname, mailname
# - dbus-uuidgen > /etc/machine-id
# - root pw?  localadmin pw?
#
# - dpkg-reconfigure
#   - postfix (get the defaults seeded, run noninteractively to get fqdn right)
#   - tzdata
#   - locales
#   - krb5-user? (or remove krb5.conf until user manually does this or installs the file?)
#
# - awesome-powers configenator
#   - update && pull first?
#   - use specified hostname if unique with match, or prompt for host/site
