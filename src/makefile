###################################################################
###################################################################
#
# Basic makefile to compile the circulation downscaling code.  
# 
# To compile this code on your machine, edit the NCDF_PATH variable and the fortran compiler
#
# To build the model simply run make.  
# 
# The primary user facing rules are: 
# 	all		: builds the model, this is the default
# 	clean	: remove any build objects (not executables)
#  	allclean: make clean and remove executables
# 	install	: make the executable and copy it to ~/bin (or other install dir)
# 	test	: build various test executables
#
###################################################################
###################################################################

FC=gfortran
NCDF_PATH = /usr/local
INSTALL_DIR = ~/bin
RM=/bin/rm
CP=/bin/cp
# LIBNETCDF = -L$(NCDF_PATH)/lib -lnetcdff -lnetcdf
# INCNETCDF = -I$(NCDF_PATH)/include
BUILD=build/

###################################################################
###################################################################
#
#	Try to identify the computer we are running on and set up library paths
#
###################################################################
###################################################################
NODENAME := $(shell uname -n)

ifeq ($(NODENAME), Patthar.local)
	NODENAME=Nomad.local
endif
ifeq ($(NODENAME), pakaldi.local)
	NODENAME=Nomad.local
endif
ifeq ($(patsubst vpn%.ucar.edu,vpn.ucar.edu,$(NODENAME)), vpn.ucar.edu)
	NODENAME=Nomad.local
endif

ifeq ($(NODENAME), Nomad.local)
	FC=gfortran
	NCDF_PATH=/usr/local
endif

ifeq ($(NODENAME), hydro-c1)
	FC=ifort
	# FC=gfortran
	ifeq ($(FC), ifort)
		NCDF_PATH = /opt/netcdf4-intel
	endif
	ifeq ($(FC), gfortran)
		NCDF_PATH = /opt/netcdf4-gcc
	endif
endif

# on yellowstone:
ifeq ($(LMOD_FAMILY_COMPILER),gnu)
	FC=gfortran
	NCDF_PATH=/glade/apps/opt/netcdf/4.3.0/gnu/4.8.2
	# LIBNETCDF = $(LIB_NCAR) # when netcdf includes are setup by the yellowstone module system
	# INCNETCDF = $(INC_NCAR)
	# LIBNETCDF = -Wl,-rpath,$(NCDF_PATH)/lib -L$(NCDF_PATH)/lib -lnetcdff -lnetcdf # if using a compiler for which netcdf includes are
	# INCNETCDF = -I$(NCDF_PATH)/include # NOT setup correctly by the yellowstone module system
endif
ifeq ($(LMOD_FAMILY_COMPILER),intel)
	FC=ifort
	NCDF_PATH=/glade/apps/opt/netcdf/4.3.0/intel/default
	# LIBNETCDF = $(LIB_NCAR) #-L$(NCDF_PATH)/lib -lnetcdff -lnetcdf
	# INCNETCDF = $(INC_NCAR) #-I$(NCDF_PATH)/include # netcdf includes are setup by the yellowstone module system
endif
ifeq ($(LMOD_FAMILY_COMPILER),pgi)
	FC=pgf90
	NCDF_PATH=/glade/apps/opt/netcdf/4.3.0/pgi/default
	# LIBNETCDF = -rpath $(NCDF_PATH)/lib -L$(NCDF_PATH)/lib -lnetcdff -lnetcdf # if using a compiler for which netcdf includes are
	# INCNETCDF = -I$(NCDF_PATH)/include # NOT setup correctly by the yellowstone module system
endif

# get GIT version info
GIT_VERSION := $(shell git describe --long --dirty --all --always | sed -e's/heads\///')

###################################################################
###################################################################
#
#	Compiler specific compilation flags
#
###################################################################
###################################################################
ifeq ($(FC), gfortran)
	COMP=-fopenmp -lgomp -O3 -ffree-line-length-none -ftree-vectorize -fimplicit-none -funroll-loops -march=native -fno-protect-parens -ffast-math #-flto #
	LINK=-fopenmp -lgomp
	PREPROC=-cpp
	MODOUTPUT=-J $(BUILD)
endif
# Intel fortran
ifeq ($(FC), ifort)
	COMP= -u -qopenmp -liomp5 -O3 -no-prec-div -xHost -ftz -fpe0
	LINK= -qopenmp -liomp5
	PREPROC=-fpp
	MODOUTPUT=-module $(BUILD)
