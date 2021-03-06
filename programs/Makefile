# ##########################################################################
# ZSTD programs - Makefile
# Copyright (C) Yann Collet 2015
#
# GPL v2 License
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along
# with this program; if not, write to the Free Software Foundation, Inc.,
# 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
#
# You can contact the author at :
#  - ZSTD source repository : http://code.google.com/p/zstd/
#  - Public forum : https://groups.google.com/forum/#!forum/lz4c
# ##########################################################################
# zstd : Command Line Utility, supporting gzip-like arguments
# datagen : Synthetic and parametrable data generator, for tests
# fuzzer  : Test tool, to check zstd integrity on target platform
# fuzzer32: Same as fuzzer, but forced to compile in 32-bits mode
# fullbench  : Precisely measure speed for each zstd inner function
# fullbench32: Same as fullbench, but forced to compile in 32-bits mode
# ##########################################################################

VERSION?= 0.4.0

DESTDIR?=
PREFIX ?= /usr/local
CPPFLAGS= -I../lib -DZSTD_VERSION=\"$(VERSION)\"
CFLAGS ?= -O3  # -falign-loops=32   # not always beneficial
CFLAGS += -std=c99 -Wall -Wextra -Wundef -Wshadow -Wcast-qual -Wcast-align -Wstrict-prototypes
FLAGS   = $(CPPFLAGS) $(CFLAGS) $(LDFLAGS) $(MOREFLAGS)

BINDIR  = $(PREFIX)/bin
MANDIR  = $(PREFIX)/share/man/man1
ZSTDDIR = ../lib

ZSTD_FILES := $(ZSTDDIR)/zstd_compress.c $(ZSTDDIR)/zstd_decompress.c $(ZSTDDIR)/fse.c $(ZSTDDIR)/huff0.c
ZSTD_LEGACY:= $(ZSTDDIR)/legacy/zstd_v01.c $(ZSTDDIR)/legacy/zstd_v02.c $(ZSTDDIR)/legacy/zstd_v03.c

ifeq ($(ZSTD_LEGACY_SUPPORT), 0)
CPPFLAGS  += -DZSTD_LEGACY_SUPPORT=0
else
ZSTD_FILES+= $(ZSTD_LEGACY)
CPPFLAGS  += -I../lib/legacy -I./legacy -DZSTD_LEGACY_SUPPORT=1
ZSTD_FILEIO_LEGACY = legacy/fileio_legacy.c
endif


# Define *.exe as extension for Windows systems
ifneq (,$(filter Windows%,$(OS)))
EXT =.exe
VOID = nul
else
EXT =
VOID = /dev/null
endif

ZBUFFTEST = -T2mn

.PHONY: default all clean install uninstall test test32 test-all

default: zstd

all: zstd zstd32 fullbench fullbench32 fuzzer fuzzer32 zbufftest zbufftest32 paramgrill datagen

zstd  : $(ZSTD_FILES) $(ZSTDDIR)/zstd_buffered.c \
        zstdcli.c fileio.c $(ZSTD_FILEIO_LEGACY) bench.c xxhash.c datagen.c 
	$(CC)      $(FLAGS) $^ -o $@$(EXT)

zstd32: $(ZSTD_FILES) $(ZSTDDIR)/zstd_buffered.c \
        zstdcli.c fileio.c $(ZSTD_FILEIO_LEGACY) bench.c xxhash.c datagen.c 
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

zstd_nolegacy :
	$(MAKE) zstd ZSTD_LEGACY_SUPPORT=0

zstd-pgo : MOREFLAGS = -fprofile-generate
zstd-pgo : clean zstd
	./zstd -b $(PROFILE_WITH)
	rm zstd
	$(MAKE) zstd MOREFLAGS=-fprofile-use

fullbench  : $(ZSTD_FILES) \
        datagen.c fullbench.c
	$(CC)      $(FLAGS) $^ -o $@$(EXT)

fullbench32: $(ZSTD_FILES) \
      datagen.c fullbench.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

