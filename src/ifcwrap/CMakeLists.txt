################################################################################
#                                                                              #
# This file is part of IfcOpenShell.                                           #
#                                                                              #
# IfcOpenShell is free software: you can redistribute it and/or modify         #
# it under the terms of the Lesser GNU General Public License as published by  #
# the Free Software Foundation, either version 3.0 of the License, or          #
# (at your option) any later version.                                          #
#                                                                              #
# IfcOpenShell is distributed in the hope that it will be useful,              #
# but WITHOUT ANY WARRANTY; without even the implied warranty of               #
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the                 #
# Lesser GNU General Public License for more details.                          #
#                                                                              #
# You should have received a copy of the Lesser GNU General Public License     #
# along with this program. If not, see <http://www.gnu.org/licenses/>.         #
#                                                                              #
################################################################################

FIND_PACKAGE(SWIG)
IF(NOT SWIG_FOUND)
	MESSAGE(FATAL_ERROR "BUILD_IFCPYTHON enabled, but unable to find SWIG. Disable BUILD_IFCPYTHON or fix SWIG paths to proceed.")
ENDIF()

INCLUDE(${SWIG_USE_FILE})

IF(NOT "$ENV{PYTHON_INCLUDE_DIR}" STREQUAL "")
	SET(PYTHON_INCLUDE_DIR $ENV{PYTHON_INCLUDE_DIR} CACHE FILEPATH "Python header files")
	MESSAGE(STATUS "Looking for Python header files in: ${PYTHON_INCLUDE_DIR}")
ENDIF()
IF(NOT "$ENV{PYTHON_LIBRARY}" STREQUAL "")
	SET(PYTHON_LIBRARY $ENV{PYTHON_LIBRARY} CACHE FILEPATH "Python library file")
	MESSAGE(STATUS "Looking for Python library file in: ${PYTHON_LIBRARY}")
ENDIF()

FIND_PACKAGE(PythonLibs)
IF(NOT PYTHONLIBS_FOUND)
	MESSAGE(FATAL_ERROR "BUILD_IFCPYTHON enabled, but unable to find Python. Disable BUILD_IFCPYTHON or fix Python paths to proceed.")
ENDIF()

INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

SET(CMAKE_SWIG_FLAGS "")
# NOTE Workaround for most likely missing debug Python libraries on Windows (requires Python built from the source).
# Python 3.5 intaller and onwards will have an option to install the debug libraries too.
IF (WIN32 AND NOT PYTHON_DEBUG_LIBRARIES)
	MESSAGE(STATUS "PYTHON_DEBUG_LIBRARIES not found, defining SWIG_PYTHON_INTERPRETER_NO_DEBUG workaround for IfcWrap.")
	ADD_DEFINITIONS(-DSWIG_PYTHON_INTERPRETER_NO_DEBUG)
ENDIF()

SET_SOURCE_FILES_PROPERTIES(IfcPython.i PROPERTIES CPLUSPLUS ON)
SWIG_ADD_MODULE(ifcopenshell_wrapper python IfcPython.i)
SWIG_LINK_LIBRARIES(ifcopenshell_wrapper IfcParse IfcGeom ${PYTHON_LIBRARIES} ${OPENCASCADE_LIBRARIES} ${ICU_LIBRARIES})

# To install IfcPython let's get the site-packages dir from python
EXECUTE_PROCESS(COMMAND
	python -c "import sys; from distutils.sysconfig import get_python_lib; sys.stdout.write(get_python_lib())"
	OUTPUT_VARIABLE python_package_dir)

IF("${python_package_dir}" STREQUAL "")
	MESSAGE(FATAL_ERROR "Python executable not in PATH, aborting")
ENDIF()

INSTALL(FILES 
		"${CMAKE_BINARY_DIR}/ifcwrap/ifcopenshell_wrapper.py"
		"${CMAKE_CURRENT_SOURCE_DIR}/../ifcopenshell-python/ifcopenshell/__init__.py"
		"${CMAKE_CURRENT_SOURCE_DIR}/../ifcopenshell-python/ifcopenshell/guid.py"
	DESTINATION "${python_package_dir}/ifcopenshell")
INSTALL(FILES
		"${CMAKE_CURRENT_SOURCE_DIR}/../ifcopenshell-python/ifcopenshell/geom/__init__.py"
		DESTINATION "${python_package_dir}/ifcopenshell/geom")
INSTALL(TARGETS _ifcopenshell_wrapper DESTINATION "${python_package_dir}/ifcopenshell")
