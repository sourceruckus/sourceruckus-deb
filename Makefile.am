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
# FIXME: actually, i did start using tmpfs for upper_layer at some
#        point already... see other FIXMEs
#
CHROOT=chroot sysroot /usr/bin/env -i HOME=/root TERM=$$TERM PATH=$$PATH


# NOTE: If --enable-devel was sepcified, we're building a devel-enabled base
#       image w/out 3rd party packages.
all: @IMG_LIST@


info:
	@echo DEBOOTSTRAP=$(DEBOOTSTRAP)
	@echo
	@echo PKGS_ES=\"$(PKGS_ES)\"
	@echo
	@echo PKGS_ES_3RDPARTY=\"$(PKGS_ES_3RDPARTY)\"
	@echo
	@echo PKGS_ESFILE=\"$(PKGS_ESFILE)\"
	@echo
	@echo PKGS_ESAUTH=\"$(PKGS_ESAUTH)\"
	@echo
	@echo PKGS_ESWEB=\"$(PKGS_ESWEB)\"
	@echo
	@echo PKGS_SERVER=\"$(PKGS_SERVER)\"
	@echo
	@echo PKGS_VMM=\"$(PKGS_VMM)\"
	@echo
	@echo PKGS_WORKSTATION=\"$(PKGS_WORKSTATION)\"
	@echo
	@echo PKGS_WORKSTATION_LOCAL\"$(PKGS_WORKSTATION_LOCAL)\"
	@echo
	@echo PKGS_COMPILATION=\"$(PKGS_COMPILATION)\"
	@echo
	@echo PKGS_ZFS=\"$(PKGS_ZFS)\"
	@echo
	@echo PKGS_FILESHARING=\"$(PKGS_FILESHARING)\"

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
PKGS_ES += bc btrfs-progs cpio cryptsetup cryptsetup-bin curl
PKGS_ES += dirmngr dmeventd dosfstools
PKGS_ES += ethtool fdisk finalrd fuse3 gawk gdisk gettext-base gir1.2-packagekitglib-1.0 git git-man gnupg
PKGS_ES += gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpgconf gpgsm groff-base htop jq keyboxd
PKGS_ES += kpartx linux-base lshw lvm2
PKGS_ES += mdadm modemmanager motd-news-config needrestart ntfs-3g openssh-client
PKGS_ES += packagekit packagekit-tools parted patch pci.ids pciutils perl perl-modules-5.38 pinentry-curses
PKGS_ES += polkitd pollinate publicsuffix
PKGS_ES += run-one sgml-base software-properties-common
PKGS_ES += squashfs-tools tcl tcl8.6 thin-provisioning-tools ubuntu-release-upgrader-core unattended-upgrades
PKGS_ES += usb-modeswitch usb-modeswitch-data usb.ids vim vim-runtime xauth xfsprogs xml-core xz-utils zerofree zstd
# additional things we want (~730M last time I checked)
PKGS_ES += lz4
PKGS_ES += iptables
PKGS_ES += network-manager
PKGS_ES += iptables-persistent #(yes for saving rules, dpkg-reconfigure later?)
PKGS_ES += tshark #(yes for non-root capture, dpkg-reconfigure wireshark-common later?)
PKGS_ES += openssh-server ntp sntp ncurses-term
PKGS_ES += dnsmasq
PKGS_ES += cifs-utils smbclient
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
PKGS_ES += logwatch libdate-manip-perl
PKGS_ES += lsof
PKGS_ES += rsync
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
PKGS_ES += sysstat
PKGS_ES += nmap
PKGS_ES += apt-file
PKGS_ES += fsarchiver
PKGS_ES += bzip2
PKGS_ES += manpages

