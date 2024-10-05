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
# FIXME: consider using tmpfs for build dir(s).  makes cleaning them out pretty
#        easy, as you just unmount... but we'd be requiring a bit more RAM
#        (probably an extra 2G or so to be safe).
#
CHROOT=chroot sysroot /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH


# NOTE: If --enable-devel was sepcified, we're building a devel-enabled base
#       image w/out 3rd party packages.
if ES_DEVEL
all: es-devel.sqsh
else
all: es.sqsh es-file.sqsh es-web.sqsh vmm.sqsh workstation.sqsh
endif


info:
	@echo DEBOOTSTRAP=$(DEBOOTSTRAP)
	@echo
	@echo PKGS_ES=\"$(PKGS_ES)\"
	@echo
	@echo PKGS_ES_3RDPARTY=\"$(PKGS_ES_3RDPARTY)\"
	@echo
	@echo PKGS_ESFILE=\"$(PKGS_ESFILE)\"
	@echo
	@echo PKGS_ESWEB=\"$(PKGS_ESWEB)\"
	@echo
	@echo PKGS_ESWEB_LOCAL=\"$(PKGS_ESWEB_LOCAL)\"
	@echo
	@echo PKGS_VMM=\"$(PKGS_VMM)\"
	@echo
	@echo PKGS_WORKSTATION=\"$(PKGS_WORKSTATION)\"
	@echo
	@echo PKGS_COMPILATION=\"$(PKGS_COMPILATION)\"
	@echo
	@echo PKGS_ZFS=\"$(PKGS_ZFS)\"

# stage1 - 211M
#
# NOTE: We change UID_MIN and GID_MIN to 500 in /etc/login.defs
#       accross-the-board to differentiate local users from krb5 users.
#       Otherwise, if krb server is unreachable, it takes a few minutes to
#       authenticate local users.  We have to make this change as early as
#       possible to keep apt install from adding users with uid higher than 500
#       (i.e., if we wait until "finalization" of es.sqsh, it's too late -
#       we've got gkrellmd and telegraf as 999 and 998).
#
DEBOOTSTRAP_DIR=$(abs_srcdir)/debootstrap
DEBOOTSTRAP=$(DEBOOTSTRAP_DIR)/debootstrap
.build/RUCKUS-DEBOOTSTRAP:
	$(MAKE) sysroot-clean
	$(MKDIR_P) $(dir $@)
	DEBOOTSTRAP_DIR=$(DEBOOTSTRAP_DIR) $(DEBOOTSTRAP) --arch=amd64 noble sysroot http://archive.ubuntu.com/ubuntu/
	sed -i 's|_MIN\([[:space:]]*\)1000|_MIN\1500|' sysroot/etc/login.defs
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
# This is going to be used for most of our domUs.  Should include what's needed
# for srv-router, srv-net, srv-auth.
#
# FIXME: we could split srv-auth's extra stuff into ES-auth if we really
#        want... but it strikes me as overkill.  It's really just the
#        difference between including the samba package or not (~45M).  That
#        would also make our base image unable to export files via smbd, but
#        that might be fine.
#
sysroot-mount-vfs: sysroot/proc/self sysroot/sys/block sysroot/dev/block

sysroot-umount-vfs:
	umount -R sysroot/{proc,sys,dev} || echo already unmounted

# after updates, sysroot is ~280M
.build/RUCKUS-ES-APT-PREP: .build/RUCKUS-DEBOOTSTRAP
	$(MAKE) sysroot-mount-vfs
	echo "deb http://archive.ubuntu.com/ubuntu/ noble main restricted universe multiverse" > sysroot/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ noble-updates main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	echo "deb http://archive.ubuntu.com/ubuntu/ noble-backports main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	echo "deb http://security.ubuntu.com/ubuntu/ noble-security main restricted universe multiverse" >> sysroot/etc/apt/sources.list
	$(CHROOT) apt update
	$(CHROOT) apt upgrade -y --autoremove
	touch $@