fuzzer  : $(ZSTD_FILES) \
      datagen.c xxhash.c fuzzer.c
	$(CC)      $(FLAGS) $^ -o $@$(EXT)

fuzzer32: $(ZSTD_FILES) \
      datagen.c xxhash.c fuzzer.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

zbufftest  : $(ZSTD_FILES) $(ZSTDDIR)/zstd_buffered.c \
      datagen.c xxhash.c zbufftest.c
	$(CC)      $(FLAGS) $^ -o $@$(EXT)

zbufftest32: $(ZSTD_FILES) $(ZSTDDIR)/zstd_buffered.c \
      datagen.c xxhash.c zbufftest.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

paramgrill : $(ZSTD_FILES) \
      datagen.c xxhash.c paramgrill.c
	$(CC)      $(FLAGS) $^ -lm -o $@$(EXT)

datagen : datagen.c datagencli.c
	$(CC)      $(FLAGS) $^ -o $@$(EXT)

clean:
	@rm -f core *.o tmp* result* *.gcda \
        zstd$(EXT) zstd32$(EXT) \
        fullbench$(EXT) fullbench32$(EXT) \
        fuzzer$(EXT) fuzzer32$(EXT) zbufftest$(EXT) zbufftest32$(EXT) \
	datagen$(EXT) paramgrill$(EXT)
	@echo Cleaning completed


#------------------------------------------------------------------------
#make install is validated only for Linux, OSX, kFreeBSD and Hurd targets
ifneq (,$(filter $(shell uname),Linux Darwin GNU/kFreeBSD GNU))

install: zstd
	@echo Installing binaries
	@install -d -m 755 $(DESTDIR)$(BINDIR)/ $(DESTDIR)$(MANDIR)/
	@install -m 755 zstd$(EXT) $(DESTDIR)$(BINDIR)/zstd$(EXT)
	@ln -sf zstd$(EXT) $(DESTDIR)$(BINDIR)/zstdcat
	@ln -sf zstd$(EXT) $(DESTDIR)$(BINDIR)/unzstd
	@echo Installing man pages
	@install -m 644 zstd.1 $(DESTDIR)$(MANDIR)/zstd.1
	@ln -sf zstd.1 $(DESTDIR)$(MANDIR)/zstdcat.1
	@ln -sf zstd.1 $(DESTDIR)$(MANDIR)/unzstd.1
	@echo zstd installation completed

uninstall:
	rm -f $(DESTDIR)$(BINDIR)/zstdcat
	rm -f $(DESTDIR)$(BINDIR)/unzstd
	[ -x $(DESTDIR)$(BINDIR)/zstd$(EXT) ] && rm -f $(DESTDIR)$(BINDIR)/zstd$(EXT)
	rm -f $(DESTDIR)$(MANDIR)/zstdcat.1
	rm -f $(DESTDIR)$(MANDIR)/unzstd.1
	[ -f $(DESTDIR)$(MANDIR)/zstd.1 ] && rm -f $(DESTDIR)$(MANDIR)/zstd.1
	@echo zstd programs successfully uninstalled

test: test-zstd test-fullbench test-fuzzer test-zbuff

test32: test-zstd32 test-fullbench32 test-fuzzer32 test-zbuff32

test-all: test test32 valgrindTest

