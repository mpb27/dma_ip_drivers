# Temporary CMakefile so that I can build in Clion
#
# Clion: https://gitlab.com/phip1611/cmake-kernel-module

cmake_minimum_required(VERSION 3.10)
project(xdma_mod C)

set(CMAKE_VERBOSE_MAKEFILE ON)

set(CMAKE_C_STANDARD 90)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(CMAKE_C_COMPILER /home/mark/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER /home/mark/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-g++)
set(ENV{CROSS_COMPILE} /home/mark/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-)
set(CROSS_COMPILE /home/mark/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-)

# From: https://gitlab.com/christophacham/cmake-kernel-module/-/blob/master/CMakeLists.txt
#       https://stackoverflow.com/questions/50877135/cmake-specify-linux-kernel-module-output-build-directory
#

# Find kernel headers.  Hard-code for now.
set(KERNELHEADERS_DIR           "/home/mark/kernel-out")
set(KERNELHEADERS_INCLUDE_DIRS
        "/home/mark/kernel-out/include"
        "/home/mark/kernel-out/source/include"
        "/home/mark/kernel-out/source/arch/arm64/include"
        CACHE PATH "Kernel headers include dirs")

# find MODULE_LICENSE("GPL"), MODULE_AUTHOR() etc.
# thanks to "merseyviking" from stack overflow
add_definitions(-D__KERNEL__ -DMODULE -DKBUILD_MODNAME="xdma_mod" -D__LIBXDMA_DEBUG__)

include_directories(${KERNELHEADERS_INCLUDE_DIRS})

# How the build command should look: https://unix.stackexchange.com/questions/511983/where-is-config-hz-defined


set(DRIVER_FILE xdma_mod.ko)
set(KBUILD_CMD $(MAKE) ARCH=arm64 CROSS_COMPILE=${CROSS_COMPILE} -C ${KERNELHEADERS_DIR} modules M=${CMAKE_CURRENT_BINARY_DIR} src=${CMAKE_CURRENT_SOURCE_DIR})

# Generate the Kbuild file through cmake.
FILE(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/Kbuild "xdma-objs := libxdma.o xdma_cdev.o cdev_ctrl.o cdev_events.o cdev_sgdma.o cdev_xvc.o cdev_bypass.o xdma_mod.o
 obj-m := xdma.o")

add_custom_command(OUTPUT ${DRIVER_FILE}
        COMMAND ${KBUILD_CMD}
        #        -E env "CROSS_COMPILE=/home/mark/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS xdma_mod.c VERBATIM)

add_custom_target(driver ALL DEPENDS ${DRIVER_FILE})

add_definitions(-include "../../../../../kernel-out/source/include/linux/kconfig.h")

add_library(dummy_target
	cdev_bypass.c
	cdev_ctrl.c	cdev_ctrl.h
	cdev_events.c
	cdev_sgdma.c	cdev_sgdma.h
	cdev_xvc.c	cdev_xvc.h
	libxdma.c	libxdma.h
	version.h
	xdma_cdev.c	xdma_cdev.h
	xdma_mod.c	xdma_mod.c
	)

