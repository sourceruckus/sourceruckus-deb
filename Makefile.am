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


all: es.sqsh es-file.sqsh es-web.sqsh vmm.sqsh workstation.sqsh


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


################################################################################
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
PKGS_ES =
PKGS_ES += alsa-ucm-conf alsa-topology-conf at bc cpio curl dirmngr
PKGS_ES += dbus-user-session dconf-gsettings-backend dconf-service dmeventd dmidecode dosfstools ethtool finalrd fuse gawk gdisk gettext-base gir1.2-packagekitglib-1.0 git git-man glib-networking glib-networking-common
PKGS_ES += glib-networking-services gnupg gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm groff-base gsettings-desktop-schemas iso-codes krb5-locales
PKGS_ES += linux-base lshw lvm2 lz4
PKGS_ES += mdadm ntfs-3g openssh-client packagekit packagekit-tools parted patch pci.ids pciutils perl perl-modules-5.30 pinentry-curses plymouth plymouth-theme-ubuntu-text policykit-1 publicsuffix
PKGS_ES += software-properties-common sound-theme-freedesktop squashfs-tools thin-provisioning-tools
PKGS_ES += tpm-udev udisks2 unattended-upgrades usb.ids usbutils vim vim-runtime xauth
# additional things we want
PKGS_ES += iptables crda
PKGS_ES += network-manager
PKGS_ES += iptables-persistent #(yes for saving rules, dpkg-reconfigure later?)
PKGS_ES += tshark #(yes for non-root capture, dpkg-reconfigure wireshark-common later?)
PKGS_ES += openssh-server ntp sntp ncurses-term
PKGS_ES += dnsmasq
PKGS_ES += cifs-utils smbclient samba attr samba-dsdb-modules samba-vfs-modules
PKGS_ES += keyutils winbind libnss-winbind libpam-winbind
PKGS_ES += acl
PKGS_ES += bind9-host bind9-dnsutils net-tools
PKGS_ES += vlan bridge-utils ifenslave
PKGS_ES += postfix #(no config, use dpkg-reconfigre to tweak later)
PKGS_ES += mailutils
PKGS_ES += zsh gkrellmd lm-sensors fancontrol read-edid i2c-tools
PKGS_ES += git-email
PKGS_ES += emacs-nox emacs-common-non-dfsg
PKGS_ES += aspell aspell-en
PKGS_ES += autofs nfs-common
PKGS_ES += ftp ncftp
PKGS_ES += wireguard-tools #(--no-install-recommends to avoid grub junk - or apt-mark hold wireguard-dkms wireguard-modules)
PKGS_ES += hdparm
PKGS_ES += logwatch libdate-manip-perl
PKGS_ES += lsof
PKGS_ES += rfkill rsync
PKGS_ES += info
PKGS_ES += traceroute
PKGS_ES += tree
PKGS_ES += zip unzip unrar
PKGS_ES += xattr
PKGS_ES += zstd
PKGS_ES += krb5-user #(specified default realm, server (krb5), and pw server (krb5pw), use dpkg-reconfigure later?)
PKGS_ES += libpam-krb5
PKGS_ES += mosh
PKGS_ES += wget
PKGS_ES += man-db wamerican whois
# compilation toolchain (~300M)
PKGS_ES += build-essential manpages-dev # (190M)
PKGS_ES += autoconf automake autopoint m4 libtool gettext pkg-config bison flex # (13M)
PKGS_ES += cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc # (62M)
PKGS_ES += yajl-tools

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
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ES)
	touch $@

sysroot/etc/apt/sources.list.d/sourceruckus.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://sourceruckus.github.io/ppa/sourceruckus.key | $(CHROOT) apt-key add -
	echo "deb https://sourceruckus.github.io/ppa focal main" > $@

sysroot/etc/apt/sources.list.d/influxdb.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://repos.influxdata.com/influxdb.key | $(CHROOT) apt-key add -
	echo "deb https://repos.influxdata.com/ubuntu focal stable" > $@

sysroot/etc/apt/sources.list.d/grafana.list: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://packages.grafana.com/gpg.key | $(CHROOT) apt-key add -
	echo "deb https://packages.grafana.com/oss/deb stable main" > $@

# NOTE: We do this instead of using add-apt-repository to make things
#	a bit more consistent in here and to avoid multiple apt update
#	calls.
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

