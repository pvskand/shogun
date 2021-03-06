# Check python dependency and return error
# Generate examples
# Check which interfaces are turned on
# Add CMakeLists file for each turned on interface

# Disabled (temporary) as non-portable solution. This is replaced by allowing any [a-zA-Z_][a-zA-Z0-9_]* as shogun keyword
# Generate type list
# execute_process(COMMAND sh ${CMAKE_CURRENT_SOURCE_DIR}/generator/types/get_type_list.sh > ${CMAKE_CURRENT_SOURCE_DIR}/generator/types/typelist)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/parser_files)

# Run example generation script
add_custom_target(meta_examples
    COMMAND ${PYTHON_EXECUTABLE}
    ${CMAKE_CURRENT_SOURCE_DIR}/generator/generate.py
    -i ${CMAKE_CURRENT_SOURCE_DIR}/src
    -o ${CMAKE_CURRENT_BINARY_DIR}
    -t ${CMAKE_CURRENT_SOURCE_DIR}/generator/targets
    -g ${CTAGS_FILE}
    --store-vars
    --parser_files_dir=${CMAKE_CURRENT_BINARY_DIR}/parser_files
    COMMENT "Generating examples from meta-language"
    DEPENDS ctags)

# get list of meta examples that can be built
INCLUDE(FindMetaExamples)
find_meta_examples()

# If testing is enabled we run the examples
IF(ENABLE_TESTING)
    IF (NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/data)
        # copy data symlink to cmake binary dir so that we can execute meta examples in there
        EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E create_symlink "${CMAKE_CURRENT_SOURCE_DIR}/data" "${CMAKE_CURRENT_BINARY_DIR}/data")
    ENDIF()
                                                              

    # c++
    IF (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/cpp AND NOT ${TRAVIS_DISABLE_META_CPP})
        IF(HAVE_CXX0X OR HAVE_CXX11)
            add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/cpp)
        ELSE()
            MESSAGE(WARNING "Skipping C++ meta examples; requires a c++11 compatible compiler.")
        ENDIF()
    ENDIF()

    # Python
    IF (PythonModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/python)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/python)
    ENDIF()

    # java
    IF (JavaModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/java)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/java)
    ENDIF()

    # octave
    IF (OctaveModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/octave)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/octave)
    ENDIF()

    # r
    IF (RModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/r)
        # temporarily disabled as R and static calls has to be fixed
        #add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/r)
    ENDIF()
    
    # csharp
    IF (CSharpModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/csharp)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/csharp)
    ENDIF()
    
    # ruby
    IF (RubyModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/ruby)
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/ruby)
    ENDIF()

    # lua
    IF (LuaModular AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/lua)
        # temporarily removed since lua modular currently
        # does not support overloaded c++ methods
        # see https://github.com/shogun-toolbox/shogun/issues/3018
        #add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/lua)
    ENDIF()
ENDIF()