# base list (pared down from ubuntu-server)
#
# This base package list is generated manually by reviewing the
# packages that would be installed by ubuntu-server, removing some
# stuff we don't want, and adding some stuff we do.  The resulting
# image should be tiny and awesome.
#
# 1. apt install ubuntu-server [n]
# 2. Review apps (skip over libs and python mods, remove initramfs
#    stuff, remove vm/cloud stuff).
# 3. apt install --no-install-recommends $PKGS_ES [n]
# 4. Compare and contrast, try again until you like the list.
#
# W/out removing things, ubuntu-server adds ~420M.  All our pruning
# reduces that to just an additional ~240M, then we add in things we
# actually want.
#
# Removed
# - apparmor
# - apport apport-*
# - appstream
# - bcache-tools
# - busybox-initramfs
# - byobu
# - cloud-*
# - cryptsetup-initramfs
# - dracut-*
# - fonts-ubuntu-console
# - fwupd fwupd-signed
# - initramfs-tools initramfs-tools-*
# - klibc-utils
# - landscape-common
# - lxd-*
# - multipath-tools (pulled in initramfs stuff)
# - open-vm-tools
# - overlayroot
# - pastebinit
# - plymouth plymouth-*
# - screen
# - sbsigntool secureboot-db
# - sg3-utils sg3-utils-udev (pulled in initramfs stuff)
# - snapd
# - sosreport
# - tmux
# - tpm-udev
# - ubuntu-server
# - update-manager-core update-notifier-common
#
# NOTE: utils to play with: kexec htop run-one zerofree
#
PKGS_ES =
PKGS_ES += bc bolt btrfs-progs cpio cryptsetup cryptsetup-bin curl
PKGS_ES += dirmngr dmeventd dmidecode dosfstools
PKGS_ES += ethtool fdisk finalrd fuse3 gawk gdisk gettext-base gir1.2-packagekitglib-1.0 git git-man gnupg
PKGS_ES += gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpgconf gpgsm groff-base htop jq keyboxd
PKGS_ES += kpartx linux-base lshw lvm2
PKGS_ES += mdadm modemmanager motd-news-config needrestart ntfs-3g open-iscsi openssh-client
PKGS_ES += packagekit packagekit-tools parted patch pci.ids pciutils perl perl-modules-5.38 pinentry-curses
PKGS_ES += polkitd pollinate publicsuffix
PKGS_ES += run-one sgml-base software-properties-common
PKGS_ES += squashfs-tools tcl tcl8.6 thin-provisioning-tools ubuntu-release-upgrader-core udisks2 unattended-upgrades
PKGS_ES += usb-modeswitch usb-modeswitch-data usb.ids vim vim-runtime xauth xfsprogs xml-core xz-utils zerofree zstd
# additional things we want (~730M last time I checked)
PKGS_ES += lz4
PKGS_ES += iptables
PKGS_ES += network-manager
PKGS_ES += iptables-persistent #(yes for saving rules, dpkg-reconfigure later?)
PKGS_ES += tshark #(yes for non-root capture, dpkg-reconfigure wireshark-common later?)
PKGS_ES += openssh-server ntp sntp ncurses-term
PKGS_ES += dnsmasq
PKGS_ES += cifs-utils smbclient samba attr samba-dsdb-modules samba-vfs-modules
PKGS_ES += keyutils winbind libnss-winbind libpam-winbind
PKGS_ES += acl
PKGS_ES += attr
PKGS_ES += bind9-host bind9-dnsutils net-tools
PKGS_ES += vlan bridge-utils ifenslave
PKGS_ES += postfix #(no config, use dpkg-reconfigre to tweak later)
PKGS_ES += libsasl2-modules
PKGS_ES += mailutils
PKGS_ES += zsh gkrellmd
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
PKGS_ES += krb5-user #(specified default realm (SOURCERUCKUS.ORG), server (krb5), and pw server (krb5pw), use dpkg-reconfigure later?)
PKGS_ES += libpam-krb5
PKGS_ES += mosh
PKGS_ES += wget
PKGS_ES += man-db wamerican whois
PKGS_ES += psmisc
PKGS_ES += fio
PKGS_ES += iperf
PKGS_ES += iperf3 #(no for run as daemon automatically, dpkg-reconfigure later? no was the default)
PKGS_ES += iotop
PKGS_ES += sdparm
PKGS_ES += sysstat
PKGS_ES += nmap
PKGS_ES += apt-file
PKGS_ES += fsarchiver
# compilation toolchain and additional development tools and libraries (~1.3G)
#
# NOTE: We normally leave all these development packages until VMM or
#       Workstation.  You only need them at this point if this is the first
#       ever bootstrap build of a new release and we need to actually compile
#       the Source Ruckus PPA packages.  If that's the case, add
#       PKGS_COMPILATION to PKGS_ES for a test build and do your thing.
#
PKGS_COMPILATION =
PKGS_COMPILATION += build-essential manpages-dev
PKGS_COMPILATION += autoconf automake autopoint m4 libtool gettext pkg-config bison flex
PKGS_COMPILATION += cmake meson ccache bin86 gdb gperf help2man markdown nasm texinfo xsltproc
PKGS_COMPILATION += yajl-tools
PKGS_COMPILATION += dwarves # needed by kernel CONFIG_DEBUG_INFO_BTF
# add these in for ruckusrd subproject deps
PKGS_COMPILATION += libkmod-dev libblkid-dev # for eudev
PKGS_COMPILATION += libaio-dev # for lvm2
PKGS_COMPILATION += liblz4-dev liblzma-dev libzstd-dev zlib1g-dev liblzo2-dev # for squashfs-tools
PKGS_COMPILATION += libssl-dev libtirpc-dev # for zfs
PKGS_COMPILATION += libacl1-dev libattr1-dev libext2fs-dev libpopt-dev libltdl-dev xxhash libxxhash-dev # for rsync
PKGS_COMPILATION += libefivar1t64 libefivar-dev libefiboot1t64 libefiboot-dev # for efibootmgr
PKGS_COMPILATION += netpbm # for groff
PKGS_COMPILATION += libbz2-dev libgcrypt20-dev # for fsarchiver
PKGS_COMPILATION += libslang2-dev # for newt
# add these for kernel compilation
PKGS_COMPILATION += libncurses-dev libelf-dev
# to build zfs deb packages
#
# NOTE: These things are required by the deb-utils target, which builds a bunch
#       of RPMs and then converts them into DEBs.  The resulting packages are
#       really only appropriate for a local apt install *.deb (because they
#       don't do ANY dependency tracking).  That's why I maintain special
#       Makefile rules (based on ruckusrd's 'dpkg' target) to build a single
#       zfs package with full dependency tracking.
#
#       In other words, we don't really need alien, and we probably don't
#       really need the two perl libs it required.  We probably don't need
#       fakeroot, either.  But we DO need the rest of them (in order to compile
#       OpenZFS later on), I haven't tested at all w/out those packages, we
#       might eventually want them, and it's not really that much more to add.
#
PKGS_COMPILATION += alien libarchive-cpio-perl libmail-sendmail-perl
PKGS_COMPILATION += libpython3-dev
#PKGS_COMPILATION += python3-distutils # no longer exists?
PKGS_COMPILATION += python3-setuptools
PKGS_COMPILATION += python3-cffi
PKGS_COMPILATION += python3-distlib python3-packaging
PKGS_COMPILATION += fakeroot
PKGS_COMPILATION += libudev-dev
# recommended at this point
PKGS_COMPILATION += libalgorithm-merge-perl libfl-dev libc6-dbg gdbserver
PKGS_COMPILATION += libfile-fcntllock-perl libwww-perl libxml-sax-expat-perl
PKGS_COMPILATION += ghostscript
PKGS_COMPILATION += libalgorithm-diff-xs-perl fonts-droid-fallback
PKGS_COMPILATION += libhtml-format-perl libpaper-utils libdata-dump-perl
PKGS_COMPILATION += libhtml-form-perl libhttp-daemon-perl libmailtools-perl
PKGS_COMPILATION += fonts-noto-mono libauthen-sasl-perl
# suggested at this point (not all of them, just a select few)
PKGS_COMPILATION += fonts-noto fonts-freefont-otf fonts-freefont-ttf
PKGS_COMPILATION += fonts-texgyre libdigest-hmac-perl
PKGS_COMPILATION += libgssapi-perl libcrypt-ssleay-perl libauthen-ntlm-perl
PKGS_COMPILATION += libxml-sax-expatxs-perl poppler-utils fonts-arphic-ukai
PKGS_COMPILATION += fonts-arphic-uming fonts-nanum fonts-noto-cjk
PKGS_COMPILATION += fonts-noto-cjk-extra fonts-noto-color-emoji fonts-noto-extra
PKGS_COMPILATION += fonts-noto-ui-core fonts-noto-ui-extra fonts-noto-unhinted
PKGS_COMPILATION += libmath-random-isaac-xs-perl
PKGS_COMPILATION += libasprintf-dev libgettextpo-dev elfutils
# to build ltfs
PKGS_COMPILATION += icu-devtools
PKGS_COMPILATION += libfuse3-dev libfuse-dev libnet-snmp-perl libcrypt-des-perl libsnmp-dev libsnmp-perl
PKGS_COMPILATION += libxml2-dev libxml2-utils libicu-dev
# to build xen
PKGS_COMPILATION += python3-dev
PKGS_COMPILATION += bcc libcurl4-openssl-dev
PKGS_COMPILATION += transfig tgif
PKGS_COMPILATION += texlive-latex-base texlive-latex-recommended
PKGS_COMPILATION += texlive-fonts-extra texlive-fonts-recommended mercurial
PKGS_COMPILATION += python3-twisted
PKGS_COMPILATION += libvncserver-dev libsdl1.2-dev libsdl2-dev libjpeg-dev
PKGS_COMPILATION += libglib2.0-dev
PKGS_COMPILATION += libnl-3-dev libnl-cli-3-dev libnl-genl-3-dev libnl-route-3-dev
PKGS_COMPILATION += libnl-idiag-3-dev libnl-xfrm-3-dev
PKGS_COMPILATION += iasl ocaml ocaml-findlib
PKGS_COMPILATION += ocamlbuild libx11-dev libyajl-dev
PKGS_COMPILATION += libpixman-1-dev pandoc figlet
PKGS_COMPILATION += libc6-dev-i386
PKGS_COMPILATION += lzma lzma-dev
PKGS_COMPILATION += libsystemd-dev
# recommends from the xen packages
PKGS_COMPILATION += gcc-multilib libdecor-0-plugin-1-gtk #default-libdecor-0-plugin-1
PKGS_COMPILATION += libgl1-amber-dri ocaml-man libtasn1-doc mesa-vulkan-drivers
PKGS_COMPILATION += libfindlib-ocaml-dev ledit python3-click lmodern dvisvgm
PKGS_COMPILATION += cm-super fonts-adf-accanthis fonts-adf-berenis fonts-adf-gillius
PKGS_COMPILATION += fonts-adf-universalis fonts-cabin fonts-cantarell fonts-clear-sans
PKGS_COMPILATION += fonts-comfortaa fonts-comic-neue fonts-croscore fonts-crosextra-caladea
PKGS_COMPILATION += fonts-crosextra-carlito fonts-dejavu-core fonts-dejavu-extra
PKGS_COMPILATION += fonts-ebgaramond-extra fonts-font-awesome fonts-gfs-artemisia
PKGS_COMPILATION += fonts-gfs-complutum fonts-gfs-didot fonts-gfs-neohellenic fonts-gfs-olga
PKGS_COMPILATION += fonts-gfs-solomos fonts-go fonts-inter fonts-lato fonts-linuxlibertine
PKGS_COMPILATION += fonts-lobstertwo fonts-oflb-asana-math fonts-open-sans fonts-paratype
PKGS_COMPILATION += fonts-roboto-slab fonts-roboto-unhinted fonts-sil-andika fonts-sil-charis
PKGS_COMPILATION += fonts-sil-gentium fonts-sil-gentium-basic fonts-sil-gentiumplus
PKGS_COMPILATION += fonts-sil-gentiumplus-compact fonts-stix texlive-fonts-extra-links
PKGS_COMPILATION += texlive-latex-extra tex-gyre tipa xfonts-75dpi libfile-mimeinfo-perl
PKGS_COMPILATION += libnet-dbus-perl libx11-protocol-perl x11-utils x11-xserver-utils
if ES_DEVEL
PKGS_ES += $(PKGS_COMPILATION)
endif

