# **************************************************************************
# * Copyright(c) 1998-2016, ALICE Experiment at CERN, All rights reserved. *
# *                                                                        *
# * Author: The ALICE Off-line Project.                                    *
# * Contributors are mentioned in the code where appropriate.              *
# *                                                                        *
# * Permission to use, copy, modify and distribute this software and its   *
# * documentation strictly for non-commercial purposes is hereby granted   *
# * without fee, provided that the above copyright notice appears in all   *
# * copies and that both the copyright notice and this permission notice   *
# * appear in the supporting documentation. The authors make no claims     *
# * about the suitability of this software for any purpose. It is          *
# * provided "as is" without express or implied warranty.                  *
# **************************************************************************

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

project(DPMJET)
enable_language(Fortran)

# Set by default DEBUG build type
# The optimisation affects the initialisation of COMMON blocks
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE DEBUG)
endif(NOT CMAKE_BUILD_TYPE)

cmake_policy(SET CMP0042 NEW)

# Use cmake ... -DFLUPRO=<directory containing libflukahp.a> ...
# if you want to build DPMJEt with Fluka support
if(FLUPRO)
  link_directories(${FLUPRO})
  set ( FSRCS dpmjet3.0-5F.f phojet1.12-35c4.f user3.0-5F.f openfile.f)
else(FLUPRO)
  set ( FSRCS dpmjet3.0-5.f phojet1.12-35c3.f user3.0-5.f openfile.f)
endif(FLUPRO)

# Add specific version of PYTHIA6 files to the library
set(FSRCS ${FSRCS}
        pythia6_common_block_address.F
        tpythia6_called_from_cc.F
        pythia6115dpm3.f
        pdf_alice.F
        pydummy.f
    )

# Additional C sources needed by PYTHIA6
set(CSRCS
    main.c
    pythia6_common_address.c
   )

# Add a library to the project using the specified source files
add_library(${PROJECT_NAME} SHARED ${FSRCS} ${CSRCS})
if(FLUPRO)
  target_link_libraries(${PROJECT_NAME} flukahp)
endif(FLUPRO)

# System dependent: Modify the way the library is build
if(${CMAKE_SYSTEM} MATCHES Darwin)
    set(CMAKE_Fortran_FLAGS "-undefined dynamic_lookup")
endif(${CMAKE_SYSTEM} MATCHES Darwin)

# Installation
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)

# Install data files
install(FILES nuclear.bin DESTINATION data)
install(FILES fitpar.dat DESTINATION data)
install(FILES dpmjet.dat DESTINATION data)
install(DIRECTORY inp DESTINATION data FILES_MATCHING PATTERN "*.dat")
install(DIRECTORY doc DESTINATION .)
