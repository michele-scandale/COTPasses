set(HOST_OS ${CMAKE_SYSTEM_NAME})
set(HOST_ARCH ${CMAKE_SYSTEM_PROCESSOR})
set(SHLIBEXT "${LTDL_SHLIB_EXT}")
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.in
  ${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg
  @ONLY
)

add_lit_testsuite(mycheck "Running the COTPasses tests"
  ${CMAKE_CURRENT_BINARY_DIR}
  PARAMS llvm_site_config=${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg
  DEPENDS COTPasses
)
set_target_properties(mycheck PROPERTIES FOLDER "Tests")