# NOTE: We want apt install to be completely noninteractive with pre-seeded
#       values for all debconf questions.  My initial idea was to run through
#       interactively and then just copy the resulting debconf config.dat into
#       subsequent runs, but reading online kinda implies that that's the
#       "wrong" thing to do, and that I should use either DEBCONF_DB_FALLBACK
#       or DEBCONF_DB_OVERRIDE variables (set to either Pipe or File) to supply
#       values...  both of those methods were providing strange errors,
#       though...  so I tried just copying config.dat in directly as I'd
#       originally planned and it worked fine... so that's what I'm doing.
#
# NOTE: While the debconf-config.dat workaround does answer the
#       questions before they pop-up, it does NOT suppress other
#       interactive things (e.g., needrestart service restart dialog).
#       So we still also supply DEBIAN_FRONTEND=noninteractive.
#
APT_INSTALL=DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends
.build/RUCKUS-ES-APT: .build/RUCKUS-ES-APT-PREP
	$(MAKE) sysroot-mount-vfs
	cp debconf-config.dat sysroot/var/cache/debconf/config.dat
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ES)
	touch $@

# NOTE: We no longer use apt-key or the legacy trusted.gpg keyring
#       because they were deprecated in jammy, and for good reason.
#       See the DEPRECATION section in apt-key(8) for details.
#
#       https://askubuntu.com/questions/1286545/what-commands-exactly-should-replace-the-deprecated-apt-key
#
sysroot/etc/apt/sources.list.d/sourceruckus.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://sourceruckus.github.io/ppa/sourceruckus.key | $(CHROOT) tee /etc/apt/keyrings/sourceruckus.asc
	echo "deb [signed-by=/etc/apt/keyrings/sourceruckus.asc] https://sourceruckus.github.io/ppa noble main" > $@