# FIXME: would add nftables so I can play with it, but afraid it will
#        mess with iptables
#
# nftables iptables-nftables-compat

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
#
# NOTE: Some of these may not technically be needed for building xen,
#       but they were in my notes as being installed the last time I
#       did this.
#
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
# to build mbsystem (generic compilation toolchain stuff)
#
# NOTE: Some of these may not technically be needed for building mbsystem,
#       but they were in my notes as being installed the last time I
#       did this.
#
PKGS_COMPILATION += autoconf-archive diffstat gfortran
PKGS_COMPILATION += gv libdevel-globaldestruction-perl
PKGS_COMPILATION += libfile-remove-perl libgfortran5 libhash-ordered-perl
PKGS_COMPILATION += libmail-box-perl libmail-message-perl libmail-transport-perl
PKGS_COMPILATION += libmime-types-perl libobject-realize-later-perl
PKGS_COMPILATION += libsub-exporter-progressive-perl libuser-identity-perl xaw3dg
# to build mbsystem (xorg stuff)
#
# NOTE: Some of these may not technically be needed for building mbsystem,
#       but they were in my notes as being installed the last time I
#       did this.
#
PKGS_COMPILATION += xorg-dev mesa-common-dev libmotif-common libmotif-dev
PKGS_COMPILATION += libsdl-image1.2-dev libsdl-mixer1.2-dev libsdl-ttf2.0-dev libsdl2-gfx-dev
PKGS_COMPILATION += libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev
PKGS_COMPILATION += libgirepository1.0-dev
PKGS_COMPILATION += python3-pygments libpipewire-0.3-common
# to build mbsystem (gis stuff, some from ubuntu-gis ppa)
#
# FIXME: we need to actually prep the ubuntu-gis ppa before this will
#        work...
#
# FIXME: that's what my notes said, but it doesn't seem like any of
#        these are coming from the Ubuntu-GIS PPA...
#
PKGS_COMPILATION += gmt geotiff-bin gdal-bin libgeotiff-dev
PKGS_COMPILATION += odbc-postgresql tdsodbc ogdi-bin
PKGS_COMPILATION += gmt-gshhg libgmt-dev libgdal-grass libgnutls28-dev
PKGS_COMPILATION += libidn11-dev libkrb5-dev libldap2-dev librtmp-dev libssh2-1-dev
PKGS_COMPILATION += netcdf-bin libfftw3-bin libfftw3-dev # libfftw3-3 # not in noble?
PKGS_COMPILATION += libfftw3-long3 libfftw3-quad3 libfftw3-double3 libfftw3-mpi-dev
PKGS_COMPILATION += libfftw3-mpi3 libfftw3-single3 libproj-dev proj-bin
PKGS_COMPILATION += libhdf4-alt-dev hdf4-tools
PKGS_COMPILATION += grass-dev grass-gui e00compr avce00 gnuplot gpsbabel
PKGS_COMPILATION += gpstrans python3-rpy2 python3-termcolor grass krb5-k5tls
PKGS_COMPILATION += libmpfr-dev gnutls-bin libhwloc-contrib-plugins idn
PKGS_COMPILATION += setserial libitpp-dev elpa-ess libparallel-forkmanager-perl gmt-dcw
PKGS_COMPILATION += subversion python3-matplotlib libnamespace-clean-perl qttranslations5-l10n
PKGS_COMPILATION += libpackage-stash-xs-perl qt5-gtk-platformtheme libwacom-bin python3-bs4
PKGS_COMPILATION += python3-html5lib python3-chardet python3-olefile r-recommended
PKGS_COMPILATION += r-base-dev

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
sysroot/etc/apt/sources.list.d/sourceruckus.list:
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://sourceruckus.github.io/ppa/sourceruckus.key | $(CHROOT) tee /etc/apt/keyrings/sourceruckus.asc
	echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/sourceruckus.asc] https://sourceruckus.github.io/ppa noble main" > $@

sysroot/etc/apt/sources.list.d/influxdb.list:
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://repos.influxdata.com/influxdata-archive_compat.key | $(CHROOT) tee /etc/apt/keyrings/influxdb.asc
	echo "deb [signed-by=/etc/apt/keyrings/influxdb.asc] https://repos.influxdata.com/debian stable main" > $@

sysroot/etc/apt/sources.list.d/grafana.list:
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://packages.grafana.com/gpg.key | $(CHROOT) tee /etc/apt/keyrings/grafana.asc
	echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" > $@

sysroot/etc/apt/sources.list.d/wine.list:
	$(MAKE) sysroot-mount-vfs
	wget -qO- https://dl.winehq.org/wine-builds/winehq.key | $(CHROOT) tee /etc/apt/keyrings/wine.asc
	echo "deb [signed-by=/etc/apt/keyrings/wine.asc] https://dl.winehq.org/wine-builds/ubuntu noble main" > $@

# NOTE: We do this instead of using add-apt-repository to make things a bit
#	more consistent in here and to avoid multiple apt update calls.
sysroot/etc/apt/sources.list.d/linbit-drbd9.list:
	$(MAKE) sysroot-mount-vfs
	$(MKDIR_P) --mode=700 sysroot/root/.gnupg
	$(CHROOT) gpgconf --launch gpg-agent
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --keyserver keyserver.ubuntu.com \
	  --recv-keys CC1B5A793C04BB3905AD837734893610CEAA9512
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --export > sysroot/etc/apt/keyrings/linbit.gpg
	$(CHROOT) gpgconf --kill all
	rm sysroot/tmp/keybox.gpg
	echo "deb [signed-by=/etc/apt/keyrings/linbit.gpg] http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu noble main" > $@

