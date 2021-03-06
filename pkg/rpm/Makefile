#!/usr/bin/make

DEFAULT_VERSION := $(shell grep 'define NXT_VERSION' ../../src/nxt_main.h \
			| sed -e 's/^.*"\(.*\)".*/\1/')

DEFAULT_RELEASE := 1

VERSION ?= $(DEFAULT_VERSION)
RELEASE ?= $(DEFAULT_RELEASE)

BUILD_DEPENDS = libxml2 libxslt rpm-build rpmlint

MODULES=
-include Makefile.*

CONFIGURE_ARGS=\
	--prefix=/usr \
	--state=%{_sharedstatedir}/unit \
	--control="unix:/var/run/control.unit.sock" \
	--pid=/var/run/unit.pid \
	--log=/var/log/unit.log \
	--tests

export CR=\\n

default:
	@echo "valid targets: all modules unit $(addprefix unit-, $(MODULES)) rpmlint specs clean"

all: check-build-depends unit modules

modules: $(addprefix unit-, $(MODULES))

specs: $(addsuffix .spec, $(addprefix rpmbuild/SPECS/unit-, $(MODULES)))

check-build-depends:
	@{ \
	not_installed= ; \
	for pkg in $(BUILD_DEPENDS); do \
		rpm -qi $${pkg} >/dev/null 2>&1 ; \
		if [ $$? -ne 0 ]; then \
			not_installed="$${not_installed} $${pkg}" ; \
		fi ; \
	done ; \
	if test -n "$${not_installed}" ; then \
		echo "" >&2 ; \
		echo "The following packages are required in order to proceed:" >&2 ; \
		echo "" >&2 ; \
		echo $${not_installed} >&2 ; \
		echo "" >&2 ; \
		exit 1 ; \
	fi \
	}

rpmbuild/SPECS:
	mkdir -p rpmbuild/SPECS

rpmbuild/SPECS/unit.spec: unit.spec.in ../../docs/changes.xml | rpmbuild/SPECS
	cat unit.spec.in | \
		sed -e "s#%%VERSION%%#$(VERSION)#g" \
		    -e "s#%%RELEASE%%#$(RELEASE)#g" \
		    -e "s#%%CONFIGURE_ARGS%%#$(CONFIGURE_ARGS)#g" \
		> rpmbuild/SPECS/unit.spec
	cd ../../docs && make ../build/unit.rpm-changelog
ifneq ($(DEFAULT_VERSION)$(DEFAULT_RELEASE), $(VERSION)$(RELEASE))
	cat ../../build/unit.rpm-changelog | sed -e \
		"s/> - $(DEFAULT_VERSION)-$(DEFAULT_RELEASE)/> - $(VERSION)-$(RELEASE)/" \
		>> rpmbuild/SPECS/unit.spec
else
	cat ../../build/unit.rpm-changelog >> rpmbuild/SPECS/unit.spec
endif

rpmbuild/SOURCES/unit-$(VERSION).tar.gz:
	cd ../.. && tar -czf pkg/rpm/rpmbuild/SOURCES/unit-$(VERSION).tar.gz \
		--transform "s#^#unit-$(VERSION)/#" \
		LICENSE NOTICE CHANGES README configure auto src test

unit: rpmbuild/SPECS/unit.spec rpmbuild/SOURCES/unit-$(VERSION).tar.gz
	@echo "===> Building $@ package" ; \
	rpmbuild -D "_topdir `pwd`/rpmbuild" -ba rpmbuild/SPECS/unit.spec && \
	ln -s rpmbuild/BUILD/$@-$(VERSION)/build $@

rpmlint:
	find rpmbuild/ -name "*.rpm" -print -exec rpmlint {} \;

rpmbuild/SPECS/unit-%.spec: unit.module.spec.in ../../docs/changes.xml | rpmbuild/SPECS
	@echo "===> Creating $@"
	@{ \
	set -e ; \
	i=100 ; \
	sources= ; \
	for src in $(MODULE_SOURCES_$*); do \
		s="`printf "Source%d: %s\n" $${i} $${src}`" ; \
		sources="$${sources}\n$${s}" ; \
		i=$$(($${i}+1)) ; \
	done ; \
	pkgname=$(shell echo $@ | cut -d '/' -f 3 | cut -d '.' -f 1) ; \
	definitions=`echo "$$MODULE_DEFINITIONS_$*" | sed -e ':a' -e 'N' -e '$$!ba' -e "s/\n/\$$CR/g"` ; \
	preinstall=`echo "$$MODULE_PREINSTALL_$*" | sed -e ':a' -e 'N' -e '$$!ba' -e "s/\n/\$$CR/g"` ; \
	files=`echo "$$MODULE_FILES_$*" | sed -e ':a' -e 'N' -e '$$!ba' -e "s/\n/\$$CR/g"` ; \
	post=`echo "$$MODULE_POST_$*" | sed -e ':a' -e 'N' -e '$$!ba' -e "s/\n/\$$CR/g"` ; \
	cat unit.module.spec.in | sed \
		-e "s#%%NAME%%#$${pkgname}#g" \
		-e "s#%%SUMMARY%%#$(MODULE_SUMMARY_$*)#g" \
		-e "s#%%VERSION%%#$(MODULE_VERSION_$*)#g" \
		-e "s#%%RELEASE%%#$(MODULE_RELEASE_$*)#g" \
		-e "s#%%UNIT_VERSION%%#$(VERSION)#g" \
		-e "s#%%UNIT_RELEASE%%#$(RELEASE)#g" \
		-e "s#%%MODULE_SOURCES%%#$${sources}#g" \
		-e "s#%%CONFIGURE_ARGS%%#$(CONFIGURE_ARGS)#g" \
		-e "s#%%MODULE_CONFARGS%%#$(MODULE_CONFARGS_$*)#g" \
		-e "s#%%MODULE_MAKEARGS%%#$(MODULE_MAKEARGS_$*)#g" \
		-e "s#%%MODULE_INSTARGS%%#$(MODULE_INSTARGS_$*)#g" \
		-e "s#%%MODULE_DEFINITIONS%%#$${definitions}#g" \
		-e "s#%%MODULE_PREINSTALL%%#$${preinstall}#g" \
		-e "s#%%MODULE_FILES%%#$${files}#g" \
		-e "s#%%MODULE_POST%%#$${post}#g" \
		> $@.tmp ; \
	}
	cd ../../docs && make ../build/unit-$*.rpm-changelog
	cat ../../build/unit-$*.rpm-changelog | sed -e \
		"s/> - $(DEFAULT_VERSION)-$(DEFAULT_RELEASE)/> - $(MODULE_VERSION_$*)-$(MODULE_RELEASE_$*)/" \
		>> $@.tmp
	mv $@.tmp $@ 

unit-%: rpmbuild/SPECS/unit-%.spec rpmbuild/SOURCES/unit-$(VERSION).tar.gz
	@echo "===> Building $@ package" ; \
	rpmbuild -D "_topdir `pwd`/rpmbuild" -ba rpmbuild/SPECS/$@.spec && \
	ln -s rpmbuild/BUILD/$@-$(VERSION)/build $@

clean:
	rm -rf rpmbuild/SPECS rpmbuild/BUILD rpmbuild/BUILDROOT rpmbuild/RPMS rpmbuild/SRPMS ../../build
	rm -f rpmbuild/SOURCES/unit-*.tar.gz unit
	find . -maxdepth 1 -type l -delete

.PHONY: default all modules specs check-build-depends rpmlint clean

.SECONDARY:
