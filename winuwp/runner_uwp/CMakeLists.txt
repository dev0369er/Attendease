cmake_minimum_required (VERSION 3.8)
set(CMAKE_SYSTEM_NAME WindowsStore)
set(CMAKE_SYSTEM_VERSION 10.0)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

include(CMakePrintHelpers)

project (runner LANGUAGES CXX)

# UWP tile and icon assets.
set(ASSET_FILES ${ASSET_FILES}
	Assets/LargeTile.scale-100.png
	Assets/LargeTile.scale-125.png
	Assets/LargeTile.scale-150.png
	Assets/LargeTile.scale-200.png
	Assets/LargeTile.scale-400.png
	Assets/LockScreenLogo.scale-200.png
	Assets/SmallTile.scale-100.png
	Assets/SmallTile.scale-125.png
	Assets/SmallTile.scale-150.png
	Assets/SmallTile.scale-200.png
	Assets/SmallTile.scale-400.png
	Assets/SplashScreen.scale-100.png
	Assets/SplashScreen.scale-125.png
	Assets/SplashScreen.scale-150.png
	Assets/SplashScreen.scale-200.png
	Assets/SplashScreen.scale-400.png
	Assets/Square44x44Logo.altform-unplated_targetsize-16.png
	Assets/Square44x44Logo.altform-unplated_targetsize-32.png
	Assets/Square44x44Logo.altform-unplated_targetsize-48.png
	Assets/Square44x44Logo.altform-unplated_targetsize-256.png
	Assets/Square44x44Logo.scale-100.png
	Assets/Square44x44Logo.scale-125.png
	Assets/Square44x44Logo.scale-150.png
	Assets/Square44x44Logo.scale-200.png
	Assets/Square44x44Logo.scale-400.png
	Assets/Square44x44Logo.targetsize-16.png
	Assets/Square44x44Logo.targetsize-24.png
	Assets/Square44x44Logo.targetsize-24_altform-unplated.png
	Assets/Square44x44Logo.targetsize-32.png
	Assets/Square44x44Logo.targetsize-48.png
	Assets/Square44x44Logo.targetsize-256.png
	Assets/Square150x150Logo.scale-100.png
	Assets/Square150x150Logo.scale-125.png
	Assets/Square150x150Logo.scale-150.png
	Assets/Square150x150Logo.scale-200.png
	Assets/Square150x150Logo.scale-400.png
	Assets/StoreLogo.png
	Assets/StoreLogo.scale-100.png
	Assets/StoreLogo.scale-125.png
	Assets/StoreLogo.scale-150.png
	Assets/StoreLogo.scale-200.png
	Assets/StoreLogo.scale-400.png
	Assets/Wide310x150Logo.scale-200.png
	Assets/WideTile.scale-100.png
	Assets/WideTile.scale-125.png
	Assets/WideTile.scale-150.png
	Assets/WideTile.scale-200.png
	Assets/WideTile.scale-400.png
)

# Configure package manifest file.
set(APP_MANIFEST_NAME Package.appxmanifest)
set(APP_MANIFEST_TARGET_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/${APP_MANIFEST_NAME})
set(SHORT_NAME ${BINARY_NAME})
set(PACKAGE_GUID "D4E599E0-6906-40A2-B342-82D06591FAC5")

configure_file(
	appxmanifest.in
	${APP_MANIFEST_TARGET_LOCATION}
	@ONLY)

set(CONTENT_FILES ${APP_MANIFEST_TARGET_LOCATION})

# Configure package content files.
set_property(SOURCE ${CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)

set(RESOURCE_FILES ${ASSET_FILES} ${CONTENT_FILES} Windows_TemporaryKey.pfx)
set_property(SOURCE ${ASSET_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)
set_property(SOURCE ${ASSET_FILES} PROPERTY VS_DEPLOYMENT_LOCATION "Assets")

set(STRING_FILES Resources.pri)
set_property(SOURCE ${STRING_FILES} PROPERTY VS_TOOL_OVERRIDE "PRIResource")

source_group("Resource Files" FILES ${RESOURCE_FILES} ${CONTENT_FILES} ${STRING_FILES})

# Configure Flutter assets using tool generated install manifest
foreach(ITEM ${INSTALL_MANIFEST_CONTENT})
	get_filename_component(ITEM_REL ${CMAKE_BINARY_DIR} DIRECTORY)
	file(RELATIVE_PATH RELPATH ${ITEM_REL} ${ITEM})

	get_filename_component(RELPATH ${RELPATH} DIRECTORY)
	get_filename_component(ITEMEXT ${ITEM} LAST_EXT)

	if("${ITEMEXT}" STREQUAL ".dll" OR "${ITEMEXT}" STREQUAL ".pdb")
		string(CONCAT RELPATH "")
	elseif ("${ITEMEXT}" STREQUAL ".so")
		file(RELATIVE_PATH RELPATH "${ITEM_REL}/winuwp" ${ITEM})
		string(REGEX REPLACE "/" "\\\\" RELPATH ${RELPATH})
		string(CONCAT RELPATH "Assets\\Data")
	elseif("${ITEMEXT}" STREQUAL ".dat")
		string(CONCAT RELPATH "Assets\\Data")
	else()
		string(REGEX REPLACE "/" "\\\\" RELPATH ${RELPATH})
		string(CONCAT RELPATH "Assets\\Data\\" ${RELPATH})
	endif()

	cmake_print_variables(${RELPATH})

	set_property(SOURCE ${ITEM} PROPERTY VS_DEPLOYMENT_CONTENT 1)
	set_property(SOURCE ${ITEM} PROPERTY VS_DEPLOYMENT_LOCATION ${RELPATH})
endforeach()

add_executable (${BINARY_NAME} WIN32
	main.cpp
	flutter_frameworkview.cpp
	"${FLUTTER_MANAGED_DIR}/generated_plugin_registrant.cc"
	${RESOURCE_FILES}
	${INSTALL_MANIFEST_CONTENT}
)
apply_standard_settings(${BINARY_NAME})
target_compile_definitions(${BINARY_NAME} PRIVATE "NOMINMAX")
target_link_libraries(${BINARY_NAME} PRIVATE WindowsApp flutter flutter_wrapper_app)
target_include_directories(${BINARY_NAME} PRIVATE "${CMAKE_SOURCE_DIR}")

add_dependencies(${BINARY_NAME} flutter_assemble)
