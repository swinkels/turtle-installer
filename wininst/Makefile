MAKENSIS="C:/Program Files/NSIS/makensis"

EXPORT_DIR=$(TEMP)/turtle-$(TURTLE_EDITION)-export

# Make sure the following variables are passed through to sub-makes.
export MAKE_PYTHONVERSION
export MAKE_PYTHONDIR
export TURTLE_EDITION
export TAGGED_VERSION

py26: MAKE_PYTHONVERSION=2.6
py26: MAKE_PYTHONDIR=Python26
py26: create-setup-from-export

py25: MAKE_PYTHONVERSION=2.5
py25: MAKE_PYTHONDIR=Python25
py25: create-setup-from-export

py24: MAKE_PYTHONVERSION=2.4
py24: MAKE_PYTHONDIR=Python24
py24: create-setup-from-export

create-setup-from-export: check-turtle-edition check-tagged-version export-tag install-wininst
	cd $(EXPORT_DIR)/wininst; make create-setup

check-turtle-edition:
ifndef TURTLE_EDITION
	@echo "Please specify the edition of Turtle for which you want to build the Windows"
	@echo "installer, e.g. 'make TURTLE_EDITION=rural TAGGED_VERSION=3.1'"
	false
endif

check-tagged-version:
ifndef TAGGED_VERSION
	@echo "Please specify the tag of the Turtle version for which you want to build the"
	@echo "Windows installer, e.g. 'make TURTLE_EDITION=rural TAGGED_VERSION=3.1'"
	false
endif

export-tag:
	- rm -rf $(EXPORT_DIR)
	svn export https://office.nelen-schuurmans.nl/svn/Products/Turtle/turtle-$(TURTLE_EDITION)/tags/$(TAGGED_VERSION) $(EXPORT_DIR)

install-wininst:
	- rm -rf $(EXPORT_DIR)/wininst/*
	cp * $(EXPORT_DIR)/wininst

create-setup: clean
	cp turtle-rural.nsi autogen-turtle-$(TURTLE_EDITION).nsi
	# If setuptools is installed as a site package, which is
	# required by zest.releaser, the execution of buildout in this
	# Makefile will not download it. However, the execution of
	# buildout in the installer requires it so we have to make
	# sure it is in the right place.
	#
	# Ideally, the buildout option 'include-site-packages=false'
	# lets buildout ignore site packages. Unfortunately, some
	# recipes ignore this option and a site packages may still
	# end up being used.
	cp setuptools*.egg ../eggs
	# we escape the dollar sign in the following sed regular expressions,
	# see the '$$', otherwise make would try to expand them
	sed -i 's/rural/$(TURTLE_EDITION)/g' autogen-turtle-$(TURTLE_EDITION).nsi
	sed -i 's/\(StrCpy $$PYTHONVERSION[ ][ ]*\).*/\1"$(MAKE_PYTHONVERSION)"/' autogen-turtle-$(TURTLE_EDITION).nsi
	sed -i 's/\(StrCpy $$PYTHONDIR[ ][ ]*\).*/\1"C:\\$(MAKE_PYTHONDIR)"/' autogen-turtle-$(TURTLE_EDITION).nsi
	sed -i 's/\(zc.buildout-[0-9]*[.][0-9]*[.][0-9]*-py\)[0-9]*[.][0-9]*/\1$(MAKE_PYTHONVERSION)/' autogen-turtle-$(TURTLE_EDITION).nsi
	# we use a forward slash as a directory separator in the shell commands
	# as the shell used by make does not like the backslash (or double
	# backslash) as one
	cd ..; c:/$(MAKE_PYTHONDIR)/python bootstrap.py
	cd ..; bin/buildout buildout:download-cache=downloads buildout:include-site-packages=false
	$(MAKENSIS) autogen-turtle-$(TURTLE_EDITION).nsi
	mv turtle-$(TURTLE_EDITION)-setup.exe turtle-$(TURTLE_EDITION)-setup-python$(MAKE_PYTHONVERSION).exe

clean:
	- rm -rf ../downloads
	- rm -rf ../eggs
	- rm -rf ../bin
	mkdir ../downloads
	mkdir ../downloads/dist
	mkdir ../eggs
	mkdir ../bin