sysroot/etc/apt/sources.list.d/influxdb.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://repos.influxdata.com/influxdata-archive_compat.key | $(CHROOT) tee /etc/apt/keyrings/influxdb.asc
	echo "deb [signed-by=/etc/apt/keyrings/influxdb.asc] https://repos.influxdata.com/debian stable main" > $@

sysroot/etc/apt/sources.list.d/grafana.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://packages.grafana.com/gpg.key | $(CHROOT) tee /etc/apt/keyrings/grafana.asc
	echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" > $@

sysroot/etc/apt/sources.list.d/wine.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://dl.winehq.org/wine-builds/winehq.key | $(CHROOT) tee /etc/apt/keyrings/wine.asc
	echo "deb [signed-by=/etc/apt/keyrings/wine.asc] https://dl.winehq.org/wine-builds/ubuntu noble main" > $@

# NOTE: We do this instead of using add-apt-repository to make things a bit
#	more consistent in here and to avoid multiple apt update calls.
#
# FIXME: this is going to result in DEPRECATION warnings about
#        apt-key... pretty sure i can pull the keys out after doing
#        this and store them as files, but how do i just do that to
#        begin with...?
#
# gpg --no-default-keyring --keyring /tmp/keybox.gpg --keyserver keyserver.ubuntu.com --recv-keys CC1B5A793C04BB3905AD837734893610CEAA9512
# gpg --no-default-keyring --keyring /tmp/keybox.gpg --export > ./linbit.gpg
#
# FIXME: we should really be putting all these keys in
#        /etc/apt/keyrings and then putting [signed-by=...] in each
#        sources file.
#
# see https://askubuntu.com/questions/1286545/what-commands-exactly-should-replace-the-deprecated-apt-key
#
sysroot/etc/apt/sources.list.d/linbit-drbd9.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	$(MKDIR_P) --mode=700 sysroot/root/.gnupg
	$(CHROOT) gpgconf --launch gpg-agent
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --keyserver keyserver.ubuntu.com \
	  --recv-keys CC1B5A793C04BB3905AD837734893610CEAA9512
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --export > /etc/apt/keyrings/linbit.gpg
	$(CHROOT) gpgconf --kill all
	rm sysroot/tmp/keybox.gpg
	echo "deb [signed-by=/etc/apt/keyrings/linbit.gpg] http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu noble main" > $@

sysroot/etc/apt/sources.list.d/ubuntu-gis.list: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	$(MKDIR_P) --mode=700 sysroot/root/.gnupg
	$(CHROOT) gpgconf --launch gpg-agent
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --keyserver keyserver.ubuntu.com \
	    --recv 6B827C12C2D425E227EDCA75089EBE08314DF160
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --export > /etc/apt/keyrings/ubuntugis.gpg
	$(CHROOT) gpgconf --kill all
	rm sysroot/tmp/keybox.gpg
	echo "deb [signed-by=/etc/apt/keyrings/ubuntugis.gpg] https://ppa.launchpadcontent.net/ubuntugis/ppa/ubuntu noble main " > $@

# NOTE: Definition of "3rd Party" in here is "upstream != Ubuntu".  That means
#       our own Source Ruckus PPA is considered "3rd Party".
#
# NOTE: Regarding the chicken-and-the-egg of the Source Ruckus PPA while
#       preping a distro-upgrade...  The easiest thing to do is manually take a
#       detour and build required packages by hand instead of somehow detecting
#       that need and automagically doing it via the build system.  I'm gonna
#       have to re-investigate required packages and stuff by hand, there's no
#       point in the added complexity required to "automagically" build things
#       instead of installing.  So, after a 'make .build/RUCKUS-ES-3RDPARTY' (which
#       should fail because missing packages), manually chroot into the
#       resulting system, build packages and upload them to our PPA.
#
# NOTE: We prep the PPAs for the ES-file and ES-web additions now, so that we
#       can easily apt install things for testing and corner-cases
#       (e.g. combined file/web server).
#
# NOTE: If --enable-devel, we're building es-devel.sqsh, leave PKGS_ES_3RDPARTY
#       empty (because they'll probably/definately fail)
#
PKGS_ES_3RDPARTY =
if !ES_DEVEL
PKGS_ES_3RDPARTY += telegraf #(135M! didn't realize it was that large)
PKGS_ES_3RDPARTY += pingenator
PKGS_ES_3RDPARTY += doit
PKGS_ES_3RDPARTY += extractenator
endif

# NOTE: For --enable-devel, we need to empty this list out as well.  We need to
#       do these by hand during the bootstrap process and update the rules as
#       we go.  The Source Ruckus PPA could just be left out, but in all
#       likelyhood at least one more of these will be invalid on our first
#       pass.
#
# NOTE: This is defined as an additional rule (instead of just adding more
#       depends to RUCKUS-ES-3RDPARTY) to silence some automake warnings.
#
.PHONY: PPAS_ES
if !ES_DEVEL
PPAS_ES: sysroot/etc/apt/sources.list.d/sourceruckus.list
PPAS_ES: sysroot/etc/apt/sources.list.d/influxdb.list
PPAS_ES: sysroot/etc/apt/sources.list.d/grafana.list
PPAS_ES: sysroot/etc/apt/sources.list.d/linbit-drbd9.list
PPAS_ES: sysroot/etc/apt/sources.list.d/ubuntu-gis.list
PPAS_ES: sysroot/etc/apt/sources.list.d/wine.list
endif

.build/RUCKUS-ES-3RDPARTY: .build/RUCKUS-ES-APT
	$(MAKE) sysroot-mount-vfs
	if [ -n "$(PKGS_ES_3RDPARTY)" ]; then \
	  $(MAKE) PPAS_ES; \
	  $(CHROOT) apt update; \
	  cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ES_3RDPARTY); \
	fi
	touch $@

