include_vendor_pkg(grpc)

add_executable(main main.cpp)
target_link_libraries(main greeting_service)
