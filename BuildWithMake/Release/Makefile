# Hey emacs, this is a -*- makefile -*-

# Copyright (c) Stanford University, The Regents of the University of
#               California, and others.
#
# All Rights Reserved.
#
# See Copyright-SimVascular.txt for additional details.
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject
# to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
# IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
# TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
# PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER
# OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
# LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
# NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

TOP = ..

include $(TOP)/include.mk

#
#  The following are helper scripts to create PC
#  distributions.  Note that some of the paths are hardcoded.
#

SV_CLUSTER = $(CLUSTER)

ifneq ($(CLUSTER),x64_linux)
  include $(TOP)/MakeHelpers/wix-3.14.x64_cygwin.mk
endif

SV_TIMESTAMP := $(shell date +"%Y-%m-%d")
DIST_DIR_TOP = package/$(SV_VERSION)
DIST_DIR := $(DIST_DIR_TOP)/$(SV_TIMESTAMP)

REAL_DIST_DIR_TOP = package/$(SV_VERSION)
REAL_DIST_DIR := $(DIST_DIR_TOP)/$(SV_TIMESTAMP)

ifeq ($(CLUSTER), x64_cygwin)
  DIST_DIR_TOP := /cygdrive/p/package/$(SV_VERSION)
  DIST_DIR := /cygdrive/p/package/$(SV_VERSION)/$(SV_TIMESTAMP)
else
  DIST_DIR_TOP := $(REAL_DIST_DIR_TOP)
  DIST_DIR := $(REAL_DIST_DIR)
endif

ifneq ($(CLUSTER),x64_linux)
  all: use_subst windows-distribution msi msi-bundle
else
  all: linux-distribution taritup
endif

use_subst:
ifeq ($(CLUSTER),x64_cygwin)
	-subst P: /D
	-subst P: $(shell cygpath -w `pwd`)
endif

