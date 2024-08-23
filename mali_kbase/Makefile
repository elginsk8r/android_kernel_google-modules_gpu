#
# (C) COPYRIGHT 2010-2018 ARM Limited. All rights reserved.
#
# This program is free software and is provided to you under the terms of the
# GNU General Public License version 2 as published by the Free Software
# Foundation, and any use by you of this program is subject to the terms
# of such GNU licence.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, you can access it online at
# http://www.gnu.org/licenses/gpl-2.0.html.
#
# SPDX-License-Identifier: GPL-2.0
#
#


KERNEL_SRC ?= /lib/modules/$(shell uname -r)/build
M ?= $(shell pwd)

#
# Meson integration configuration values
#
CONFIG_MALI_PLATFORM_NAME="meson"
CONFIG_MALI_PLATFORM_POWER_DOWN_ONLY=y
CONFIG_MALI_GATOR_SUPPORT=n

# Core kbase configuration options
CONFIG_MALI_EXPERT=y
CONFIG_MALI_MIDGARD_DVFS=y
CONFIG_MALI_DMA_FENCE=y

#
# Default configuration values
#
# Dependency resolution is done through statements as Kconfig
# is not supported for out-of-tree builds.
#
CONFIGS :=
ifeq ($(MALI_KCONFIG_EXT_PREFIX),)
    CONFIG_MALI_MIDGARD ?= m
    ifeq ($(CONFIG_MALI_MIDGARD),m)
        CONFIG_MALI_PLATFORM_NAME ?= "devicetree"
        CONFIG_MALI_GATOR_SUPPORT ?= y

        ifeq ($(CONFIG_MALI_MIDGARD_DVFS),y)
            # Prevent misuse when CONFIG_MALI_MIDGARD_DVFS=y
            CONFIG_MALI_DEVFREQ ?= n
        else
            CONFIG_MALI_DEVFREQ ?= y
        endif

        #
        # Expert/Debug/Test released configurations
        #
        ifeq ($(CONFIG_MALI_EXPERT), y)
            ifneq ($(CONFIG_MALI_NO_MALI), y)
                # Prevent misuse when CONFIG_MALI_NO_MALI=n
                CONFIG_MALI_ERROR_INJECT = n
            endif
            ifeq ($(CONFIG_MALI_DEBUG), y)
                CONFIG_MALI_MIDGARD_ENABLE_TRACE ?= y
                CONFIG_MALI_SYSTEM_TRACE ?= y
                ifeq ($(CONFIG_SYNC_FILE), y)
                    CONFIG_MALI_FENCE_DEBUG ?= y
                else
                    CONFIG_MALI_FENCE_DEBUG = n
                endif
            else
                # Prevent misuse when CONFIG_MALI_DEBUG=n
                CONFIG_MALI_MIDGARD_ENABLE_TRACE = n
                CONFIG_MALI_FENCE_DEBUG = n
            endif
        else
            # Prevent misuse when CONFIG_MALI_EXPERT=n
            CONFIG_MALI_CORESTACK = n
            CONFIG_MALI_PWRSOFT_765 = n
            CONFIG_MALI_2MB_ALLOC = n
            CONFIG_MALI_MEMORY_FULLY_BACKED = n
            CONFIG_MALI_JOB_DUMP = n
            CONFIG_MALI_NO_MALI = n
            CONFIG_MALI_ERROR_INJECT = n
            CONFIG_MALI_DEBUG = n
            CONFIG_MALI_MIDGARD_ENABLE_TRACE = n
            CONFIG_MALI_FENCE_DEBUG = n
        endif

        ifeq ($(CONFIG_MALI_DEBUG), y)
            CONFIG_MALI_FENCE_DEBUG ?= y
            CONFIG_MALI_KUTF ?= y
            ifeq ($(CONFIG_MALI_KUTF), y)
                CONFIG_MALI_KUTF_IRQ_TEST ?= y
            else
                # Prevent misuse when CONFIG_MALI_KUTF=n
                CONFIG_MALI_KUTF_IRQ_TEST = n
            endif
        else
            # Prevent misuse when CONFIG_MALI_DEBUG=n
            CONFIG_MALI_FENCE_DEBUG = n
            CONFIG_MALI_KUTF = y
            CONFIG_MALI_KUTF_IRQ_TEST = n
        endif
    else
        # Prevent misuse when CONFIG_MALI_MIDGARD=n
        CONFIG_MALI_KUTF = n
        CONFIG_MALI_KUTF_IRQ_TEST = n
    endif

    # All Mali CONFIG should be listed here
    CONFIGS += \
        CONFIG_MALI_MIDGARD \
        CONFIG_MALI_GATOR_SUPPORT \
        CONFIG_MALI_MIDGARD_DVFS \
        CONFIG_MALI_MIDGARD_ENABLE_TRACE \
        CONFIG_MALI_DEVFREQ \
        CONFIG_MALI_DMA_FENCE \
        CONFIG_MALI_PLATFORM_NAME \
        CONFIG_MALI_PLATFORM_POWER_DOWN_ONLY \
        CONFIG_MALI_EXPERT \
        CONFIG_MALI_CORESTACK \
        CONFIG_MALI_DEBUG \
        CONFIG_MALI_FENCE_DEBUG \
        CONFIG_MALI_NO_MALI \
        CONFIG_MALI_ERROR_INJECT \
        CONFIG_MALI_SYSTEM_TRACE \
        CONFIG_MALI_2MB_ALLOC \
        CONFIG_MALI_PWRSOFT_765 \
        CONFIG_MALI_JOB_DUMP \
        CONFIG_MALI_PRFCNT_SET_SECONDARY