# the finalization step - set default boot target, disable some services, tweak
# some files, etc.
#
# NOTE: We clone awesome-powers and prep it's submodules in here, so that it's
#       available if we're installing w/out an internet connection later on.
#
# NOTE: We do NOT create local users here.  That's handled by awesome-powers
#       during installation/groom.
#
# NOTE: We create globally-managed-devices.conf because all interfaces end up
#       "unmanaged" in NetworkManager w/out it.  (This was definately needed
#       for focal, but I forgot to check if it's still needed with
#       jammy... doesn't hurt anything though)
#
# NOTE: We disable and mask the RPC security service for NFS here, because upon
#       joining an AD domain, even if you don't have nfs-kernel-server
#       installed, this service autostarts and fails.
#
# FIXME: should really move awesome-powers to /opt
#
.build/RUCKUS-ES-FINAL: .build/RUCKUS-ES-3RDPARTY
	$(MAKE) sysroot-mount-vfs
	echo "rootfs / auto defaults 0 0" > sysroot/etc/fstab
	mkdir -p sysroot/usr/lib/modules
	mkdir -p sysroot/usr/lib/firmware
	mkdir -p sysroot/usr/src
	touch sysroot/etc/NetworkManager/conf.d/10-globally-managed-devices.conf
	$(MKDIR_P) sysroot/scrap && cd sysroot/scrap && \
	    git clone https://github.com/sourceruckus/awesome-powers.git && \
	    cd awesome-powers && ./autogen.sh
	$(CHROOT) apt clean
	$(CHROOT) systemctl set-default multi-user.target
	$(CHROOT) systemctl disable dnsmasq
	$(CHROOT) systemctl disable rpc-svcgssd.service
	$(CHROOT) systemctl mask rpc-svcgssd.service
	touch $@

# FIXME: migrate to -comp zstd.  Ubuntu 18.04's squashfs doesn't support it,
#        but we could always compile our own in /usr/local or something. (or
#        provide a newer bionic deb via source ruckus ppa)
#
#        i can't believe it's 2024 and i'm still running bionic on my
#        build box... need to prioritize upgrading that.  with lz4
#        compression, es-devel.sqsh is 1.3G and the uncompressed
#        contents of the sysroot dir after building is, wait for it,
#        1.3G.  lame.  oh wait, the actual size on disk is more than
#        twice that, but it's being compressed transparently by zfs.
#        that means all the size measurements i've got in here are
#        COMPRESSED sizes.
#
# NOTE: If --enable-devel, just swap out the name of this rule.
#
if ES_DEVEL
es-devel.sqsh: .build/RUCKUS-ES-FINAL
else
es.sqsh: .build/RUCKUS-ES-FINAL
endif
	$(MAKE) sysroot-umount-vfs
	mksquashfs sysroot $@ -comp lz4 -Xhc -noappend -wildcards

sysroot-clean:
	$(MAKE) sysroot-umount-vfs
	umount -l sqsh_layer-vmm || echo not mounted
	rm -rf sqsh_layer-vmm
	umount -l sqsh_layer-es-web || echo not mounted
	rm -rf sqsh_layer-es-web
	umount -l sqsh_layer-es-file || echo not mounted
	rm -rf sqsh_layer-es-file
	umount -l sqsh_layer-es || echo not mounted
	rm -rf sqsh_layer-es
	umount -l upper_layer || echo not mounted
	rm -rf upper_layer
	umount -l sysroot || echo not mounted
	rm -rf sysroot

clean-local: sysroot-clean
	rm -rf .build



################################################################################
# ES-file
#
# Based on ES, adds file-server specific packages.
#
# NOTE: This image includes upstream Wine (including full 32bit library
#       support) so you can run Windows junk.  For example, at the vet hospital
#       we run AVImarkServer.
#
PKGS_ESFILE =
PKGS_ESFILE += nfs-kernel-server
PKGS_ESFILE += drbd-utils # from previously added upstream ppa
PKGS_ZFS =
PKGS_ZFS += zfs # our package, from sourceruckus ppa
PKGS_ZFS += zfs-auto-snapshot # ubuntu ppa, but now using our zfs package
# deps for sanoid, so we can use a development snapshot if desired
PKGS_ZFS += pv lzop mbuffer libcapture-tiny-perl libconfig-inifiles-perl
PKGS_ZFS += sanoid # ubuntu ppa, but now using our zfs package
PKGS_ESFILE += $(PKGS_ZFS)
PKGS_ESFILE += minidlna #FIXME: jammy already installed?:  libaacs0 libgdk-pixbuf2.0-bin librsvg2-common libbdplus0
PKGS_ESFILE += wsdd
# FIXME: is the orthanc stuff going to need apache2?
PKGS_ESFILE += orthanc
PKGS_ESFILE += orthanc-dicomweb
PKGS_ESFILE += orthanc-doc javascript-common
#PKGS_ESFILE += orthanc-imagej mesa-vulkan-drivers libatk-wrapper-java-jni fonts-dejavu-extra
#PKGS_ESFILE += orthanc-mysql
#PKGS_ESFILE += orthanc-postgresql
PKGS_ESFILE += orthanc-webviewer
PKGS_ESFILE += orthanc-wsi libopenslide0
PKGS_ESFILE += gmt gmt-dcw gmt-gshhg proj-bin #+315M
#PKGS_ESFILE += mbsystem # FIXME: durp, UbuntuGIS dropped mbsystem in focal?

