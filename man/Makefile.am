# Make coreutils man pages.				-*-Makefile-*-

# Copyright (C) 2002-2012 Free Software Foundation, Inc.

# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.

# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

dist_man1_MANS = $(MAN)

man_aux = $(dist_man1_MANS:.1=.x)

EXTRA_DIST = $(man_aux) $(NO_INSTALL_PROGS_DEFAULT:%=%.x) help2man \
	$(NO_INSTALL_PROGS_DEFAULT:%=%.1)
MAINTAINERCLEANFILES = $(dist_man1_MANS)

# Depend on ../.version to get version number changes.
common_dep = ../.version

all_programs := $(strip $(filter-out [, $(shell \
  cd $(top_srcdir)/src && MAKEFLAGS= $(MAKE) -s all_programs.list)))

get-c-for-man = $(strip \
  $(patsubst arch, uname, \
  $(patsubst dir, ls, \
  $(patsubst vdir, ls, \
  $(patsubst sha%sum, md5sum, \
  $1))))).c

# Dependencies for each generated man page.
$(foreach p, $(patsubst ginstall,install,$(all_programs)), \
  $(eval $p.1: $(common_dep) $(srcdir)/$p.x) \
  $(eval $p.1: $(top_srcdir)/src/$(call get-c-for-man,$p)))

# This is required so that changes to e.g., emit_bug_reporting_address
# provoke regeneration of all $(MAN) files.
$(MAN): $(top_srcdir)/src/system.h

SUFFIXES = .x .1

# Ensure that help2man runs the ../src/ginstall binary as
# 'install' when creating install.1.
# Similarly, ensure that it uses the ../src/[ binary to create test.1.
t = $*.td
mapped_name = `echo $*|sed 's/^install$$/ginstall/; s/^test$$/[/'`

# Note the use of $t/$*, rather than just '$*' as in other packages.
# That is necessary to avoid failures for programs that are also shell built-in
# functions like echo, false, printf, pwd.
.x.1:
	$(AM_V_GEN)case '$(PERL)' in				\
	  *"/missing "*)					\
	    echo 'WARNING: cannot update man page $@ since perl is missing' \
	      'or inadequate' 1>&2				\
	    ;;							\
	  *)							\
	    rm -f $@ $@-t					\
	    && {						\
		 rm -rf $t;					\
		 mkdir $t;					\
		 (cd $t && $(LN_S) ../../src/$(mapped_name) $*); \
		    $(PERL) -- $(srcdir)/help2man		\
		     --source='$(PACKAGE_STRING)'		\
		     --include=$(srcdir)/$*.x			\
		     --output=$t/$@ $t/$*;			\
	       }						\
	    && sed 's|$*\.td/||g' $t/$@ > $@-t			\
	    && rm -rf $t					\
	    && chmod -w $@-t					\
	    && mv $@-t $@					\
	    ;;							\
	esac

# Option descriptions should not start with a capital letter
.PHONY: sc_option_desc_uppercase
sc_option_desc_uppercase: $(dist_man1_MANS) $(NO_INSTALL_PROGS_DEFAULT:%=%.1)
	@grep '^\\fB\\-' -A1 *.1 | LC_ALL=C grep '\.1.[A-Z][a-z]' && \
	    { echo 1>&2 '$(ME): found initial capitals in --help';   \
	      exit 1; } || :;

.PHONY: sc_man_file_correlation
sc_man_file_correlation: check-x-vs-1 check-programs-vs-x

# Sort in traditional ASCII order, regardless of the current locale;
# otherwise we may get into trouble with distinct strings that the
# current locale considers to be equal.
ASSORT = LC_ALL=C sort

# Ensure that for each .x file in this directory, there is a
# corresponding .1 file in the definition of $(dist_man1_MANS) above.
# But since that expansion usually lacks programs like arch and hostname,
# add them here manually.
.PHONY: check-x-vs-1
check-x-vs-1:
	@PATH=../src$(PATH_SEPARATOR)$$PATH; export PATH;		\
	t=$@-t;								\
	(cd $(srcdir) && ls -1 *.x) | sed 's/\.x$$//' | $(ASSORT) > $$t;\
	(echo $(dist_man1_MANS) $(NO_INSTALL_PROGS_DEFAULT)		\
	  | tr -s ' ' '\n' | sed 's/\.1$$//')				\
	  | $(ASSORT) -u | diff - $$t || { rm $$t; exit 1; };		\
	rm $$t

.PHONY: check-programs-vs-x
check-programs-vs-x:
	@status=0;					\
	for p in $(all_programs); do			\
	  test $$p = ginstall && p=install || : ;	\
	  test -f $(srcdir)/$$p.x			\
	    || { echo missing $$p.x 1>&2; status=1; };	\
	done;						\
	exit $$status
