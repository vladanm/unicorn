# we can run tests in parallel with GNU make
all::

pid := $(shell echo $$PPID)

RUBY = ruby
RAKE = rake
-include ../local.mk
ifeq ($(RUBY_VERSION),)
  RUBY_VERSION := $(shell $(RUBY) -e 'puts RUBY_VERSION')
endif

ifeq ($(RUBY_VERSION),)
  $(error unable to detect RUBY_VERSION)
endif

RUBY_ENGINE := $(shell $(RUBY) -e 'puts((RUBY_ENGINE rescue "ruby"))')
export RUBY_ENGINE

isolate_libs := ../tmp/isolate/$(RUBY_ENGINE)-$(RUBY_VERSION).mk
$(isolate_libs): ../script/isolate_for_tests
	@cd .. && $(RUBY) script/isolate_for_tests
-include $(isolate_libs)
MYLIBS := $(RUBYLIB):$(ISOLATE_LIBS)

T = $(wildcard t[0-9][0-9][0-9][0-9]-*.sh)

all:: $(T)

# can't rely on "set -o pipefail" since we don't require bash or ksh93 :<
t_pfx = trash/$@-$(RUBY_ENGINE)-$(RUBY_VERSION)
TEST_OPTS =
# TRACER = strace -f -o $(t_pfx).strace -s 100000
# TRACER = /usr/bin/time -o $(t_pfx).time

ifdef V
  ifeq ($(V),2)
    TEST_OPTS += --trace
  else
    TEST_OPTS += --verbose
  endif
endif

random_blob:
	dd if=/dev/urandom bs=1M count=30 of=$@.$(pid)
	mv $@.$(pid) $@

ssl-stamp:
	./sslgen.sh
	> $@

$(T): random_blob ssl-stamp

dependencies := socat curl
deps := $(addprefix .dep+,$(dependencies))
$(deps): dep_bin = $(lastword $(subst +, ,$@))
$(deps):
	@which $(dep_bin) > $@.$(pid) 2>/dev/null || :
	@test -s $@.$(pid) || \
	  { echo >&2 "E '$(dep_bin)' not found in PATH=$(PATH)"; exit 1; }
	@mv $@.$(pid) $@
dep: $(deps)

test_prefix := $(CURDIR)/../test/$(RUBY_ENGINE)-$(RUBY_VERSION)
$(test_prefix)/.stamp:
	$(MAKE) -C .. test-install

$(T): export RUBY := $(RUBY)
$(T): export RAKE := $(RAKE)
$(T): export PATH := $(test_prefix)/bin:$(PATH)
$(T): export RUBYLIB := $(test_prefix)/lib:$(MYLIBS)
$(T): dep $(test_prefix)/.stamp trash/.gitignore
	$(TRACER) $(SHELL) $(SH_TEST_OPTS) $@ $(TEST_OPTS)

trash/.gitignore:
	mkdir -p $(@D)
	echo '*' > $@

clean:
	$(RM) -r trash/*

.PHONY: $(T) clean
