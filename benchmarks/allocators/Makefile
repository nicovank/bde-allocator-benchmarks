OPTIM = -O3 -DBSLS_ASSERT_LEVEL_NONE -DNDEBUG -flto
GROWTH_SIZE = 20  # use 16 or less for testing

BSLSRC = ../..
BSL = $(BSLSRC)/groups/bsl
BDL = $(BSLSRC)/groups/bdl
BSLLIB = $(BSLSRC)/build/groups/bsl
BDLLIB = $(BSLSRC)/build/groups/bdl

INCLUDES = \
   -I$(BSL)/bsls -I$(BSL)/bslma -I$(BSL)/bslscm -I$(BSL)/bslh \
   -I$(BSL)/bsl+bslhdrs -I$(BSL)/bslstl -I$(BSL)/bslmf -I$(BSL)/bslalg \
   -I$(BDL)/bdlscm -I$(BDL)/bdlma

DEFS = -D_REENTRANT -D_POSIX_PTHREAD_SEMANTICS -DBSLS_IDENT_OFF
WAFCONFIGARGS = --abi-bits=64 --cpp-std=11 --build-type release

LIBS = \
   -Wl,-Bstatic \
   -L$(BDLLIB) -lbdl -L$(BSLLIB) -lbsl \
   -Wl,-Bdynamic \
   -lpthread

CFLAGS_BDE = $(OPTIM) $(DEFS) $(CFLAGS) -std=c99
CXXFLAGS_BDE = $(OPTIM) $(DEFS) $(CXXFLAGS)
CXXFLAGS_LOCAL = $(CXXFLAGS_BDE) -std=c++14 $(INCLUDES) $(CXXFLAGS)
LDFLAGS_LOCAL = $(LIBS) $(LDFLAGS)

all: run

BINARIES = growth growth-DS159long shgrowth \
           locality-AS1 locality-AS7 locality-AS9 locality-AS13 \
           zation tention

build: $(BINARIES)

$(BINARIES): bde-tag

OUTPUT =  results/*-result results/growth-*-* \
    results/shuffle results/schedule-AS1 results/schedule-AS7

clean:
	rm -rf $(BINARIES) bde-tag $(BSLSRC)/build growth-*-* *-result

reallyclean: clean
	rm -rf $(OUTPUT)

bde-tag: Makefile
	( \
	cd $(BSLSRC) && \
	export CFLAGS='$(CFLAGS_BDE)' && \
	export CXXFLAGS='$(CXXFLAGS_BDE)' && \
	waf configure $(WAFCONFIGARGS) && \
	waf build -j$(shell nproc) \
	)
	touch bde-tag

# section 7
growth: growth.cc allocont.h bde-tag
	$(CXX) -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

growth-DS159long: growth-DS159long.cc allocont.h bde-tag
	$(CXX) -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

shgrowth: shgrowth.cc allocont.h bde-tag
	$(CXX) -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

# section 8
locality-AS1: locality.cc allocont.h
	$(CXX) -DSTDALLOC -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)
locality-AS7: locality.cc allocont.h
	$(CXX) -DCTMULTI -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)
locality-AS9: locality.cc allocont.h
	$(CXX) -DRTMULTI -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)
locality-AS13: locality.cc allocont.h
	$(CXX) -DRTMULTIMONO -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

# section 9
zation: zation.cc
	$(CXX) -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

# section 10
tention: tention.cc
	$(CXX) -o $@ $(CXXFLAGS_LOCAL) $< $(LDFLAGS_LOCAL)

run: run-locality run-zation run-tention run-growth \
     run-shuffle run-schedule-AS1 run-schedule-AS7

run-growth: growth
	@echo With GROWTH_SIZE 20 this may take a full day to complete:
	SIZE=$(GROWTH_SIZE) ; \
	for i in 04 05 06 07 08 09 10 11 12 13 14 15 16; do \
           echo ./growth $$SIZE $$i - ; \
           (cd results; ../growth $$SIZE $$i - | tee "growth-$$SIZE-$$i") ; \
        done && \
	(cd results; cat growth-$$SIZE-* >growth-result ) && \
	(cd results; ./reduce-growth-results; rm T*; )

run-growth-DS159long: growth-DS159long
	@echo With GROWTH_SIZE 20 this may take a full day to complete:
	SIZE=$(GROWTH_SIZE) ; \
	for i in 04 05 06 07 08 09 10 11 12 13 14 15 16; do \
           echo ./growth-DS159long $$SIZE $$i - ; \
           mkdir -p results-DS159long; \
           (cd results-DS159long; ../growth-DS159long $$SIZE $$i - | tee "growth-$$SIZE-$$i") ; \
        done && \ 
	(cd results-DS159long; cat growth-$$SIZE-* >growth-result ) && \
	(cd results-DS159long; ./reduce-growth-results; rm T*; )

run-shgrowth: shgrowth
	@echo With GROWTH_SIZE 20 this may take a full day to complete:
	SIZE=$(GROWTH_SIZE) ; \
	for i in 04 05 06 07 08 09 10 11 12 13 14 15 16; do \
           echo ./shgrowth $$SIZE $$i - ; \
           (cd results; ../shgrowth $$SIZE $$i - | tee "growth-$$SIZE-$$i") ; \
        done && \
	(cd results; cat growth-$$SIZE-* >growth-result ) && \
	(cd results; ./reduce-growth-results; rm T* )

run-locality: locality-AS1 locality-AS7 locality-AS9 locality-AS13
	@( \
	echo "********** using AS1 default std::allocator:"; \
	./test-locality ./locality-AS1 2>&1; \
	echo "********** using AS7 compile-time-bound multipool"; \
	./test-locality ./locality-AS7 2>&1; \
	echo "********** using AS9 polymorphic multipool:"; \
	./test-locality ./locality-AS9 2>&1; \
	echo "********** using AS13 polymorphic multipool backed by monotonic:"; \
	./test-locality ./locality-AS13 2>&1; \
	) | tee >results/locality-result
	(cd results; ./reduce-locality-results)

run-shuffle: locality-AS1
	( \
    mkdir -p results/shuffle && cd results/shuffle && \
    for i in ../../test-shuffle-scripts/s*; do \
        o=$${i##*/} ; \
        $$i  2>&1 | tee o$${o%%_*} ; \
    done ; \
    )

run-schedule-AS1: locality-AS1
	( \
    mkdir -p results/schedule-AS1 && cd results/schedule-AS1 && \
    for i in ../../test-schedule-scripts/z*; do \
        o=$${i##*/} ; \
        $$i locality-AS1 2>&1 | tee o$${o} ; \
    done ; \
    )

run-schedule-AS7: locality-AS7
	( \
    mkdir -p results/schedule-AS7 && cd results/schedule-AS7 && \
    for i in ../../test-schedule-scripts/z*; do \
        o=$${i##*/} ; \
        $$i locality-AS7 2>&1 | tee o$${o} ; \
    done ; \
    )

run-zation: zation
	(ulimit -v 5000000; time ./test-zation 2>/dev/null | tee zation-result)
	mv zation-result results/

run-tention: tention
	time ./test-tention | tee tention-result
	mv tention-result results/