zstd-playTests: datagen
	@echo "\n**** frame concatenation **** "
	@echo "hello " > hello.tmp
	@echo "world!" > world.tmp
	@cat hello.tmp world.tmp > helloworld.tmp
	$(ZSTD) hello.tmp > hello.zstd
	$(ZSTD) world.tmp > world.zstd
	@cat hello.zstd world.zstd > helloworld.zstd
	$(ZSTD) -df helloworld.zstd > result.tmp
	cat result.tmp
	sdiff helloworld.tmp result.tmp
	@rm *.tmp *.zstd
	@echo frame concatenation test completed
	@echo "**** flush write error test **** "
	echo foo | $(ZSTD) > /dev/full; if [ $$? -eq 0 ] ; then echo "write error not detected!"; false; fi
	echo foo | $(ZSTD) | $(ZSTD) -d > /dev/full; if [ $$? -eq 0 ] ; then echo "write error not detected!"; false; fi
	@echo "**** zstd round-trip tests **** "
	@./datagen             | md5sum > tmp1
	./datagen              | $(ZSTD) -v    | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen              | $(ZSTD) -6 -v | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g270000000 | md5sum > tmp1
	./datagen -g270000000  | $(ZSTD) -v    | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g270000000  | $(ZSTD) -v2   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g270000000  | $(ZSTD) -v3   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g140000000 -P60| md5sum > tmp1
	./datagen -g140000000 -P60 | $(ZSTD) -v4   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g140000000 -P60 | $(ZSTD) -v5   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g140000000 -P60 | $(ZSTD) -v6   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g70000000 -P70 | md5sum > tmp1
	./datagen -g70000000 -P70  | $(ZSTD) -v7   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g70000000 -P70  | $(ZSTD) -v8   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g70000000 -P70  | $(ZSTD) -v9   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g35000000 -P75 | md5sum > tmp1
	./datagen -g35000000 -P75  | $(ZSTD) -v10  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g35000000 -P75  | $(ZSTD) -v11  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g35000000 -P75  | $(ZSTD) -v12  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g18000000 -P80 | md5sum > tmp1
	./datagen -g18000000 -P80  | $(ZSTD) -v13  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g18000000 -P80  | $(ZSTD) -v14  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g18000000 -P80  | $(ZSTD) -v15  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g18000000 -P80  | $(ZSTD) -v16  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g18000000 -P80  | $(ZSTD) -v17  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g50000000 -P94 | md5sum > tmp1
	./datagen -g50000000 -P94  | $(ZSTD) -v18  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g50000000 -P94  | $(ZSTD) -v19  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	@./datagen -g99000000 -P99 | md5sum > tmp1
	./datagen -g99000000 -P99  | $(ZSTD) -v20  | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   
	./datagen -g6000000000 -P99| md5sum > tmp1
	./datagen -g6000000000 -P99| $(ZSTD) -vq   | $(ZSTD) -d  | md5sum > tmp2
	@diff tmp1 tmp2   

test-zstd: ZSTD = ./zstd
test-zstd: zstd zstd-playTests

test-zstd32: ZSTD = ./zstd32
test-zstd32: zstd32 zstd-playTests

test-zstd_nolegacy: ZSTD = ./zstd
test-zstd_nolegacy: zstd_nolegacy zstd-playTests

test-fullbench: fullbench datagen
	./fullbench -i1
	./fullbench -i1 -P0

test-fullbench32: fullbench32 datagen
	./fullbench32 -i1
	./fullbench32 -i1 -P0

test-fuzzer: fuzzer
	./fuzzer

test-fuzzer32: fuzzer32
	./fuzzer32

test-zbuff: zbufftest
	./zbufftest $(ZBUFFTEST)

test-zbuff32: zbufftest32
	./zbufftest32 $(ZBUFFTEST)

valgrindTest: zstd datagen fuzzer fullbench zbufftest
	@echo "\n ---- valgrind tests : memory analyzer ----"
	valgrind --leak-check=yes --error-exitcode=1 ./datagen -g50M > $(VOID)
	./datagen -g16KB > tmp
	valgrind --leak-check=yes --error-exitcode=1 ./zstd -vf tmp $(VOID)
	./datagen -g2930KB > tmp
	valgrind --leak-check=yes --error-exitcode=1 ./zstd -5 -vf tmp tmp2
	valgrind --leak-check=yes --error-exitcode=1 ./zstd -vdf tmp2 $(VOID)
	./datagen -g64MB > tmp
	valgrind --leak-check=yes --error-exitcode=1 ./zstd -vf tmp $(VOID)
	@rm tmp
	valgrind --leak-check=yes --error-exitcode=1 ./fuzzer -i1000 -t1
	valgrind --leak-check=yes --error-exitcode=1 ./fullbench -i1
	valgrind --leak-check=yes --error-exitcode=1 ./zbufftest -T1mn

endif
