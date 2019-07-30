include_vendor_pkg(grpc)

function (compile_proto proto_file)
    get_filename_component(proto "${proto_file}" ABSOLUTE)
    get_filename_component(proto_path "${proto}" PATH)
    get_filename_component(proto_name "${proto}" NAME_WE)

    file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${proto_name})
    set(proto_srcs "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}/${proto_name}.pb.cc")
    set(proto_hdrs "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}/${proto_name}.pb.h")
    set(grpc_srcs "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}/${proto_name}.grpc.pb.cc")
    set(grpc_hdrs "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}/${proto_name}.grpc.pb.h")

    add_custom_command(
            OUTPUT "${proto_srcs}" "${proto_hdrs}" "${grpc_srcs}" "${grpc_hdrs}"
            COMMAND ${_PROTOBUF_PROTOC}
            ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}"
            --cpp_out "${CMAKE_CURRENT_BINARY_DIR}/${proto_name}"
            -I "${proto_path}"
            --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
            "${proto}"
            DEPENDS "${proto}")

    add_library(${proto_name} STATIC
            ${proto_srcs}
            ${proto_hdrs}
            ${grpc_srcs}
            ${grpc_hdrs}
            )

    target_include_directories(${proto_name} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/${proto_name})

    add_dependencies(${proto_name} protoc)

    target_link_libraries(${proto_name} PUBLIC libprotobuf grpc++)
endfunction()

file(GLOB_RECURSE PROTO_INPUT_FILES ${CMAKE_CURRENT_LIST_DIR}/*.proto)
foreach (FILE ${PROTO_INPUT_FILES})
    message(STATUS "Adding target to compile ${FILE}")
    compile_proto(${FILE})
endforeach()