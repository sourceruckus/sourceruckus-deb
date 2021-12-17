SUBDIRS =
#SUBDIRS += src

top_docs = README COPYING AUTHORS TODO NOTES

EXTRA_DIST =
EXTRA_DIST += $(top_docs)


build-es/.RUCKUS-DEBOOTSTRAP:
	debootstrap --arch=amd64 focal $(dir $@) http://archive.ubuntu.com/ubuntu/
	touch $@

# - get chrooted
# - setup apt sources
# - do stage 2 apt install
#   - compilation toolchain
#   - emacs zsh gkrellm gkrellmd
# - get ready to compile stuff
#   - openzfs
#   - drbd9
#   - ruckusrd
#   - pingenator
#   - awesome-powers
# - get 3rd party stuff
#   - teamviewer (even though it probably won't ever work)
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
	echo chroot $(dir $@) /usr/bin/env -i HOME=/root TERM=$TERM /bin/bash --login
	touch $@

# FIXME: temporary PHONY for testing deps... remove when done
.PHONY: es.sqsh
es.sqsh: build-es/.RUCKUS-ES
	echo mksquashfs $(dir $^) $@ -comp lz4 -Xhc -noappend -wildcards -e .RUCKUS*

#es-wine.sqsh: builde-es-wine/.DONE
#	mksquashfs ...