endif
# PGI fortran
ifeq ($(FC), pgf90)
	COMP=-O2 -mp -Mdclchk #-fast -O3 -mp -c -Mdclchk
	LINK=-mp
	PREPROC=-Mpreprocess
	MODOUTPUT=-module $(BUILD)
endif

# Cray fortran
ifeq ($(FC), ftn)
	COMP= -h omp vector2 -O2 -eI
	LINK= -fopenmp
	PREPROC= -eZ
	MODOUTPUT= -J $(BUILD) -em
endif

###################################################################
###################################################################
#
# Various compiling options.  Set the MODE variable with "make MODE=debugslow" etc.
#
###################################################################
###################################################################
ifeq ($(MODE), debugslow)
	ifeq ($(FC), ifort)
		COMP= -debug -debug-parameters all -traceback -ftrapuv -g -fpe0 -u -check all -check noarg_temp_created -CB
		LINK=
	endif
	ifeq ($(FC), gfortran)
		COMP= -g -Wall -fbounds-check -fbacktrace -finit-real=nan -ffree-line-length-none
		LINK=
	endif
	ifeq ($(FC), pgf90)
		COMP= -g -Mbounds -Mlist -Minfo  -Mdclchk
		LINK=
	endif
	ifeq ($(FC), ftn)
		COMP=-h noomp -g -m 0 -R abcsp
		LINK=-h noomp
		PREPROC=-eZ
		MODOUTPUT=-e m -J $(BUILD)
	endif
endif
ifeq ($(MODE), debug)
	ifeq ($(FC), ifort)
		COMP= -debug -O1 -u -check all -check noarg_temp_created -traceback -fpe0 -fast-transcendentals -xhost
		LINK=
	endif
	ifeq ($(FC), gfortran)
		COMP= -O1 -g -fbounds-check -fbacktrace -finit-real=nan -ffree-line-length-none
		LINK=
	endif
	ifeq ($(FC), pgf90)
		COMP= -gopt -O1 -Mbounds -Mlist -Minfo  -Mdclchk
		LINK=
	endif
	ifeq ($(FC), ftn)
		COMP=-O1 -h noomp -g
		LINK=-h noomp
		PREPROC=-eZ
		MODOUTPUT=-e m -J $(BUILD)
	endif
endif
ifeq ($(MODE), debugompslow)
	ifeq ($(FC), ifort)
		# COMP= -qopenmp -liomp5 -debug -debug-parameters all -traceback -ftrapuv -g -fpe0 -c -u -check all -check noarg_temp_created -CB
		COMP= -qopenmp -liomp5 -debug -u	-fpe0 -traceback -check all -check noarg_temp_created -fp-stack-check
		LINK= -qopenmp -liomp5
	endif
	ifeq ($(FC), gfortran)
		COMP= -fopenmp -lgomp -g -fbounds-check -fbacktrace -finit-real=nan -ffree-line-length-none
		LINK= -fopenmp -lgomp
	endif
	ifeq ($(FC), pgf90)
		COMP= -g -Mbounds -Mlist -Minfo -mp -Mdclchk
		LINK= -mp
	endif
	ifeq ($(FC), ftn)
		COMP= -g -m 0 -R abcsp
		LINK=
		PREPROC=-eZ
		MODOUTPUT=-e m -J $(BUILD)
	endif
endif
ifeq ($(MODE), debugomp)
	ifeq ($(FC), ifort)
		COMP= -qopenmp -liomp5 -debug -O3 -u -traceback -fpe0 -ftz -fast-transcendentals  -xHost #-check all -check noarg_temp_created -fpe0 -fast-transcendentals -xhost
		LINK= -qopenmp -liomp5
	endif
	ifeq ($(FC), gfortran)
		COMP= -fopenmp -lgomp -O1 -g -fbounds-check -fbacktrace -finit-real=nan -ffree-line-length-none
		LINK= -fopenmp -lgomp
	endif
	ifeq ($(FC), pgf90)
		COMP= -g -O1 -Mbounds -Mlist -Minfo -mp -Mdclchk
		LINK= -mp
	endif
	ifeq ($(FC), ftn)
		COMP=-O1 -g
		LINK=
		PREPROC=-eZ
		MODOUTPUT=-e m -J $(BUILD)
	endif
endif