veryclean: use_subst msiclean msi-bundle-clean
	-rm -f tar_files/*.tar
	-rm -f tar_files/*.tar.gz
	-rm -Rf $(DIST_DIR_TOP)
	-rm -Rf package
	-rm -Rf tmp/*

#
# create script for sv
#
sv-startup-configure:
	-rm tmp/startup_configure.tcl
	-echo "set SV_BUILD_ID     $(SV_TIMESTAMP)" > tmp/startup_configure.tcl
	-echo "set SV_FULL_VER_NO  $(SV_FULL_VER_NO)" >> tmp/startup_configure.tcl
	-echo "set SV_MAJOR_VER_NO $(SV_MAJOR_VER_NO)" >> tmp/startup_configure.tcl
	-echo "set SV_TIMESTAMP    $(SV_TIMESTAMP)" >> tmp/startup_configure.tcl
	-echo "set SV_PLATFORM     $(SV_PLATFORM)" >> tmp/startup_configure.tcl
	-echo "set SV_VERSION      $(SV_VERSION)" >> tmp/startup_configure.tcl
ifeq ($(SV_USE_PYTHON),1)
	-echo "set SV_USE_PYTHON   ON" >> tmp/startup_configure.tcl
else
	-echo "set SV_USE_PYTHON   OFF" >> tmp/startup_configure.tcl
endif

#
#  Create msi installer for SimVascular
#

msiclean:
	-rm -f *~
	-rm -f $(SV_VERSION).msi
	-rm -f tmp/simvascular.wixpdb
	-rm -f simvascular.wixpdb
	-rm -f tmp/sv.wixobj
	-rm -f tmp/sv.wxs
	-rm -f tmp/uuidgen.exe*
	-rm -f tmp/uuidgen.obj*

msi-simvascular:	msiclean
ifeq ($(CXX_COMPILER_VERSION),mingw-gcc)
	x86_64-w64-mingw32-gcc -o tmp/uuidgen.exe windows_msi_helpers/uuidgen.c -lole32
else
	CL /Fetmp/uuidgen.exe /Fotmp/uuidgen.obj windows_msi_helpers/uuidgen.c ole32.lib
endif
	-echo "Create WXS file..."
	-tclsh windows_msi_helpers/makewxs.tcl $(SV_VERSION) $(SV_PLATFORM) $(DIST_DIR_TOP) $(SV_MAJOR_VER_NO) Releases$(SV_POSTFIX) sv.bat $(SV_VERSION) $(SV_FULL_VER_NO) $(TCL_LIBRARY) $(TK_LIBRARY)
	-echo "Run candle..."
	-$(WIXUI_PATH)/candle.exe -ext WixUIExtension -ext WixUtilExtension -out tmp/sv.wixobj tmp/sv.wxs
	-echo "Run light..."
	-$(WIXUI_PATH)/light.exe -ext WixUIExtension -ext WixUtilExtension -out $(SV_VERSION).msi tmp/sv.wixobj

msi-bundle-clean:
	-rm -f tmp/bundle.wixobj
	-rm -f simvascular-bundle.wixpdb
	-rm -f simvascular-bundle.exe
	-rm -f tmp/wix-simvascular-bundle.wxs

msi-bundle: msi-bundle-clean mksedscript
	-echo "Create WXS file..."
	-sed -f tmp/sedscript windows_installer_packages/wix-simvascular-bundle.wxs > tmp/wix-simvascular-bundle.wxs
	-echo "Run candle..."
	-$(WIXUI_PATH)/candle.exe -ext WixBalExtension -ext WixUIExtension -ext WixUtilExtension -out tmp/bundle.wixobj tmp/wix-simvascular-bundle.wxs
	-echo "Run light..."
	-$(WIXUI_PATH)/light.exe tmp/bundle.wixobj -ext WixBalExtension -ext WixUIExtension -ext WixUtilExtension -out simvascular-bundle.exe

msi: msiclean msi-simvascular

mksedscript:
	-mkdir -p tmp
	-echo "puts \"s/REPLACE_TIMESTAMP/[file tail $(DIST_DIR)]/g\"" | tclsh | cat > tmp/sedscript
	-echo "puts \"s/REPLACE_SV_TIMESTAMP/[file tail $(DIST_DIR)]/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_OS/$(SV_OS)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_VERSION/$(SV_VERSION)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_PLATFORM/$(SV_PLATFORM)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_POSTFIX/$(SV_POSTFIX)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_MAJOR_VER_NO/$(SV_MAJOR_VER_NO)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_FULL_VER_NO/$(SV_FULL_VER_NO)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_SV_CLUSTER/$(CLUSTER)/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_TCL_VERSION_NO/$(notdir $(TCL_LIBRARY))/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_TK_VERSION_NO/$(notdir $(TK_LIBRARY))/g\"" | tclsh | cat >> tmp/sedscript
	-echo "puts \"s/REPLACE_TCLSH/$(notdir $(TCLSH))/g\"" | tclsh | cat >> tmp/sedscript

common-pre-distribution: tmp/sedscript
	-date > tmp/release-date
	-rm -f tar_files/*.tar
	-rm -f tar_files/*.tar.gz
	-rm -Rf $(DIST_DIR_TOP)
	-mkdir -p $(DIST_DIR)
	-date > $(DIST_DIR)/release-date
	-chmod -R a+rwx $(DIST_DIR_TOP)/../../package
	-echo "puts \"[file tail $(DIST_DIR)]\"" | tclsh > tmp/timestamp
	-cp -f License.rtf $(DIST_DIR)

copy-sv-executables:
	-cp -f $(TOP)/Bin/simvascular.exe $(DIST_DIR)/simvascular-bin.exe
	-cp -f $(TOP)/Lib/$(LIB_BUILD_DIR)/*simvascular_globals*.$(SOEXT)* $(DIST_DIR)
ifeq ($(SV_USE_PYTHON),1)
	-cp -f $(TOP)/Lib/$(LIB_BUILD_DIR)/*simvascular_python_interp*.$(SOEXT)* $(DIST_DIR)
endif
	-cp -f $(TOP)/Lib/$(LIB_BUILD_DIR)/*simvascular_*.$(SOEXT)* $(DIST_DIR)
	-cp -fR $(TOP)/Bin/plugins $(DIST_DIR)

copy-sv-tcl-code:
	-mkdir -p $(DIST_DIR)/Tcl
	-cp -Rf $(TOP)/../Tcl $(DIST_DIR)/UnTcl
         # relace the full version number in the tcl launch script
	-sed -f tmp/sedscript $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_startup.tcl > $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_startup.tcl.new
	-mv $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_startup.tcl.new $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_startup.tcl
	-sed -f tmp/sedscript $(DIST_DIR)/UnTcl/SimVascular_2.0/GUI/splash.tcl > $(DIST_DIR)/UnTcl/SimVascular_2.0/GUI/splash.tcl.new
	-mv $(DIST_DIR)/UnTcl/SimVascular_2.0/GUI/splash.tcl.new $(DIST_DIR)/UnTcl/SimVascular_2.0/GUI/splash.tcl
	-tclsh windows_msi_helpers/prepare_simvascular_tcl_release.tcl $(DIST_DIR)
	-cp -f $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_startup.tcl $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp -f $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_vtk_init.tcl $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp -f $(DIST_DIR)/UnTcl/SimVascular_2.0/launch_xf.tcl $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp -R $(DIST_DIR)/UnTcl/External $(DIST_DIR)/Tcl
	-cp -R $(DIST_DIR)/UnTcl/Help $(DIST_DIR)/Tcl
	-cp $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular_logo.jpg $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp $(DIST_DIR)/UnTcl/SimVascular_2.0/splash.gif $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp $(DIST_DIR)/UnTcl/SimVascular_2.0/simvascular.rc $(DIST_DIR)
	-rm -Rf $(DIST_DIR)/UnTcl
	-tclsh tcl_helper_scripts/create_tclIndex.tcl $(DIST_DIR)/Tcl/SimVascular_2.0
	-cp -f tmp/startup_configure.tcl $(DIST_DIR)/Tcl

copy-tcl-libs:
	-cp -fR $(TCL_BASE)/lib $(DIST_DIR)
	-cp -fR $(TCL_BASE)/bin $(DIST_DIR)
	-cp -fR $(TCL_BASE)/bin/*.$(SOEXT)* $(DIST_DIR)
	-cp -fR $(TCL_BASE)/lib/*.$(SOEXT)* $(DIST_DIR)
	-cp -fR $(TKCXIMAGE_DLL) $(DIST_DIR)

copy-external-libs:
ifeq ($(SV_USE_VTK),1)
	-cp -fd $(VTK_LIBDIRS)/*.$(SOEXT)* $(DIST_DIR)
	-cp -fd $(VTK_BINDIRS)/*.$(SOEXT)* $(DIST_DIR)
endif
#ifeq ($(SV_USE_ZLIB),1)
#	-cp -f $(ZLIB_LIBDIR)/zlib.$(SOEXT)* $(DIST_DIR)
#endif
ifeq ($(SV_USE_ITK),1)
	-cp -f $(ITK_BIN_LIBS)/*.$(SOEXT)* $(DIST_DIR)
endif
ifeq ($(SV_USE_GDCM),1)
	-cp -f $(SV_GDCM_DLLS)/*gdcm*.$(SOEXT)* $(DIST_DIR)
ifeq ($(CLUSTER),x64_cygwin)
	-cp -f $(SV_GDCM_DLLS)/gdcm*.exe* $(DIST_DIR)
else
	-cp -fR $(SV_GDCM_DLLS)/gdcm* $(DIST_DIR)
endif
	-cp -f $(SV_GDCM_DLLS)/socketxx.$(SOEXT)* $(DIST_DIR)
endif
ifeq ($(SV_USE_PYTHON),1)
#	-cp -fR $(PYTHON_HOME)/Lib $(DIST_DIR)/python-packages
#	-cp -fR $(VTK_PYTHON_PACKAGES) $(DIST_DIR)/python-packages
	-cp -fR $(PYTHON_HOME) $(DIST_DIR)/Python27
	-cp -fR $(VTK_PYTHON_PACKAGES) $(DIST_DIR)/Python27/Lib/vtk-packages
	-cp -f $(SV_PYTHON_SO_PATH)/*.$(SOEXT)* $(DIST_DIR)
	-cp -Rf $(TOP)/../Python $(DIST_DIR)/Python
endif
ifeq ($(SV_USE_QT),1)
ifeq ($(CLUSTER), x64_cygwin)
	-cp -fd $(QT_BIN_LIBS)/*.$(SOEXT)* $(DIST_DIR)
else
	-cp -fd $(QT_LIBDIRS)/*.$(SOEXT)* $(DIST_DIR)
endif
	-cp -fR $(QT_PLUGIN_PATH) $(DIST_DIR)/qt-plugins
endif
ifeq ($(SV_USE_OPENCASCADE),1)
	-cp -fd $(SV_OPENCASCADE_SO_PATH)/*.$(SOEXT)* $(DIST_DIR)
endif
ifeq ($(SV_USE_MITK),1)
	-mkdir -p $(DIST_DIR)/mitk
ifeq ($(CLUSTER), x64_cygwin)
	-cp -fRd $(MITK_BINDIRS) $(DIST_DIR)/mitk
#	-mkdir -p $(DIST_DIR)/mitk/bin/plugins
#	-cp -fR $(SV_MITK_PLUGIN_PATH) $(DIST_DIR)/mitk/bin/plugins
#	-cp -f $(SV_MITK_SO_PATH)/liborg*.$(SOEXT)* $(DIST_DIR)/mitk/bin/plugins/RelWithDebInfo
else
	-mkdir -p $(DIST_DIR)/mitk/bin
	-mkdir -p $(DIST_DIR)/mitk/lib/plugins
	-cp -fR $(MITK_BINDIRS)/dcm* $(DIST_DIR)
	-cp -f $(MITK_LIBDIRS)/*.$(SOEXT)* $(DIST_DIR)/mitk/lib
	-cp -fR $(MITK_LIBDIRS)/MitkCore $(DIST_DIR)/mitk/lib
	-cp -fR $(MITK_LIBDIRS)/MitkPython $(DIST_DIR)/mitk/lib
	-cp -fR $(MITK_LIBDIRS)/Python $(DIST_DIR)/mitk/lib
	-cp -f $(SV_MITK_PLUGIN_PATH)/* $(DIST_DIR)/mitk/lib/plugins
	-cp -f $(SV_MITK_SO_PATH)/liborg*.$(SOEXT)* $(DIST_DIR)/mitk/lib/plugins
endif
endif

copy-external-libs-windows-only:
	-cp -f dicom2/dicom2-win32.exe $(DIST_DIR)/dicom2.exe
ifeq ($(USE_GLIB),1)
	-cp $(GLIB_DLLS) $(DIST_DIR)
endif
ifeq ($(FORTRAN_COMPILER_VERSION),mingw-gfortran)
	-cp -f /usr/x86_64-w64-mingw32/sys-root/mingw/bin/*.dll $(DIST_DIR)
endif

copy-external-libs-linux-only:
	-cp -f dicom2/dicom2-linux.exe $(DIST_DIR)/dicom2

generate-wrapper-scripts:
	-sed -f tmp/sedscript $(SV_OS)_wrapper_scripts/$(SV_OS)-dist-core-script > $(DIST_DIR)/$(SV_VERSION)
	-sed -f tmp/sedscript $(SV_OS)_wrapper_scripts/$(SV_OS)-dist-core-script > $(DIST_DIR)/xfsv$(SV_POSTFIX)
	-sed -f tmp/sedscript $(SV_OS)_wrapper_scripts/$(SV_OS)-dist-simvascular-script >> $(DIST_DIR)/$(SV_VERSION)
	-sed -f tmp/sedscript $(SV_OS)_wrapper_scripts/$(SV_OS)-dist-xfsv-script >> $(DIST_DIR)/xfsv$(SV_POSTFIX)
	-cp -f $(SV_OS)_wrapper_scripts/$(SV_OS)-simvascular-cmdline-script $(DIST_DIR)/$(SV_VERSION)-cmdline
	-cat $(DIST_DIR)/$(SV_VERSION) >> $(DIST_DIR)/$(SV_VERSION)-cmdline
ifeq ($(CLUSTER),x64_cygwin)
	-cp -f $(SV_OS)_wrapper_scripts/sv.bat $(DIST_DIR)
	-chmod a+rx $(DIST_DIR)/sv.bat
endif

common-post-distribution:
	-sed -f tmp/sedscript platform_independent_scripts/post-install.sh > $(DIST_DIR)/post-install.sh
	-sed -f tmp/sedscript platform_independent_scripts/generic-launch-script > $(DIST_DIR)/generic_launch_script
	-chmod -f a+rx $(DIST_DIR)/*.$(SOEXT)
	-chmod -f a+rx $(DIST_DIR)/*.sh
	-chmod -f a+rx $(DIST_DIR)/generic_launch_script
	-chmod -f a+rx $(DIST_DIR)/$(SV_VERSION)
	-chmod -f a+rx $(DIST_DIR)/$(SV_VERSION)-cmdline
	-chmod -f a+rx $(DIST_DIR)/*.$(SOEXT)*
	-chmod -f -R a+rx $(DIST_DIR)
	-chmod -f -R u+w $(DIST_DIR)

linux-distribution: mksedscript common-pre-distribution copy-sv-executables sv-startup-configure copy-tcl-libs copy-sv-tcl-code copy-external-libs copy-external-libs-linux-only generate-wrapper-scripts common-post-distribution

windows-distribution: mksedscript common-pre-distribution copy-sv-executables sv-startup-configure copy-tcl-libs copy-sv-tcl-code copy-external-libs copy-external-libs-windows-only generate-wrapper-scripts common-post-distribution

taritup:
	-mkdir tar_files
	-tar --preserve-permissions -cvf tar_files/simvascular-$(SV_OS)-$(SV_PLATFORM).`awk '{print $$2"-"$$3"-"$$6}' tmp/release-date`.tar package
	-gzip tar_files/simvascular-$(SV_OS)-$(SV_PLATFORM).`awk '{print $$2"-"$$3"-"$$6}' tmp/release-date`.tar
