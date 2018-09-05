cmake_minimum_required(VERSION 3.10)

project(manual NONE)

# -- set UseLATEX options ------------------------------------------------------

set(LATEX_USE_SYNCTEX yes)

# -- list all .tex files -------------------------------------------------------

set(sources
  Actors.tex
  Brokers.tex
  CommonPitfalls.tex
  ConfiguringActorApplications.tex
  Error.tex
  FAQ.tex
  FirstSteps.tex
  GroupCommunication.tex
  Introduction.tex
  ManagingGroupsOfWorkers.tex
  MessageHandlers.tex
  MessagePassing.tex
  Messages.tex
  MigrationGuides.tex
  NetworkTransparency.tex
  OpenCL.tex
  ReferenceCounting.tex
  Registry.tex
  RemoteSpawn.tex
  Scheduler.tex
  Streaming.tex
  TypeInspection.tex
  UsingAout.tex
  Utility.tex
)

# -- process .tex.in files -----------------------------------------------------

configure_file("cmake/variables.tex.in"
               "${CMAKE_CURRENT_BINARY_DIR}/variables.tex"
               @ONLY)

# -- add manual.pdf as target --------------------------------------------------

add_latex_document(
  manual.tex
  INPUTS ${sources} "variables.tex"
  IMAGE_DIRS "../pdf"
  FORCE_PDF
  TARGET_NAME manual
)

# -- pandoc setup for generating rst files -------------------------------------

if(NOT EXISTS ${PANDOC_EXECUTABLE})
  find_program(PANDOC_EXECUTABLE pandoc)
endif()

add_custom_target(rst)

macro(generate_rst texfile)
  get_filename_component(rstfile_we "${texfile}" NAME_WE)
  set(rstfile "${rstfile_we}.rst")
  add_custom_target("${rstfile}"
                    DEPENDS "${texfile}"
                    COMMAND ${PANDOC_EXECUTABLE}
                            "--filter=${CMAKE_CURRENT_SOURCE_DIR}/filter.py"
                            --wrap=none -f latex
                            -o "${CMAKE_CURRENT_BINARY_DIR}/../rst/${rstfile}"
                            "${CMAKE_CURRENT_BINARY_DIR}/${texfile}"
                    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}")
  add_dependencies(rst "${rstfile}")
endmacro()

if(NOT EXISTS ${PANDOC_EXECUTABLE})
  message(STATUS "Pandoc not found. Install Pandoc or set PANDOC_EXECUTABLE.")
else()
  file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/../rst")
  foreach(texfile ${sources})
    generate_rst(${texfile})
  endforeach()
endif()
