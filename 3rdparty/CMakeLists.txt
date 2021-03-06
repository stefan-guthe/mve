cmake_minimum_required(VERSION 3.0)

project(3rdparty)
include(ExternalProject)

set(LIBJPEG_FILENAME "jpegsr9a.zip")
set(LIBPNG_FILENAME "lpng1610.zip")
set(ZLIB_FILENAME "zlib128.zip")
set(LIBTIFF_FILENAME "tiff-4.0.3.zip")
set(GLEW_FILENAME "glew-1.10.0.zip")
set(QT5_FILENAME "qt-everywhere-opensource-src-5.2.1.zip")
set(GTEST_FILENAME "gtest-1.7.0.zip")

set(LIBJPEG_URL "http://www.ijg.org/files/${LIBJPEG_FILENAME}")
set(LIBPNG_URL "http://download.sourceforge.net/libpng/${LIBPNG_FILENAME}")
set(ZLIB_URL "http://download.sourceforge.net/libpng/${ZLIB_FILENAME}")
set(LIBTIFF_URL "ftp://ftp.remotesensing.org/pub/libtiff/${LIBTIFF_FILENAME}")
set(GLEW_URL "http://downloads.sourceforge.net/project/glew/glew/1.10.0/${GLEW_FILENAME}")
set(QT5_URL "http://download.qt-project.org/official_releases/qt/5.2/5.2.1/single/${QT5_FILENAME}")
set(GTEST_URL "https://googletest.googlecode.com/files/${GTEST_FILENAME}")

set(PATCHES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/patches")

file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")
file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include")
file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/lib")

message("Forcing build type to \"Release\"")
message("Reminder: Select Release configuration in Visual Studio before building")
set(CMAKE_BUILD_TYPE Release CACHE STRING "")

find_file(WIN32_MAK_LOCATION win32.mak
	DOC "Location of win32.mak"
	PATHS
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.0A\\Include"
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.1\\Include"
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.1A\\Include"
		ENV Path)
ExternalProject_Add(libjpeg
	PREFIX build/libjpeg
	URL ${LIBJPEG_URL}
	URL_HASH SHA256=144aeeb75240241fbfae3f1ddc86829525174ba04405ef9159884f241e112752
	CONFIGURE_COMMAND nmake /f makefile.vc setup-v10
	BUILD_COMMAND nmake /f makefile.vc
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jconfig.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jerror.h   ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jmorecfg.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jpegint.h  ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jpeglib.h  ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/libjpeg.lib  ${CMAKE_CURRENT_SOURCE_DIR}/lib
	BUILD_IN_SOURCE 1)
ExternalProject_Add_Step(libjpeg copy_win32_mak
	COMMAND ${CMAKE_COMMAND} -E copy ${WIN32_MAK_LOCATION} <SOURCE_DIR>
	DEPENDEES download
	DEPENDERS configure)

ExternalProject_Add(zlib
	PREFIX build/zlib
	URL ${ZLIB_URL}
	URL_HASH SHA256=879d73d8cd4d155f31c1f04838ecd567d34bebda780156f0e82a20721b3973d5
	CMAKE_ARGS
		-DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
		-DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_CURRENT_SOURCE_DIR}
	BUILD_IN_SOURCE 1)

ExternalProject_Add(libpng
	DEPENDS zlib
	PREFIX build/libpng
	URL ${LIBPNG_URL}
	URL_HASH SHA256=d6d884e22c2af9ccef21349acd88064e38f551fd79a4ef86255781d2e65ea202
	CMAKE_ARGS
		-DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
		-DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_CURRENT_SOURCE_DIR}
		-DZLIB_ROOT:STRING=${CMAKE_CURRENT_SOURCE_DIR}
	BUILD_IN_SOURCE 1)

ExternalProject_Add(libtiff
	PREFIX build/libtiff
	URL ${LIBTIFF_URL}
	URL_HASH SHA256=c77f0d23b4c6fef96ed18ddc811990b35687848ccd2bacd7edc327b5795b6481
	CONFIGURE_COMMAND ""
	BUILD_COMMAND nmake /f Makefile.vc lib
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffconf.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiff.h     ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffio.h   ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffio.hxx ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffvers.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff.lib   ${CMAKE_CURRENT_SOURCE_DIR}/lib
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff_i.lib ${CMAKE_CURRENT_SOURCE_DIR}/lib
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff.dll ${CMAKE_CURRENT_SOURCE_DIR}/bin
	BUILD_IN_SOURCE 1)

if(CMAKE_BUILD_TYPE STREQUAL "Debug"
		OR CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo"
		OR NOT CMAKE_BUILD_TYPE)
	set(VS_CONFIG "Debug")
elseif(CMAKE_BUILD_TYPE STREQUAL "Release"
		OR CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
	set(VS_CONFIG "Release")
endif()

if(CMAKE_CL_64)
	set(VS_PLATFORM "x64")
else()
	set(VS_PLATFORM "Win32")
endif()
ExternalProject_Add(glew
	PREFIX build/glew
	URL ${GLEW_URL}
	URL_HASH SHA256=43c6229d787673ac1d35ebaad52dfdcc78c8b55d13ee78d8e4d7e4a6cb72b050
	CONFIGURE_COMMAND ""
	PATCH_COMMAND ${CMAKE_COMMAND} -E copy ${PATCHES_DIR}/glew.rc build
	BUILD_COMMAND cd build/vc10
		COMMAND devenv /upgrade glew.sln
		COMMAND msbuild glew.sln /m /p:Configuration=${VS_CONFIG} /p:Platform=${VS_PLATFORM}
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/build/glew/src/glew/include ${CMAKE_CURRENT_SOURCE_DIR}/include
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/glew/src/glew/lib/${VS_CONFIG}/${VS_PLATFORM}/glew32s.lib ${CMAKE_CURRENT_SOURCE_DIR}/lib
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/build/glew/src/glew/bin/${VS_CONFIG}/${VS_PLATFORM} ${CMAKE_CURRENT_SOURCE_DIR}/bin
	BUILD_IN_SOURCE 1)

#ExternalProject_Add(qt5
#	PREFIX qt5
#	URL ${QT5_URL}
#	URL_HASH SHA256=a75a07b886b54023aeffb2daae552a65062c9c1f14d410acad0d47341c45539d
#	CONFIGURE_COMMAND
#		configure -debug-and-release -opensource -shared -no-audio-backend -mp
#				   -opengl desktop -skip qtactiveqt -skip qtconnectivity
#				   -skip qtdeclarative -skip qtdoc -skip qtgraphicaleffects
#				   -skip qtlocation -skip qtmultimedia -skip qtquick1
#				   -skip qtquickcontrols -skip qtscript -skip qtsensors
#				   -skip qtserialport -skip qttools -skip qttranslations
#				   -skip qtwebkit -skip qtwebkit-examples
#	BUILD_COMMAND nmake
#	BUILD_IN_SOURCE 1
#	INSTALL_COMMAND "")

ExternalProject_Add(gtest
	PREFIX build/gtest
	URL ${GTEST_URL}
	URL_HASH SHA256=247ca18dd83f53deb1328be17e4b1be31514cedfc1e3424f672bf11fd7e0d60d
	CMAKE_ARGS
		-DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/build/gtest/src/gtest/include ${CMAKE_CURRENT_SOURCE_DIR}/include
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/build/gtest/src/gtest/${CMAKE_BUILD_TYPE} ${CMAKE_CURRENT_SOURCE_DIR}/lib
	BUILD_IN_SOURCE 1)

