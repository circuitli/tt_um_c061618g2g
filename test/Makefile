# Makefile
# See https://docs.cocotb.org/en/stable/quickstart.html for more info

# defaults
SIM ?= icarus
FST ?= -fst # Use more efficient FST format
TOPLEVEL_LANG ?= verilog
SRC_DIR = $(PWD)/../src
MACROS_DIR     := macros
PROJECT_SOURCES = project.v
#COMPILE_ARGS += -g2012
# If your includes are inside 'src/'

# Dynamically compute the absolute path of the subproject from the test directory
SUBPROJECT_ROOT = $(abspath $(PWD)/../deps/c061618g2)
VERILOG_INCLUDE_DIRS += $(SUBPROJECT_ROOT)

# =========================================================================
# PRECISE SYSTEMVERILOG EXTENSION SCAN FIX
# This explicitly forces Icarus to search for and parse '.v' files 
# when scanning your test directories, ensuring 'tb.v' is discovered!
# =========================================================================
COMPILE_ARGS += -Y .v

# Fallback to your local formal project copy if PDK_ROOT isn't defined on Windows
#PDK_ROOT ?= $(PWD)/formal
RAW_PDK_PRIMITIVES = $(PDK_ROOT)/gf180mcuD/libs.ref/gf180mcu_fd_sc_mcu7t5v0/verilog/primitives.v
VERILOG_SOURCES += $(PDK_ROOT)/gf180mcuD/libs.ref/gf180mcu_fd_sc_mcu7t5v0/verilog/gf180mcu_fd_sc_mcu7t5v0.v

ifeq ($(SIM),verilator)
    # 2. Feed the search search paths directly into Verilator's native -y system.
    # This instructs Verilator to look across the entire OpenLane PDK directory tree 
    # to automatically locate sg13g2.v and resolve the decoupling cells on its own.
    COMPILE_ARGS += -y $(PWD) -y $(PWD)/../src
    COMPILE_ARGS += -y $(PDK_ROOT)/gf180mcuD/libs.ref/gf180mcu_fd_sc_mcu7t5v0/verilog
    COMPILE_ARGS += -y /home/runner/.openlane/pdk/gf180mcuD/libs.ref/gf180mcu_fd_sc_mcu7t5v0/verilog
    COMPILE_ARGS += -v $(PDK_ROOT)/gf180mcuD/libs.ref/gf180mcu_fd_pr/verilog/gf180mcu_fd_pr.v
    COMPILE_ARGS += -v $(RAW_PDK_CELLS)
    COMPILE_ARGS += -v $(RAW_PDK_PRIMITIVES)


    # 3. CRITICAL FLAGS: Add -y to let Verilator discover your module dependencies,
    # and turn off MODDUP to ignore duplicate standard cell models if they overlap.
    COMPILE_ARGS += --timing -y $(PWD) -y $(PWD)/../src
    COMPILE_ARGS += -Wno-fatal -Wno-UNOPTFLAT -Wno-BLKANDNBLK -Wno-MODDUP -Wno-PINMISSING
else ifeq ($(SIM),icarus)
    COMPILE_ARGS += -DCOCOTB_SIM
    # This allows Icarus to load real nanosecond timing data from the PDK.
    # We pass the default typical timing corner resolution mapping:
    COMPILE_ARGS += -Ttyp
    # =========================================================================
    # TIMESCALE RESOLUTION INJECTION
    # This format uses the '+' syntax. It bypasses 'invalid option -- -' errors
    # and locks the 1ps precision grid globally for BOTH RTL and GLS flows.
    # =========================================================================
    #COMPILE_ARGS += -timescale 1ns/1ps

    # Define filenames
    CLEAN_LOCAL_CELLS = $(CURDIR)/sg13g2_stdcell_functional.v

    # 1. Create a local functional copy with the problematic lines deleted
    #$(shell sed '/ifnone/d' $(RAW_PDK_CELLS) > $(CLEAN_LOCAL_CELLS))

    ifneq ($(GATES),yes)
        # RTL simulation:
        SIM_BUILD		= sim_build/rtl
        
        #2. Append compilation macros
        COMPILE_ARGS += -DFUNCTIONAL -Dfunctional_primitive -gno-specify

        VERILOG_SOURCES += $(addprefix $(SRC_DIR)/,$(PROJECT_SOURCES))

        # Append the path to your macro's internal gate-level netlist file
        #VERILOG_SOURCES += $(wildcard $(PWD)/../macros/nl/*.v)

        VERILOG_SOURCES += $(RAW_PDK_PRIMITIVES)
        VERILOG_SOURCES += $(RAW_PDK_CELLS)

        #VERILOG_SOURCES += $(CLEAN_LOCAL_CELLS)
    else
        # Gate level simulation:
        SIM_BUILD		= sim_build/gl
        COMPILE_ARGS    += -DGL_TEST
        COMPILE_ARGS    += -DFUNCTIONAL -Dfunctional_primitive
        COMPILE_ARGS    += -DUSE_POWER_PINS
        COMPILE_ARGS    += -DSIM
        COMPILE_ARGS    += -DUNIT_DELAY=\#1

        # Append the path to your macro's internal gate-level netlist file
        #VERILOG_SOURCES += $(wildcard $(PWD)/../macros/nl/*.v)

        # 3. Strip the raw PDK path from the sources and append our clean local version instead
        #VERILOG_SOURCES := $(filter-out $(RAW_PDK_CELLS) %sg13g2_stdcell.v, $(VERILOG_SOURCES))
        #VERILOG_SOURCES += $(CLEAN_LOCAL_CELLS)
        VERILOG_SOURCES += $(RAW_PDK_PRIMITIVES)
        VERILOG_SOURCES += $(RAW_PDK_CELLS)

        # this gets copied in by the GDS action workflow
        VERILOG_SOURCES += $(PWD)/gate_level_netlist.v
    endif
endif

# Allow sharing configuration between design and testbench via `include`:
COMPILE_ARGS 		+= -I..  -I$(SRC_DIR)

# Include the testbench sources:
VERILOG_SOURCES += $(PWD)/tb.v
TOPLEVEL = tb
COCOTB_TOPLEVEL = tb

# List test modules to run, separated by commas and without the .py suffix:
MODULE              = test
COCOTB_TEST_MODULES = test

# include cocotb's make rules to take care of the simulator setup
include $(shell cocotb-config --makefiles)/Makefile.sim