# NOTE: Definition of "3rd Party" in here is "upstream != Ubuntu".
#       That means our own Source Ruckus PPA is considered "3rd
#       Party".
#
# NOTE: Regarding the chicken-and-the-egg of the Source Ruckus PPA
#       while preping a distro-upgrade...  The easiest thing to do is
#       manually take a detour and build required packages by hand
#       instead of somehow detecting that need and automagically doing
#       it via the build system.  I'm gonna have to re-investigate
#       required packages and stuff by hand, there's no point in the
#       added complexity required to "automagically" build things
#       instead of installing.  So, after a 'make
#       sysroot/.RUCKUS-ES-3RDPARTY' (which should fail because
#       missing packages), manually chroot into the resulting system,
#       build packages and upload them to our PPA.
#
# NOTE: I'm including ruckusrd on ES even though I only intend to use
#       it as a virtual machine so that we have the included dev
#       scripts (e.g., doit, extractenator).  Also, I'm planning to
#       add sqsh_layer maintenance functionality to ruckusrd, which
#       we'll want on all the layers.
#
# NOTE: We prep the PPAs for the ES-file and ES-web additions now, so
#       that we can easily apt install things for testing and
#       corner-cases (e.g. combined file/web server).
#
PKGS_ES_3RDPARTY =
PKGS_ES_3RDPARTY += telegraf #(135M! didn't realize it was that large)
PKGS_ES_3RDPARTY += ruckusrd
PKGS_ES_3RDPARTY += pingenator
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/sourceruckus.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/influxdb.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/grafana.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/etc/apt/sources.list.d/linbit-drbd9.list
sysroot/.RUCKUS-ES-3RDPARTY: sysroot/.RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) apt update
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ES_3RDPARTY)
	touch $@

# the finalization step - set default boot target, disable some
# services, tweak some files, etc.
#
# NOTE: We clone awesome-powers and prep it's submodules in here, so
#       that it's available if we're installing w/out an internet
#       connection later on.
#
# NOTE: We do NOT create local users here.  That's handled by
#       awesome-powers during installation/groom.
#
# NOTE: We change UID_MIN and GID_MIN to 500 in /etc/login.defs
#       accross-the-board to differentiate local users from krb5
#       users.  Otherwise, if krb server is unreachable, it takes a
#       few minutes to authenticate local users.
#
# NOTE: We create globally-managed-devices.conf because all interfaces
#       end up "unmanaged" in NetworkManager w/out it.
#
sysroot/.RUCKUS-ES-FINAL: sysroot/.RUCKUS-ES-3RDPARTY
	$(MAKE) sysroot-mount-vfs
	sed -i 's|_MIN\([[:space:]]*\)1000|_MIN\1500|' sysroot/etc/login.defs
	echo "rootfs / auto defaults 0 0" > sysroot/etc/fstab
	touch sysroot/etc/NetworkManager/conf.d/10-globally-managed-devices.conf
	mkdir -p sysroot/scrap && cd sysroot/scrap && \
	    git clone https://github.com/sourceruckus/awesome-powers.git && \
	    cd awesome-powers && ./autogen.sh
	$(CHROOT) apt clean
	$(CHROOT) systemctl set-default multi-user.target
	$(CHROOT) systemctl disable dnsmasq
	touch $@

# FIXME: migrate to -comp zstd.  Ubuntu 18.04's squashfs doesn't
#        support it, but we could always compile our own in /usr/local
#        or something.
#
es.sqsh: sysroot/.RUCKUS-ES-FINAL
	umount -R $(dir $^)/{proc,sys,dev}
	mksquashfs $(dir $^) $@ -comp lz4 -Xhc -noappend -wildcards -e .RUCKUS*


################################################################################
# ES-file
#
# Based on ES, adds file-server specific packages.
#
# NOTE: This image includes upstream Wine (including full 32bit
#       library support) so you can run Windows junk.  For example, at
#       the vet hospital we run AVImarkServer.
#
#
# FIXME: todo
#
# - setup overlay w/ es.sqsh
# - enable 32bit libs across the board
# - apt
#   - gcc-multilib (?) (85M)
#   - wine (via upstream ppa)
#
PKGS_ESFILE =
PKGS_ESFILE += drbd-utils # from previously added upstream ppa
PKGS_ESFILE += openzfs # from previously added sourceruckus ppa # FIXME: get the pkg names right
PKGS_ESFILE += minidlna
PKGS_ESFILE += orthanc
PKGS_ESFILE += orthanc-dicomweb
PKGS_ESFILE += orthanc-doc
PKGS_ESFILE += orthanc-imagej
#PKGS_ESFILE += orthanc-mysql
#PKGS_ESFILE += orthanc-postgresql
PKGS_ESFILE += orthanc-webviewer
PKGS_ESFILE += orthanc-wsi