# FIXME: do i really need this one?  way back we got mbsystem from
#        here, along with other things like gmt, i thought...  but
#        after bionic (i think), mbsystem got dropped from Ubuntu-GIS
#        and I started building it myself with jammy... but by that
#        time i think all it's deps were in regular ubuntu...?
#
#        i think it still provides *newer* versions of packages,
#        including qgis
#
sysroot/etc/apt/sources.list.d/ubuntu-gis.list:
	$(MAKE) sysroot-mount-vfs
	$(MKDIR_P) --mode=700 sysroot/root/.gnupg
	$(CHROOT) gpgconf --launch gpg-agent
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --keyserver keyserver.ubuntu.com \
	    --recv 6B827C12C2D425E227EDCA75089EBE08314DF160
	$(CHROOT) gpg --no-default-keyring --keyring /tmp/keybox.gpg --export > /sysroot/etc/apt/keyrings/ubuntugis.gpg
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
	rm -fv sysroot/tmp/*
	touch $@


# FIXME: configure flag for this?  or maybe have configure check for
#        zstd support in mksquashfs?
#
#SQSH_COMP=-comp lz4 -Xhc
SQSH_COMP=-comp zstd

# NOTE: If --enable-devel, just swap out the name of this rule.
#
if ES_DEVEL
es-devel.sqsh: .build/RUCKUS-ES-FINAL
else
es.sqsh: .build/RUCKUS-ES-FINAL
endif
	$(MAKE) sysroot-umount-vfs
	mksquashfs sysroot $@ $(SQSH_COMP) -noappend -wildcards

sysroot-clean:
	$(MAKE) sysroot-umount-vfs
	umount -l sqsh_layer-vmm || echo not mounted
	rm -rf sqsh_layer-vmm
	umount -l sqsh_layer-server || echo not mounted
	rm -rf sqsh_layer-server
	umount -l sqsh_layer-es-web || echo not mounted
	rm -rf sqsh_layer-es-web
	umount -l sqsh_layer-es-auth || echo not mounted
	rm -rf sqsh_layer-es-auth
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
PKGS_FILESHARING =
PKGS_FILESHARING += nfs-kernel-server
PKGS_FILESHARING += samba samba-dsdb-modules samba-vfs-modules
PKGS_FILESHARING += wsdd

PKGS_ZFS =
PKGS_ZFS += zfs # our package, from sourceruckus ppa
PKGS_ZFS += zfs-auto-snapshot # ubuntu ppa, but now using our zfs package
# deps for sanoid, so we can use a development snapshot if desired
PKGS_ZFS += pv lzop mbuffer libcapture-tiny-perl libconfig-inifiles-perl
PKGS_ZFS += sanoid # ubuntu ppa, but now using our zfs package

PKGS_ESFILE =
PKGS_ESFILE += $(PKGS_FILESHARING)
PKGS_ESFILE += minidlna libaacs0 libgdk-pixbuf2.0-bin pocketsphinx-en-us librsvg2-common libbdplus0
# FIXME: at this rate, i'm probably never going to play with
#        orthanc... so commenting it out
#
# FIXME: is the orthanc stuff going to need apache2?
#PKGS_ESFILE += orthanc
#PKGS_ESFILE += orthanc-dicomweb
#PKGS_ESFILE += orthanc-doc javascript-common
##PKGS_ESFILE += orthanc-imagej mesa-vulkan-drivers libatk-wrapper-java-jni fonts-dejavu-extra
##PKGS_ESFILE += orthanc-mysql
##PKGS_ESFILE += orthanc-postgresql
#PKGS_ESFILE += orthanc-webviewer
#PKGS_ESFILE += orthanc-wsi libopenslide0


# prepare an overlay mount on sysroot
#
# FIXME: my build box is using zfs, which can't be used for the upper layer.
#        oopsie.  for now lets use a 2G tmpfs mount and see how that goes.
#        long term, we should check for an appropriate fs via configure and add
#        --with-tmpfs-upper=SIZE argument.
#
# FIXME: ok, i think that's fixed in upstream ZFS 2.2.something,
#        definately in 2.3.  for the record, using tmpfs for all the
#        images prior to workstation.sqsh works fine with 4G of ram
#        (defaults to 2G tmpfs)... but the workstation needs ~11.5G of
#        freespace... so yeah, RAM probably not good for that.
#
# FIXME: oooh... building on a vm, my build dir is *already* an
#        overlay...  crap bag.  I'll either have to pass in a data vol
#        or make an sparse ext4 image...  probably better to pass in a
#        data vol.  :-/
#
sqsh_layer-es: es.sqsh
	$(MKDIR_P) $@
	mount es.sqsh $@

upper_layer:
	$(MKDIR_P) $@
#	mount -t tmpfs none $@

.build/RUCKUS-ESFILE-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-ESFILE-APT: .build/RUCKUS-ESFILE-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ESFILE)
	touch $@

# service enable/dislabe, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-ESFILE-FINAL: .build/RUCKUS-ESFILE-APT
	$(MAKE) sysroot-mount-vfs
	rm -fv sysroot/tmp/*
	touch $@

# mksquashfs
es-file.sqsh: .build/RUCKUS-ESFILE-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards


################################################################################
# ES-auth
#
# Based on ES, not ES-file, and adds auth-server stuff.
#
# NOTE: Currently just samba, for samba-active-directory, but we could
#       branch out and try other products.
#
PKGS_ESAUTH =
PKGS_ESAUTH += samba
PKGS_ESAUTH += python3-idna samba-dsdb-modules samba-vfs-modules samba-ad-provision

.build/RUCKUS-ESAUTH-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-ESAUTH-APT: .build/RUCKUS-ESAUTH-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ESAUTH)
	touch $@

# service enable/disable, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-ESAUTH-FINAL: .build/RUCKUS-ESAUTH-APT
	$(MAKE) sysroot-mount-vfs
	rm -fv sysroot/tmp/*
	touch $@

es-auth.sqsh: .build/RUCKUS-ESAUTH-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards


################################################################################
# ES-web
#
# Based on ES, not ES-file, and adds web-server stuff.
#
PKGS_ESWEB =
PKGS_ESWEB += apache2 ssl-cert
PKGS_ESWEB += javascript-common
PKGS_ESWEB += influxdb #  upstream ppa added already
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

# NOTE: I considered adding the unifi deb to my ppa... but I'm pretty
#       sure it adds it's own ppa as upstream for updates, so that's
#       probably a bad idea.  Instead, we'll have a list of local
#       .debs to install (and rules to download them via wget).
#
# FIXME: ubiquiti now has it's own ppa, yay
#
# FIXME: it also needs us to add the mongodb ppa, though.
#
# https://help.ui.com/hc/en-us/articles/220066768-Updating-and-Installing-Self-Hosted-UniFi-Network-Servers-Linux
#
# https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/
#
# https://pcmasterx.com/index.php/blog/unifi-network-controller-install-on-ubuntu-server-24-04
#
#PKGS_ESWEB_LOCAL =
#PKGS_ESWEB_LOCAL += unifi

#sysroot/tmp/unifi.deb:
#	wget -qO- https://dl.ui.com/unifi/7.1.66/unifi_sysvinit_all.deb > $@

.build/RUCKUS-ESWEB-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-ESWEB-APT: .build/RUCKUS-ESWEB-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_ESWEB)
	touch $@

# service enable/disable, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-ESWEB-FINAL: .build/RUCKUS-ESWEB-APT
	$(MAKE) sysroot-mount-vfs
	rm -fv sysroot/tmp/*
	touch $@

es-web.sqsh: .build/RUCKUS-ESWEB-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards


################################################################################
# Server
#
# Based on ES, not ES-file, ES-web, or ES-auth, this is the first
# layer that is intended for use on physical hardware (i.e., not a
# virtual machine), so it's where we start to add in bootloader and
# more hardware-centric things.
#
PKGS_SERVER =
PKGS_SERVER += drbd-utils # from previously added upstream ppa
PKGS_SERVER += $(PKGS_ZFS)
PKGS_SERVER += $(PKGS_FILESHARING)
PKGS_SERVER += ruckusrd
PKGS_SERVER += firmwarenator
PKGS_SERVER += microcodenator
PKGS_SERVER += syslinux syslinux-common dosfstools extlinux isolinux syslinux-efi syslinux-utils
PKGS_SERVER += efibootmgr
PKGS_SERVER += efitools efivar
PKGS_SERVER += secureboot-db
PKGS_SERVER += powermgmt-base
PKGS_SERVER += fwupd fwupd-signed
PKGS_SERVER += smartmontools
PKGS_SERVER += hdparm
PKGS_SERVER += sdparm
PKGS_SERVER += rfkill
PKGS_SERVER += bolt
PKGS_SERVER += dmidecode
PKGS_SERVER += open-iscsi
PKGS_SERVER += udisks2
PKGS_SERVER += lm-sensors fancontrol read-edid i2c-tools
PKGS_SERVER += ltfs

.build/RUCKUS-SERVER-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

# apt install
.build/RUCKUS-SERVER-APT: .build/RUCKUS-SERVER-OVL
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_SERVER)
	touch $@

# service enable/dislabe, tweaks, etc
#
# FIXME: zfs services aren't enabled by default... did i do that on purpose?
#
# FIXME: audit services
#
.build/RUCKUS-SERVER-FINAL: .build/RUCKUS-SERVER-APT
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) systemctl enable zfs-import-cache.service zfs-import.target zfs-mount.service \
	  zfs-share.service zfs-volume-wait.service zfs-volumes.target zfs-zed.service \
	  zfs.target
	rm -fv sysroot/tmp/*
	touch $@

server.sqsh: .build/RUCKUS-SERVER-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards


################################################################################
# VMM (Xen Dom0)
#
# Based on Server, adds things needed for our VMM setup.
#
# NOTE: Currently just xen, but we could add other products here later.
#
PKGS_VMM =
PKGS_VMM += xen
PKGS_VMM += qemu-system-x86 grub-xen-host qemu-utils seabios ovmf
PKGS_VMM += default-libdecor-0-plugin-1
PKGS_VMM += librsvg2-common os-prober at-spi2-core libgdk-pixbuf2.0-bin libgtk-3-bin
PKGS_VMM += ipxe-qemu-256k-compat-efi-roms qemu-system-gui qemu-system-modules-spice
PKGS_VMM += qemu-system-modules-opengl qemu-block-extra cpu-checker
PKGS_VMM += libgl1-amber-dri

sqsh_layer-server: server.sqsh
	$(MKDIR_P) $@
	mount server.sqsh $@

.build/RUCKUS-VMM-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es sqsh_layer-server upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-server:sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
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
	rm -fv sysroot/tmp/*
	touch $@

vmm.sqsh: .build/RUCKUS-VMM-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards

# special target for adding compilation toolchain to an already
# prepped sysroot
sysroot-add-compilation:
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(PKGS_COMPILATION)


################################################################################
# Workstation
#
# Based off Server (which exludes ES-file and ES-web additions), adds everything
# else we want in an awesome workstation of doom (including the kitchen sink).
#
# NOTE: This image includes upstream Wine (including full 32bit library
#       support) so you can run Windows junk.
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
# - remove packages that got pulled in via ubuntu-desktop?  can i just mask these prior to install?
#   - thunderbird
#   - stupid autoupdate gui notification thing (update-manager or update-notifier?  looks like they both depend on each other)
#
.PHONY: PPAS_WORKSTATION
PPAS_WORKSTATION: sysroot/etc/apt/sources.list.d/wine.list
#PPAS_WORKSTATION: sysroot/etc/apt/sources.list.d/ubuntu-gis.list # FIXME: unneeded?
#PPAS_WORKSTATION: sysroot/etc/apt/sources.list.d/qgis.list # FIXME: pure upstream or UbuntuGIS?
#PPAS_WORKSTATION: sysroot/etc/apt/sources.list.d/steam.list


PKGS_WORKSTATION =
PKGS_WORKSTATION += ubuntu-desktop # FIXME: we'll want to prune things out of this later
# development stuff
PKGS_WORKSTATION += $(PKGS_COMPILATION)
PKGS_WORKSTATION += kernel-builder
PKGS_WORKSTATION += gcc-multilib g++-multilib gfortran-multilib
# GIS stuff
PKGS_WORKSTATION += gmt gmt-dcw gmt-gshhg proj-bin
PKGS_WORKSTATION += mbsystem
# everything else we want
PKGS_WORKSTATION += gimp gimp-help-en gimp-data-extras gsfonts libbluray-bdj mesa-opencl-icd libvdpau-va-gl1
PKGS_WORKSTATION += groff ghostscript imagemagick libpaper1 netpbm psutils
PKGS_WORKSTATION += ffmpeg
PKGS_WORKSTATION += gnuplot
PKGS_WORKSTATION += inkscape xfig dia
PKGS_WORKSTATION += audacity audacious
PKGS_WORKSTATION += opus-tools librsvg2-bin speex vorbis-tools sox
PKGS_WORKSTATION += va-driver-all vdpau-driver-all mesa-utils
PKGS_WORKSTATION += mesa-vulkan-drivers
PKGS_WORKSTATION += wireshark gkrellm gitk git-gui
PKGS_WORKSTATION += vlc
PKGS_WORKSTATION += gnome-tweaks rdesktop
PKGS_WORKSTATION += emacs-gtk # replaces emacs-nox
PKGS_WORKSTATION += wine-devel # upstream ppa

sqsh_layer-vmm: vmm.sqsh
	$(MKDIR_P) $@
	mount vmm.sqsh $@

.build/RUCKUS-WORKSTATION-OVL:
	$(MAKE) sysroot-clean
	$(MAKE) sqsh_layer-es sqsh_layer-server sqsh_layer-vmm upper_layer
	$(MKDIR_P) sysroot upper_layer/{upper,work}
	mount -t overlay -o lowerdir=sqsh_layer-vmm:sqsh_layer-server:sqsh_layer-es,upperdir=upper_layer/upper,workdir=upper_layer/work none sysroot
	touch $@

.build/RUCKUS-WORKSTATION-PPAS: .build/RUCKUS-WORKSTATION-OVL
	$(MAKE) sysroot-mount-vfs
	$(MAKE) PPAS_WORKSTATION
	touch $@

# add 32bit architecture support
.build/RUCKUS-WORKSTATION-ARCH: .build/RUCKUS-WORKSTATION-PPAS
	$(MAKE) sysroot-mount-vfs
	$(CHROOT) dpkg --add-architecture i386
	$(CHROOT) apt update
	touch $@

# apt install
#
# NOTE: We do --install-recommends for this one!  Bring on the kitchen
#       sink!
#
.build/RUCKUS-WORKSTATION-APT: .build/RUCKUS-WORKSTATION-ARCH
	$(MAKE) sysroot-mount-vfs
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) --install-recommends $(PKGS_WORKSTATION)
	touch $@

sysroot/tmp/google-chrome.deb:
	wget -qO- https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb > $@

sysroot/tmp/google-earth.deb:
	wget -qO- https://dl.google.com/linux/direct/google-earth-stable_current_amd64.deb > $@

.PHONY: DOWNLOADS_WORKSTATION
DOWNLOADS_WORKSTATION: sysroot/tmp/google-chrome.deb
DOWNLOADS_WORKSTATION: sysroot/tmp/google-earth.deb

PKGS_WORKSTATION_LOCAL =
PKGS_WORKSTATION_LOCAL += google-chrome
PKGS_WORKSTATION_LOCAL += google-earth
#PKGS_WORKSTATION_LOCAL += teamviewer
#PKGS_WORKSTATION_LOCAL += zoom
#PKGS_WORKSTATION_LOCAL += teams

# fetch debs and install locally
#
# FIXME: last time I did this, i installed via a single apt call with
#        a patsubst like this:
#
#	 cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) $(patsubst %,/tmp/%.deb,$(PKGS_ESWEB_LOCAL))
#
#        i should be able to do that here, but i need to figure out
#        out to get a single make call to fetch all the packages
#        still.  maybe the same way we handle multiple ppa creation?
#
#        passes the eyeball test, gotta test it
#
.build/RUCKUS-WORKSTATION-LOCAL: .build/RUCKUS-WORKSTATION-APT
	$(MAKE) sysroot-mount-vfs
	$(MAKE) DOWNLOADS_WORKSTATION
	cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) --install-recommends $(patsubst %,/tmp/%.deb,$(PKGS_WORKSTATION_LOCAL))
#	for x in $(PKGS_WORKSTATION_LOCAL); do \
#	  $(MAKE) sysroot/tmp/$$x.deb; \
#	  cat debconf-config.dat | $(CHROOT) $(APT_INSTALL) --install-recommends sysroot/tmp/$$x.deb; \
#	done
	touch $@

# service enable/disable, tweaks, etc
#
# FIXME: audit services
#
.build/RUCKUS-WORKSTATION-FINAL: .build/RUCKUS-WORKSTATION-LOCAL
	$(MAKE) sysroot-mount-vfs
	rm -fv sysroot/tmp/*
	touch $@

workstation.sqsh: .build/RUCKUS-WORKSTATION-FINAL
	$(MAKE) sysroot-umount-vfs
	mksquashfs upper_layer/upper $@ $(SQSH_COMP) -noappend -wildcards





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
