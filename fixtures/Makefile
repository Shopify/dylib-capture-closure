# Create three objects:
# 
# c(executable) --> b(lib) --> a(lib)
TARGETS = liba.dylib libb.dylib c

DIR = $(patsubst %/,%,$(dir $(realpath $(firstword $(MAKEFILE_LIST)))))

default: $(TARGETS)

liba.dylib: dummy.o
	gcc -dynamiclib -install_name $(DIR)/$@ -o $@ $<

libb.dylib: dummy.o liba.dylib
	gcc -dynamiclib -install_name $(DIR)/$@ -L$(DIR) -la -o $@ $<

c: dummy.o libb.dylib
	gcc -L$(DIR) -lb -o $@ $<

dummy.o: dummy.c
	gcc -o $@ -c $<

clean:
	rm -f $(TARGETS) dummy.o

.PHONY: default clean