# prepare an overlay mount on sysroot
#
# FIXME: my build box is using zfs, which can't be used for the upper layer.
#        oopsie.  for now lets use a 2G tmpfs mount and see how that goes.
#        long term, we should check for an appropriate fs via configure and add
#        --with-tmpfs-upper=SIZE argument.
#
# FIXME: shoot, making this depend on es.sqsh seems to be messing things up.
#        if sqsh_layer-es already exists, make is still executing this
#        rule... did i dork up a timestamp or something?
#
sqsh_layer-es:
	$(MKDIR_P) $@
	mount es.sqsh $@

upper_layer:
	$(MKDIR_P) $@
	mount -t tmpfs none $@

.build/RUCKUS-ESFILE-CLEAN:
	$(MAKE) sysroot-clean
	touch $@

.build/RUCKUS-ESFILE-OVL: .build/RUCKUS-ESFILE-CLEAN sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-ESFILE-APT: .build/RUCKUS-ESFILE-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ESFILE)
	touch $@

# add 32bit architecture support
.build/RUCKUS-ESFILE-32bit: .build/RUCKUS-ESFILE-APT
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) dpkg --add-architecture i386
	$(CHROOT) apt update
	touch $@

# special install block for wine because we ONLY want to specify
# --install-recommends for wine
.build/RUCKUS-ESFILE-WINE: .build/RUCKUS-ESFILE-32bit
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) --install-recommends winehq-stable
	touch $@

# service enable/dislabe, tweaks, etc
#
# FIXME: zfs services aren't enabled by default... did i do that on purpose?
#
# FIXME: audit services
#
.build/RUCKUS-ESFILE-FINAL: .build/RUCKUS-ESFILE-WINE
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) systemctl enable zfs-import-cache.service zfs-import.target zfs-mount.service \
	  zfs-share.service zfs-volume-wait.service zfs-volumes.target zfs-zed.service \
	  zfs.target
	touch $@

# mksquashfs
es-file.sqsh: es.sqsh .build/RUCKUS-ESFILE-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ -comp lz4 -Xhc -noappend -wildcards


################################################################################
# ES-web
#
# Based on ES, not ES-file, and adds web-server stuff.
#
PKGS_ESWEB =
PKGS_ESWEB += apache2 ssl-cert
PKGS_ESWEB += javascript-common
PKGS_ESWEB += influxdb2 influxdb2-cli # upstream ppa added already
PKGS_ESWEB += grafana # upstream ppa added already
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
# sudo /bin/systemctl daemon-reload
# sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
# sudo /bin/systemctl start grafana-server
#
# FIXME: didn't mongodb get dropped from ubuntu?  looks like unifi
#        delivers its own mongodb, too
#PKGS_ESWEB += mongodb-server mongo-tools # for unifi

# FIXME: Considered adding the unifi deb to my ppa... but I'm pretty sure it
#        adds it's own ppa as upstream for updates, so that's probably a bad
#        idea.  Instead, we'll have a list of local .debs to install (and rules
#        to download them via wget).
#
PKGS_ESWEB_LOCAL =
PKGS_ESWEB_LOCAL += unifi
#PKGS_ESWEB_LOCAL += chrome

sysroot/tmp/unifi.deb:
	wget -qO- https://dl.ui.com/unifi/7.1.66/unifi_sysvinit_all.deb > $@

sysroot/tmp/chrome.deb:
	wget -qO- https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb > $@

.build/RUCKUS-ESWEB-CLEAN:
	$(MAKE) sysroot-clean
	touch $@

.build/RUCKUS-ESWEB-OVL: .build/RUCKUS-ESWEB-CLEAN sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-ESWEB-APT: .build/RUCKUS-ESWEB-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ESWEB)
	touch $@

# fetch debs and install locally
.build/RUCKUS-ESWEB-LOCAL: .build/RUCKUS-ESWEB-APT sysroot/tmp/unifi.deb
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(patsubst %,/tmp/%.deb,$(PKGS_ESWEB_LOCAL))
	touch $@

# service enable/disable, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-ESWEB-FINAL: .build/RUCKUS-ESWEB-LOCAL
	$(MAKE) sysroot-mount-vfs
	touch $@

es-web.sqsh: .build/RUCKUS-ESWEB-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ -comp lz4 -Xhc -noappend -wildcards


