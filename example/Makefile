SBCL_DIR=/tmp/sbcl
XC_HOST=sbcl
#for roswell
#XC_HOST="ros -L sbcl-bin without-roswell=t run -- --no-userinit --no-sysinit"

.PHONY: all clean run setup-sbcl-dir

UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
  CFLAGS += -Wl,-pagezero_size,0x100000
  RUN_PREFIX += DYLD_LIBRARY_PATH=`pwd`:$(SBCL_DIR)/src/runtime
else
  RUN_PREFIX += LD_LIBRARY_PATH=`pwd`:$(SBCL_DIR)/src/runtime
endif

all: example

setup-sbcl-dir:
	git clone --depth 30 https://github.com/sbcl/sbcl $(SBCL_DIR)
	cd $(SBCL_DIR); \
	 echo '"1.0.99.999"' > version.lisp-expr; \
	 sh make.sh --with-sb-linkable-runtime --xc-host=$(XC_HOST);  \
	 sh make-shared-library.sh

libcalc.core libcalc.c: libcalc.lisp
	$(SBCL_DIR)/run-sbcl.sh \
	  --eval "(require :asdf)" \
	  --load `pwd`/../sbcl-librarian.asd \
	  --script libcalc.lisp

libcalc.so: libcalc.c libcalc.core
	$(CC) -o $@ $<  --shared -lsbcl -L$(SBCL_DIR)/src/runtime

example: example.c libcalc.so
	$(CC) -o $@ $< -lcalc -lsbcl -L. -L$(SBCL_DIR)/src/runtime $(CFLAGS)

run:
	$(RUN_PREFIX) ./example

clean:
	rm -f libcalc.c libcalc.h libcalc.core libcalc.so example
