verilator  ?= 1
top        ?= 0
rv32m      ?= 1
rv32e      ?= 0
rv32b      ?= 0
rv32c      ?= 0
debug      ?= 0
coverage   ?= 0
memsize    ?= 256

# Run flags
RFLAGS      = +trace $(if $(debug), +dump)

TARGET      = sim

ifeq (, $(shell which stdbuf))
STDBUF      =
else
STDBUF      = stdbuf -o0 -e0
endif

TARGET_SIM  = iverilog

ifeq ($(top),1)
    _top := 1
endif

ifeq ($(coverage),1)
    _coverage := 1
endif

ifeq ($(rv32m),1)
    _rv32m := 1
endif

ifeq ($(rv32e),1)
    _rv32e := 1
endif

ifeq ($(rv32b),1)
    _rv32b := 1
endif

ifeq ($(rv32c),1)
    _rv32c := 1
endif

ifeq ($(verilator),1)
BFLAGS      = -O3 -cc -Wall -sv -Wno-STMTDLY -Wno-UNUSED -Wno-TIMESCALEMOD -Wno-PINCONNECTEMPTY -Wno-UNSIGNED\
                -Wno-WIDTH -Wno-SYMRSVDWORD -Wno-UNDRIVEN \
              +define+MEMSIZE=$(memsize) \
              $(if $(_top), +define+SINGLE_RAM) \
              $(if $(_rv32m), +define+RV32M_ENABLED) \
              $(if $(_rv32e), +define+RV32E_ENABLED) \
              $(if $(_rv32b), +define+RV32B_ENABLED) \
              $(if $(_rv32c), +define+RV32C_ENABLED) \
              $(if $(_coverage), --coverage) \
              --trace-fst --Mdir sim_cc --build --exe sim_main.cpp getch.cpp elfloader.c
TARGET_SIM  = verilator
else
BFLAGS      = $(if $(_top), -D SINGLE_RAM=1) \
              $(if $(_rv32m), -DRV32M_ENABLED=1) \
              $(if $(_rv32e), -DRV32E_ENABLED=1) \
              $(if $(_rv32b), -DRV32B_ENABLED=1) \
              $(if $(_rv32c), -DRV32C_ENABLED=1)
endif

TOP_MODULE_FLAG = --top-module srv32_top

FILELIST    =  -f filelist.txt #$(if $(_top), ../rtl/top_s.v, ../rtl/srv32_top.sv)  GIVE FROM FILELIST.TXT

all: $(TARGET)

$(TARGET):
	CXXFLAGS=-DMEMSIZE=$(memsize) $(TARGET_SIM) $(BFLAGS) -o $(TARGET) $(FILELIST) 
	if [ "$(verilator)" = "1" ]; then \
        echo "Building with Verilator... "; \
		mv sim_cc/sim .; \
	fi

%.elf: $(TARGET) checkcode.awk
	if [ ! -f ../sw/$*/$*.elf ]; then \
		make -C ../sw $*; \
	fi
	$(STDBUF) ./$(TARGET) $(RFLAGS) ../sw/$*/$*.elf | awk -f $(filter %.awk, $^)
	@if [ -f coverage.dat ]; then \
		mv coverage.dat $*_cov.dat; \
	fi

clean:
	@$(RM) $(TARGET) wave.* trace.log dump.txt
	@$(RM) -rf sim_cc *_cov.dat

distclean: clean

