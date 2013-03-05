SH_LIKE  = sh ksh bash zsh
CSH_LIKE = csh tcsh
OTHER    = perl python lisp
D =  $(shell pwd)

ALL_SHELLS = $(SH_LIKE) $(CSH_LIKE) $(OTHER)

all: $(ALL_SHELLS) modulerc

%: %.in Makefile
	perl -MCwd -MFile::Basename -pe 'BEGIN {$$d=dirname(cwd)} s|\$$MODULESHOME|$$d|g' $< > $@

clean:
	rm -f $(ALL_SHELLS) modulerc

modulerc:
	echo module use $(MODULESHOME)/modulefiles > modulerc
	echo "module use ~/.modules" >> modulerc