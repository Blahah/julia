JULIAHOME = ..
include ../Make.inc

PCRE_CONST = 0x[0-9a-fA-F]+|[-+]?\s*[0-9]+

# These are all the values needed for the RawVersionInfo struct
version_string = $(shell cat ../VERSION)
commit = $(shell git rev-parse HEAD 2>/dev/null)
commit_short = $(shell git rev-parse --short HEAD 2>/dev/null)
build_number = $(shell echo $$(git describe --tags --long 2>/dev/null) | awk -F- '{ if( NF > 1 ) print $$(NF-1); else print 0; }')
git_tag = $(shell git describe --tags --abbrev=0 2>/dev/null)
prerelease = $(shell [ -z "$(git_tag)" -o "$(git_tag)" = "v$(version_string)" ] && echo false || echo true)

git_branch = $(shell git branch 2>/dev/null | sed -n '/\* /s///p')
git_time = $(shell git log -1 --pretty=format:%ct 2>/dev/null)
ifneq ($(git_time), )
	ifneq (,$(filter $(OS), Darwin FreeBSD))
		date_string = "$(shell /bin/date -jr $(git_time) -u '+%Y-%m-%d %H:%M:%S %Z')"
	else
		date_string = "$(shell /bin/date --date=@$(git_time) -u '+%Y-%m-%d %H:%M:%S %Z')"
	endif
else
	date_string = ""
endif
dirty = $(shell [ -z "$(shell git status --porcelain 2>/dev/null)" ] && echo "false" || echo "true" )



all: pcre_h.jl errno_h.jl build_h.jl.phony fenv_constants.jl file_constants.jl uv_constants.jl

pcre_h.jl:
	@$(call PRINT_PERL, $(CPP) -dM $(shell $(PCRE_CONFIG) --prefix)/include/pcre.h | perl -nle '/^\s*#define\s+PCRE_(\w*)\s*\(?($(PCRE_CONST))\)?\s*$$/ and print "const $$1 = uint32($$2)"' | sort > $@)

errno_h.jl:
	@$(call PRINT_PERL, echo '#include "errno.h"' | $(CPP) -dM - | perl -nle 'print "const $$1 = int32($$2)" if /^#define\s+(E\w+)\s+(\d+)\s*$$/' | sort > $@)

fenv_constants.jl: ../src/fenv_constants.h
	@$(PRINT_PERL) ${CPP} -P -DJULIA ../src/fenv_constants.h | tail -n 8 | perl -ple 's/\sFE_UN\w+/ 0x10/g; s/\sFE_O\w+/ 0x08/g; s/\sFE_DI\w+/ 0x04/g; s/\sFE_INV\w+/ 0x01/g; s/\sFE_TON\w+/ 0x00/g; s/\sFE_UP\w+/ 0x800/g; s/\sFE_DO\w+/ 0x400/g; s/\sFE_TOW\w+/ 0xc00/g' > $@

file_constants.jl: ../src/file_constants.h
	@$(call PRINT_PERL, $(CPP) -P -DJULIA ../src/file_constants.h | perl -nle 'print "$$1 0o$$2" if /^(\s*const\s+[A-z_]+\s+=)\s+(0[0-9]*)\s*$$/; print "$$1" if /^\s*(const\s+[A-z_]+\s+=\s+([1-9]|0x)[0-9A-z]*)\s*$$/' > $@)

uv_constants.jl: ../src/uv_constants.h
	@$(call PRINT_PERL, $(CPP) -P "-I$(LIBUV_INC)" -DJULIA ../src/uv_constants.h | tail -n 16 > $@)

build_h.jl.phony:
	@$(CC) -E -P build.h -I../src/support | grep . > $@
	@echo "const ARCH = :$(ARCH)" >> $@
ifeq ($(OS),$(BUILD_OS))
	@echo "const MACHINE = \"$(BUILD_MACHINE)\"" >> $@
else
	@echo "const MACHINE = \"$(shell $(CC) -dumpmachine)\"" >> $@
endif
	@echo "const libm_name = \"$(LIBMNAME)\"" >> $@
	@echo "const libblas_name = \"$(LIBBLASNAME)\"" >> $@
	@echo "const liblapack_name = \"$(LIBLAPACKNAME)\"" >> $@
ifeq ($(USE_BLAS64), 1)
	@echo "const USE_BLAS64 = true" >> $@
else
	@echo "const USE_BLAS64 = false" >> $@
endif

	@echo "immutable BuildInfo" >> $@
	@echo "    version_string::ASCIIString" >> $@
	@echo "    commit::ASCIIString" >> $@
	@echo "    commit_short::ASCIIString" >> $@
	@echo "    branch::ASCIIString" >> $@
	@echo "    build_number::Int" >> $@
	@echo "    date_string::ASCIIString" >> $@
	@echo "    dirty::Bool" >> $@
	@echo "    prerelease::Bool" >> $@
	@echo "end" >> $@


	@echo "const BUILD_INFO = BuildInfo( \
			'\"$(version_string)\"', \
			'\"$(commit)\"', \
			'\"$(commit_short)\"', \
			'\"$(git_branch)\"', \
			$(build_number), \
			'\"$(date_string)\"', \
			$(dirty), \
			$(prerelease) \
			)" | xargs >> $@

	@# This to ensure that we always rebuild this file, but only when it is modified do we touch build_h.jl,
	@# ensuring we rebuild the system image as infrequently as possible
	@if ! cmp -s $@ build_h.jl; then \
		$(call PRINT_PERL,) \
		mv $@ build_h.jl; \
	else \
		rm -f $@; \
	fi

.PHONY: build_h.jl.phony



clean:
	rm -f *# *~
	rm -f pcre_h.jl
	rm -f errno_h.jl
	rm -f build_h.jl
	rm -f fenv_constants.jl
	rm -f uv_constants.jl
	rm -f file_constants.jl
