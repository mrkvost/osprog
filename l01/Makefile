# fakt netusim, ako urobit, aby som bol schopny program spustit sam stylom ./program bez toho, aby som predtym definoval LD_LIBRARY_PATH=${PWD}
program:
	gcc -c -o number.o -fPIC number.c
	gcc -o libnumber.so -shared number.o
	gcc -c -o program.o -fPIC program.c
	gcc -o program program.o -lnumber -L${PWD}

clean:
	@rm *.o

distclean:
	@rm program *.so *.o

test:
	@a=$$(LD_LIBRARY_PATH=${PWD} ./program) || echo $$?
