DIRS = analyses \
       ansi-c \
       assembler \
       big-int \
       cbmc \
       cpp \
       goto-analyzer \
       goto-cc \
       goto-checker \
       goto-diff \
       goto-instrument \
       goto-harness \
       goto-programs \
       goto-symex \
       jsil \
       json \
       json-symtab-language \
       langapi \
       linking \
       memory-analyzer \
       pointer-analysis \
       solvers \
       statement-list \
       symtab2gb \
       util \
       xmllang \
       # Empty last line

all: cbmc.dir \
     goto-analyzer.dir \
     goto-cc.dir \
     goto-diff.dir \
     goto-instrument.dir \
     goto-harness.dir \
     symtab2gb.dir \
     # Empty last line

ifeq ($(OS),Windows_NT)
  detected_OS := Windows
else
  detected_OS := $(shell sh -c 'uname 2>/dev/null || echo Unknown')
endif

ifeq ($(detected_OS),Linux)
  all: memory-analyzer.dir
else ifeq ($(WITH_MEMORY_ANALYZER),1)
  all: memory-analyzer.dir
endif

###############################################################################

util.dir: big-int.dir

# everything but big-int depends on util
$(patsubst %, %.dir, $(filter-out big-int util, $(DIRS))): util.dir

.PHONY: languages
.PHONY: clean

cpp.dir: ansi-c.dir linking.dir

languages: util.dir langapi.dir \
           cpp.dir ansi-c.dir xmllang.dir assembler.dir \
           jsil.dir json.dir json-symtab-language.dir statement-list.dir

solvers.dir: util.dir

goto-harness.dir: util.dir goto-programs.dir langapi.dir linking.dir \
                  json.dir json-symtab-language.dir \
                  goto-instrument.dir

goto-instrument.dir: languages goto-programs.dir pointer-analysis.dir \
                     goto-symex.dir linking.dir analyses.dir solvers.dir

goto-checker.dir: solvers.dir goto-symex.dir goto-programs.dir

cbmc.dir: languages solvers.dir goto-symex.dir analyses.dir \
          pointer-analysis.dir goto-programs.dir linking.dir \
          goto-instrument.dir goto-checker.dir

goto-analyzer.dir: languages analyses.dir goto-programs.dir linking.dir \
                   goto-instrument.dir goto-checker.dir

goto-diff.dir: languages goto-programs.dir pointer-analysis.dir \
               linking.dir analyses.dir goto-instrument.dir \
               solvers.dir goto-symex.dir

goto-cc.dir: languages goto-programs.dir linking.dir

memory-analyzer.dir: util.dir goto-programs.dir langapi.dir linking.dir \
	                   ansi-c.dir

symtab2gb.dir: util.dir goto-programs.dir langapi.dir linking.dir \
               json.dir json-symtab-language.dir

# building for a particular directory

$(patsubst %, %.dir, $(DIRS)):
	## Entering $(basename $@)
	$(MAKE) $(MAKEARGS) -C $(basename $@)

# generate source files

$(patsubst %, %_generated_files, $(DIRS)):
	$(MAKE) $(MAKEARGS) -C $(patsubst %_generated_files, %, $@) generated_files

generated_files: $(patsubst %, %_generated_files, $(DIRS))

# cleaning

clean: $(patsubst %, %_clean, $(DIRS))

$(patsubst %, %_clean, $(DIRS)):
	$(MAKE) $(MAKEARGS) -C $(patsubst %_clean, %, $@) clean ; \

# minisat2 and glucose download, for your convenience

DOWNLOADER = lwp-download
TAR = tar

minisat2-download:
	@echo "Downloading Minisat 2.2.1"
	@for i in $$(seq 1 3) ; do \
	  $(DOWNLOADER) \
	    http://ftp.debian.org/debian/pool/main/m/minisat2/minisat2_2.2.1.orig.tar.gz && \
	    exit 0 ; \
		$(RM) minisat2_2.2.1.orig.tar.gz ; \
	  if [ $$i -lt 3 ] ; then echo "Re-trying in 10 seconds" 1>&2 ; sleep 10 ; fi ; \
	done ; exit 1
	@$(TAR) xfz minisat2_2.2.1.orig.tar.gz
	@rm -Rf ../minisat-2.2.1
	@mv minisat2-2.2.1 ../minisat-2.2.1
	@(cd ../minisat-2.2.1; patch -p1 < ../scripts/minisat-2.2.1-patch)
	@rm minisat2_2.2.1.orig.tar.gz

cudd-download:
	@echo "Downloading Cudd 3.0.0"
	@$(DOWNLOADER) https://sourceforge.net/projects/cudd-mirror/files/cudd-3.0.0.tar.gz/download cudd-3.0.0.tar.gz
	@$(TAR) xfz cudd-3.0.0.tar.gz
	@$(RM) -r ../cudd-3.0.0
	@mv cudd-3.0.0 ../
	@$(RM) cudd-3.0.0.tar.gz
	@echo "Compiling Cudd 3.0.0"
	@(cd ../cudd-3.0.0; ./configure; $(MAKE))

glucose-download:
	@echo "Downloading glucose-syrup"
	@$(DOWNLOADER) http://www.labri.fr/perso/lsimon/downloads/softwares/glucose-syrup.tgz
	@$(TAR) xfz glucose-syrup.tgz
	@rm -Rf ../glucose-syrup
	@mv glucose-syrup ../
	@(cd ../glucose-syrup; patch -p1 < ../scripts/glucose-syrup-patch)
	@rm glucose-syrup.tgz

cadical_release = rel-06w
cadical-download:
	@echo "Downloading CaDiCaL $(cadical_release)"
	@curl -L https://github.com/arminbiere/cadical/archive/$(cadical_release).tar.gz | tar xz
	@rm -Rf ../cadical
	@mv cadical-$(cadical_release) ../cadical
	@(cd ../cadical; patch -p1 < ../scripts/cadical-patch)
	@cd ../cadical && CXX=$(CXX) CXXFLAGS=-O3 ./configure --debug && make

doc :
	doxygen

.PHONY: minisat2-download cudd-download glucose-download cadical-download