################################################################################
# VMM (Xen Dom0)
#
# Based on ES, not ES-file or ES-web, adds things needed for our VMM setup.
# This is the first layer that is intended for use on physical hardware (i.e.,
# not a virtual machine), so it's where we start to add in bootloader and more
# hardware-centric things.
#
# - apt
#   - efibootmgr
#   - syslinux extlinux isolinux
#   - ntfs3g?
#   - fwupd
#   - dkms? (probably wait until we have compiler for that)
#   - smartmontools
#   - drbd9 (should I wait until here and use upstream ppa?)
# - compile/apt
#   - ruckusrd (if we haven't already)
#   - xen
PKGS_VMM =
PKGS_VMM += drbd-utils # from previously added upstream ppa
PKGS_VMM += $(PKGS_ZFS)
PKGS_VMM += ruckusrd
PKGS_VMM += firmwarenator
PKGS_VMM += microcodenator
PKGS_VMM += syslinux syslinux-common dosfstools extlinux isolinux syslinux-efi syslinux-utils
PKGS_VMM += efibootmgr
PKGS_VMM += efitools efivar
PKGS_VMM += fwupd fwupd-signed
PKGS_VMM += smartmontools
PKGS_VMM += lm-sensors fancontrol read-edid i2c-tools
PKGS_VMM += xen

.build/RUCKUS-VMM-CLEAN:
	$(MAKE) sysroot-clean
	touch $@

.build/RUCKUS-VMM-OVL: .build/RUCKUS-VMM-CLEAN sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-VMM-APT: .build/RUCKUS-VMM-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_VMM)
	touch $@

# service enable/disable, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-VMM-FINAL: .build/RUCKUS-VMM-APT
	$(MAKE) sysroot-mount-vfs
	touch $@

vmm.sqsh: .build/RUCKUS-VMM-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ -comp lz4 -Xhc -noappend -wildcards

# special target for adding compilation toolchain to an already
# prepped sysroot
sysroot-add-compilation:
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_COMPILATION)


################################################################################
# Workstation
#
# Based off VMM (which exludes ES-file and ES-web additions), adds everything
# else we want in an awesome workstation of doom (including the kitchen sink).
#
# NOTE: Similar to ES-file, we want a full multilib setup with upstream Wine for
#       the Workstation.
#
# - 32bit update
# - via apt
#   - gcc-multilib (?)
#   - ubuntu-desktop (maybe exclude a few pkgs?)
#   - jackd?
#   - vlc (via upstream snap?)
#   - dvd player stuff (maybe?)
#   - special font packages (maybe?)
#   - rdesktop (and more?)
#   - libglvnd-dev (what did i need this for...?)
#   - wine (upstream ppa?)
# - 3rd party stuff
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
.build/RUCKUS-WORKSTATION-3RDPARTY: sysroot/etc/apt/sources.list.d/qgis.list # FIXME: pure upstream or UbuntuGIS?
.build/RUCKUS-WORKSTATION-3RDPARTY: sysroot/etc/apt/sources.list.d/steam.list

PKGS_WORKSTATION =
# things we want from esfile
PKGS_WORKSTATION += nfs-kernel-server
PKGS_WORKSTATION += wsdd
PKGS_WORKSTATION += gmt gmt-dcw gmt-gshhg proj-bin
#PKGS_WORKSTATION += mbsystem # FIXME: durp, UbuntuGIS dropped mbsystem in focal?
# development stuff
PKGS_WORKSTATION += $(PKGS_COMPILATION)
PKGS_WORKSTATION += kernel-builder
PKGS_WORKSTATION += gcc-multilib g++-multilib gfortran-multilib
# everything else we want
PKGS_WORKSTATION += gimp gimp-help-en gimp-data-extras gsfonts libbluray-bdj mesa-opencl-icd libvdpau-va-gl1
PKGS_WORKSTATION += groff ghostscript imagemagick libpaper1 netpbm psutils
PKGS_WORKSTATION += ffmpeg
PKGS_WORKSTATION += gnuplot
PKGS_WORKSTATION += inkscape xfig dia
PKGS_WORKSTATION += audacity audacious
PKGS_WORKSTATION += opus-tools librsvg2-bin speex vorbis-tools sox

PKGS_WORKSTATION += va-driver-all vdpau-driver-all mesa-utils
#PKGS_WORKSTATION += mesa-vulkan-drivers # jammy already installed

PKGS_WORKSTATION += wireshark gkrellm gitk git-gui
PKGS_WORKSTATION += vlc
PKGS_WORKSTATION += gnome-tweaks rdesktop
PKGS_WORKSTATION += emacs-gtk # replaces emacs-nox
PKGS_WORKSTATION += wine


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
#   - useradd -c "Local Administrator" -G adm,sudo,cdrom,dip,plugdev,lpadmin,lxd,sambashare -m -s /bin/zsh localadmin
#
#
# - create new host.sqsh from overlay mount (perhaps?)
#
# NOTE: time-out... i think the installer might just end up living in
#       awesome-powers...
