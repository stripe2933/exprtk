# Add leading <padchar> to string, the result string has length of given <length> parameter.
# Usage: pad_string(<output> <str> <padchar> <length>)
function(pad_string output str padchar length)
    string(LENGTH "${str}" _strlen)
    math(EXPR _strlen "${length} - ${_strlen}")

    if(_strlen GREATER 0)
        string(REPEAT ${padchar} ${_strlen} _pad)
        string(PREPEND str ${_pad})
    endif()

    set(${output} "${str}" PARENT_SCOPE)
endfunction()

foreach(example_no RANGE 1 19)
    pad_string(example_no ${example_no} "0" 2)
    add_executable(exprtk_simple_example_${example_no} exprtk_simple_example_${example_no}.cpp)
    target_link_libraries(exprtk_simple_example_${example_no} PRIVATE exprtk)
endforeach()