PROF=
ifeq ($(MODE), profile)
	ifeq ($(FC), ifort)
		PROF=-pg -debug inline-debug-info -shared-intel
		COMP= -u -qopenmp -liomp5 -O3 -no-prec-div -xHost -ftz #because -fast includes -static # not available in ifort <13 -align array64byte
	endif
	ifeq ($(FC), gfortran)
		PROF=-g
	endif
endif
ifeq ($(MODE), fast) # WARNING -ipo (included in -fast) takes forever for very little gain, and this may be unstable
	ifeq ($(FC), ifort)
		COMP= -u -qopenmp -liomp5 -fast -ftz #-fast-transcendentals # not available in ifort <13: -align array64byte
	endif
endif

###################################################################
###################################################################
#
# Should not need to edit anything below this line
#
###################################################################
###################################################################
# copy required libraries into a directory accessible on compute nodes and set LD_RUN_PATH e.g.
# export LD_RUN_PATH=$LD_RUN_PATH:/path/to/netcdf/libraries/lib:/path/to/fftw/libraries/lib

# flags forall
FCFLAGS = $(COMP) -I$(NCDF_PATH)/include -I$(BUILD)
# libraries needed for linking
LDFLAGS = $(LINK) -L$(NCDF_PATH)/lib -lnetcdf -lnetcdff

OBJS =  \
	$(BUILD)time.o            \
	$(BUILD)string.o          \
	$(BUILD)data_structures.o \
	$(BUILD)geo_reader.o      \
	$(BUILD)io_routines.o     \
	$(BUILD)gcm.o    		  \
	$(BUILD)reanalysis.o      \
	$(BUILD)observations.o    \
	$(BUILD)configuration.o   \
	$(BUILD)main.o
	# $(BUILD)ds_model.o       	\
	# $(BUILD)numerics.o     	\
	# $(BUILD)coefficients.o    \
	

CAL_TEST_OBJS =    				\
    $(BUILD)time.o 				\
	$(BUILD)test_calendar.o
	
MASTERFILE_TEST_OBJS =    		\
	$(BUILD)time.o		 		\
	$(BUILD)string.o	 		\
	$(BUILD)data_structures.o 	\
	$(BUILD)master_file.o       \
	$(BUILD)test_master_file.o

# These are the primary user facing rules
#
# make [all, test, install, clean, allclean]
#
# "make" builds all
all: downscale

test: test_calendar test_master_file

install: downscale
	cp downscale $(INSTALL_DIR)/

clean:
	$(RM) $(BUILD)* 2>/dev/null ||:

allclean:clean
	$(RM) downscale test_calendar 2>/dev/null ||:

# higher level rules
downscale: $(OBJS)
	$(FC) $(FCFLAGS) -o $@ $^ $(LDFLAGS)

test_calendar : $(CAL_TEST_OBJS)
	$(FC) $(FCFLAGS) -o $@ $^ $(LDFLAGS)

test_master_file : $(MASTERFILE_TEST_OBJS)
	$(FC) $(FCFLAGS) -o $@ $^ $(LDFLAGS)


# specify specific builds and dependencies. 
$(BUILD)time.o: utilities/time.f90
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)test_calendar.o: tests/test_calendar.f90 $(BUILD)time.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)test_master_file.o: tests/test_master_file.f90 $(BUILD)master_file.o \
							$(BUILD)string.o $(BUILD)time.o $(BUILD)data_structures.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)data_structures.o: objects/data_structures.f90 $(BUILD)time.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@
	
$(BUILD)string.o: utilities/string.f90
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)master_file.o: objects/master_file.f90 $(BUILD)string.o $(BUILD)time.o $(BUILD)data_structures.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)io_routines.o: io/io_routines.f90
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)configuration.o: objects/configuration.f90 $(BUILD)data_structures.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)geo_reader.o: utilities/geo_reader.f90 $(BUILD)data_structures.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)gcm.o: objects/gcm.f90 $(BUILD)data_structures.o $(BUILD)geo_reader.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)observations.o: objects/observations.f90 $(BUILD)data_structures.o $(BUILD)geo_reader.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)reanalysis.o: objects/reanalysis.f90 $(BUILD)data_structures.o $(BUILD)geo_reader.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@

$(BUILD)main.o: main/main.f90 $(BUILD)gcm.o $(BUILD)reanalysis.o $(BUILD)observations.o $(BUILD)configuration.o
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@


$(BUILD)%.o: %.f90
	$(FC) $(FCFLAGS) -c $< $(MODOUTPUT) -o $@