################################################################################
# ES-web
#
# Based on ES, not ES-file, and adds web-server stuff.
#
PKGS_ESWEB =
PKGS_ESWEB += apache2 ssl-cert
PKGS_ESWEB += influxdb # upstream ppa added already
PKGS_ESWEB += grafana # upstream ppa added already

# FIXME: Considered adding the unifi deb to my ppa... but I'm pretty
#        sure it adds it's own ppa as upstream for updates, so that's
#        probably a bad idea.  Instead, we'll have a list of local
#        .debs to install (and rules to download them via wget).
#
PKGS_ESWEB_LOCAL =
PKGS_ESWEB_LOCAL += unifi


################################################################################
# VMM (Xen Dom0)
#
# Based on ES, not ES-file or ES-web, adds things needed for our VMM
# setup.  This is the first layer that is intended for use on physical
# hardware (i.e., not a virtual machine), so it's where we start to
# add in bootloader and more hardware-centric things.
#
# - apt
#   - efibootmgr
#   - syslinux extlinux isolinux
#   - ntfs3g?
#   - fwupd
#   - dkms
#   - smartmontools
#   - drbd9 (should I wait until here and use upstream ppa?)
# - compile/apt
#   - ruckusrd (if we haven't already)
#   - xen
PKGS_VMM =
PKGS_VMM += drbd-utils # from previously added upstream ppa
PKGS_VMM += openzfs # from previously added sourceruckus ppa # FIXME: get the pkg names right
PKGS_VMM += groff ghostscript imagemagick libpaper1 netpbm psutils


################################################################################
# Workstation
#
# Based off VMM (which exludes ES-file and ES-web additions), adds
# everything else we want in an awesome workstation of doom (including
# the kitchen sink).
#
# NOTE: Similar to ES-web, we want a full multilib setup with upstream
#       Wine for the Workstation.
#
# - 32bit update
# - via apt
#   - ubuntu-desktop (maybe exclude a few pkgs?)
#   - wireshark gkrellm gitk git-gui
#   - emacs (replaces emacs-nox)
#   - firefox
#   - libreoffice
#   - audacity audacious rhythmbox sox ogg123 vorbis-tools etc
#   - jackd
#   - gimp inkscape
#   - vlc (via upstream snap?)
#   - dvd player stuff (maybe?)
#   - special font packages (maybe?)
#   - rdesktop (and more?)
# - 3rd party stuff
#   - wine (upstream ppa)
#   - teamviewer (wget the .deb, because it won't really work until I have graphical boot?)
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
#
sysroot/.RUCKUS-WORKSTATION-3RDPARTY: sysroot/etc/apt/sources.list.d/wine.list
sysroot/.RUCKUS-WORKSTATION-3RDPARTY: sysroot/etc/apt/sources.list.d/qgis.list
sysroot/.RUCKUS-WORKSTATION-3RDPARTY: sysroot/etc/apt/sources.list.d/steam.list



# Things for installer to do:
#
# FIXME: prompt (e.g., dialog, whiptail) or cli for values?
#
# - prompt for top layer (ES, ES-file, ES-web, VMM, or Workstation)
# - prompt for hostname (select from awesome-powers list? or just entry?)
# - prompt for domain name (or prompt for fqdn above?)
# - prompt for really destroy provided volume
#
# - create ext4 fs on provided device/volume
# - create upper fs hierarchy upper/{upper,work}
# - mount overlay appropriately
# - /etc/hostname, /etc/mailname
# - dbus-uuidgen > /etc/machine-id
#
# via chroot:
#
# - root pw?  localadmin pw?
# - /etc/fstab?
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
#   - get the following things cm'd:
#     - grkrellmd
#     - unattended-upgrades
#     - sshd.conf
#     - ssh.conf
#     - autofs
#     - telegraf.conf
#     - krb5.conf
#     - pam.d/common-passwd (or change localadmin uid to 500?)
#   - service enable/disable?
#   - useradd?
#
#
# - create new host.sqsh from overlay mount (perhaps?)
#
# NOTE: time-out... i think the installer might just end up living in
#       awesome-powers...