endif

# MAKE_ARGS to pass the custom CONFIGs on out-of-tree build
#
# Generate the list of CONFIGs and values.
# $(value config) is the name of the CONFIG option.
# $(value $(value config)) is its value (y, m).
# When the CONFIG is not set to y or m, it defaults to n.
MAKE_ARGS := $(foreach config,$(CONFIGS), \
                    $(if $(filter y m,$(value $(value config))), \
                        $(value config)=$(value $(value config)), \
                        $(value config)=n))

ifeq ($(MALI_KCONFIG_EXT_PREFIX),)
    MAKE_ARGS += CONFIG_MALI_PLATFORM_NAME=$(CONFIG_MALI_PLATFORM_NAME)
endif

#
# EXTRA_CFLAGS to define the custom CONFIGs on out-of-tree build
#
# Generate the list of CONFIGs defines with values from CONFIGS.
# $(value config) is the name of the CONFIG option.
# When set to y or m, the CONFIG gets defined to 1.
EXTRA_CFLAGS := $(foreach config,$(CONFIGS), \
                    $(if $(filter y m,$(value $(value config))), \
                        -D$(value config)=1))

ifeq ($(MALI_KCONFIG_EXT_PREFIX),)
    EXTRA_CFLAGS += -DCONFIG_MALI_PLATFORM_NAME='\"$(CONFIG_MALI_PLATFORM_NAME)\"'
    EXTRA_CFLAGS += -DCONFIG_MALI_NO_MALI_DEFAULT_GPU='\"$(CONFIG_MALI_NO_MALI_DEFAULT_GPU)\"'
endif

#
# KBUILD_EXTRA_SYMBOLS to prevent warnings about unknown functions
#
BASE_SYMBOLS = 

EXTRA_SYMBOLS += \
    $(BASE_SYMBOLS)

CFLAGS_MODULE += -Wall -Werror

# The following were added to align with W=1 in scripts/Makefile.extrawarn
# from the Linux source tree (v5.18.14)
CFLAGS_MODULE += -Wextra -Wunused -Wno-unused-parameter
CFLAGS_MODULE += -Wmissing-declarations
CFLAGS_MODULE += -Wmissing-format-attribute
CFLAGS_MODULE += -Wmissing-prototypes
CFLAGS_MODULE += -Wold-style-definition
# The -Wmissing-include-dirs cannot be enabled as the path to some of the
# included directories change depending on whether it is an in-tree or
# out-of-tree build.
CFLAGS_MODULE += $(call cc-option, -Wunused-but-set-variable)
CFLAGS_MODULE += $(call cc-option, -Wunused-const-variable)
CFLAGS_MODULE += $(call cc-option, -Wpacked-not-aligned)
CFLAGS_MODULE += $(call cc-option, -Wstringop-truncation)
# The following turn off the warnings enabled by -Wextra
CFLAGS_MODULE += -Wno-sign-compare
CFLAGS_MODULE += -Wno-shift-negative-value
# This flag is needed to avoid build errors on older kernels
CFLAGS_MODULE += $(call cc-option, -Wno-cast-function-type)

KBUILD_CPPFLAGS += -DKBUILD_EXTRA_WARN1

# The following were added to align with W=2 in scripts/Makefile.extrawarn
# from the Linux source tree (v5.18.14)
CFLAGS_MODULE += -Wdisabled-optimization
# The -Wshadow flag cannot be enabled unless upstream kernels are
# patched to fix redefinitions of certain built-in functions and
# global variables.
CFLAGS_MODULE += $(call cc-option, -Wlogical-op)
CFLAGS_MODULE += -Wmissing-field-initializers
# -Wtype-limits must be disabled due to build failures on kernel 5.x
CFLAGS_MODULE += -Wno-type-limits
CFLAGS_MODULE += $(call cc-option, -Wmaybe-uninitialized)
CFLAGS_MODULE += $(call cc-option, -Wunused-macros)
# The following ensures the stack frame does not get larger than a page
CFLAGS_MODULE += -Wframe-larger-than=4096

KBUILD_CPPFLAGS += -DKBUILD_EXTRA_WARN2

# This warning is disabled to avoid build failures in some kernel versions
CFLAGS_MODULE += -Wno-ignored-qualifiers

ifeq ($(CONFIG_GCOV_KERNEL),y)
    CFLAGS_MODULE += $(call cc-option, -ftest-coverage)
    CFLAGS_MODULE += $(call cc-option, -fprofile-arcs)
    EXTRA_CFLAGS += -DGCOV_PROFILE=1
endif

ifeq ($(CONFIG_MALI_KCOV),y)
    CFLAGS_MODULE += $(call cc-option, -fsanitize-coverage=trace-cmp)
    EXTRA_CFLAGS += -DKCOV=1
    EXTRA_CFLAGS += -DKCOV_ENABLE_COMPARISONS=1
endif

# we get the symbols from modules using KBUILD_EXTRA_SYMBOLS to prevent warnings about unknown functions
modules:
	$(MAKE) -C $(KERNEL_SRC) M=$(M) W=1 $(MAKE_ARGS) EXTRA_CFLAGS="$(EXTRA_CFLAGS)" KBUILD_EXTRA_SYMBOLS="$(EXTRA_SYMBOLS)" modules

modules_install:
	$(MAKE) -C $(KERNEL_SRC) M=$(M) modules_install

clean:
	$(MAKE) -C $(KERNEL_SRC) M=$(M) clean
