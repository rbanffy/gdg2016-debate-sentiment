# Override VERSION if not defined
VERSION?=$(shell resources/get_current_sdk_version.sh)

# Override VENV if not defined
VENV?=.env

help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  venv       to build the working virtual environment, and to install requirements for deployment"
	@echo "  pep8       to run pep8 on the src and tests folders"
	@echo "  pyflakes   to run pyflakes on the src and tests folders"
	@echo "  clean_venv to remove the created virtualenv folder"
	@echo "  travis	    runs all tests (Travis CI hook)"

# Consciously avoiding "all" target because we may want to use it for building
# the actual product rather than a sane testing environment
venv: virtualenv requirements appenginesdk

# Creates the virtualenv, adds 'src' to the python path
virtualenv:
	virtualenv $(CURDIR)/$(VENV)
	cp $(CURDIR)/resources/autogenerated $(CURDIR)/$(VENV)/lib/python2.7/site-packages/src.pth
	echo "$(CURDIR)/src/" >> $(CURDIR)/$(VENV)/lib/python2.7/site-packages/src.pth

# Install all modules that will be used during development.
requirements:
	$(CURDIR)/$(VENV)/bin/pip install --cache-dir $(CURDIR)/cache -r $(CURDIR)/resources/requirements.txt

# Download the current Appengine SDK, unpack it in the virtualenv's
# lib directory and add it and its patches to the gae.pth file that'll
# set up the virtualenv's path.
appenginesdk: virtualenv directories
	@wget -c https://storage.googleapis.com/appengine-sdks/featured/google_appengine_$(VERSION).zip -O $(CURDIR)/cache/google_appengine_$(VERSION).zip
	@unzip -q -o $(CURDIR)/cache/google_appengine_$(VERSION).zip -d $(CURDIR)/$(VENV)/lib
	@ln -svf $(CURDIR)/$(VENV)/lib/google_appengine/*.py $(CURDIR)/$(VENV)/bin/
	@cp $(CURDIR)/resources/autogenerated $(CURDIR)/$(VENV)/lib/python2.7/site-packages/gae.pth
	@echo "$(CURDIR)/$(VENV)/lib/google_appengine/" >> $(CURDIR)/$(VENV)/lib/python2.7/site-packages/gae.pth
	@echo "import dev_appserver; dev_appserver.fix_sys_path()" >> $(CURDIR)/$(VENV)/lib/python2.7/site-packages/gae.pth

# This creates directories that will be used during setup processes.
directories:
	mkdir -p $(CURDIR)/build $(CURDIR)/cache

# A useful target for PEP-8'ing your source tree
pep8:
	find $(CURDIR)/src/ -name *.py -exec pep8 {} \;
	find $(CURDIR)/tests/ -name *.py -exec pep8 {} \;

# The same for pyflakes
pyflakes:
	find $(CURDIR)/src/ -name *.py -exec pyflakes {} \;
	find $(CURDIR)/tests/ -name *.py -exec pyflakes {} \;

clean_dirs:
	rm -rf $(CURDIR)/build/*

clean_cache:
	rm -rf $(CURDIR)/cache/*

# Also avoiding the "clean" target for the reasons described at the "venv" target
# Deletes the virtualenv
clean_venv: clean_dirs
	rm -rf $(CURDIR)/$(VENV)

travis: venv
	.env/bin/nosetests
