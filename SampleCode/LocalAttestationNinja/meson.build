project('LocalAttestationNinja', 'c', 'cpp',
  version : '1.0.0',
  default_options : ['cpp_compiler=g++'])


root_dir = meson.current_source_dir()


sgx_sdk = get_option('SGX_SDK')
if sgx_sdk == ''
    warning('import SGX SDK environment variable')
    sgx_sdk = '/opt/intel/sgxsdk/'
endif

sgx_sdk_path = sgx_sdk


#-------------------------------------------------------------------
# This is the output folder.
#-------------------------------------------------------------------
bin_dir = 'bin'
topdir = root_dir
outdir = bin_dir

cp = find_program('cp')
cc = meson.get_compiler('c')
#cxx = meson.get_compiler('cpp')

#if cxx.get_id() != 'g++'
#  error('This project requires g++')
#endif

common_flags = []
cc_version = cc.version()
if cc_version.version_compare('>=8')
    common_flags += ['-fcf-protection']
endif

# ------------------------------------------------------------------
#  Define common variables
# ------------------------------------------------------------------
sgx_mode = get_option('SGX_MODE')
sgx_arch = get_option('SGX_ARCH')
sgx_debug = get_option('SGX_DEBUG')
sgx_prerelease = get_option('SGX_PRERELEASE')

#-------------------------------------------------------------------
# Define common compile flags used for GCC and G++ 
#-------------------------------------------------------------------
common_flags += ['-ffunction-sections', '-fdata-sections']

common_flags += [
    '-Wall', '-Wextra', '-Wchar-subscripts', '-Wno-coverage-mismatch', '-Winit-self',
    '-Wpointer-arith', '-Wreturn-type', '-Waddress', '-Wsequence-point', '-Wformat-security',
    '-Wmissing-include-dirs', '-Wfloat-equal', '-Wundef', '-Wshadow',
    '-Wcast-align', '-Wconversion', '-Wredundant-decls'
]

if sgx_debug
    if sgx_prerelease
        error('Cannot set SGX_DEBUG and SGX_PRERELEASE at the same time!!')
    endif
endif

if sgx_debug
    common_flags += ['-ggdb', '-DDEBUG', '-DDEBUG_LEVEL=TRACE_DEBUG']
else
    common_flags += ['-O2', '-UDEBUG']
endif

c_flags = common_flags
cpp_flags = common_flags

# additional warnings flags for C
c_flags += ['-Wjump-misses-init', '-Wstrict-prototypes', '-Wunsuffixed-float-constants']

# additional warnings flags for C++
cpp_flags += ['-Wnon-virtual-dtor', '-std=c++11']

# ----------------------------------------------------------------
#  Define common link options
# ----------------------------------------------------------------
common_link_args = ['-Wl,-z,relro,-z,now,-z,noexecstack']

# Compiler and linker options for an Enclave
#
# We are using '--export-dynamic' so that `g_global_data_sim' etc.
# will be exported to dynamic symbol table.
#
# When `pie' is enabled, the linker (both BFD and Gold) under Ubuntu 14.04
# will hide all symbols from dynamic symbol table even if they are marked
# as `global' in the LD version script.
enclave_c_args = ['-ffreestanding', '-nostdinc', '-fvisibility=hidden', '-fpie']
if cc_version.version_compare('>=8')
    enclave_c_args += ['-fcf-protection']
endif
enclave_cpp_args = enclave_c_args + ['-nostdinc++']
enclave_link_args = common_link_args



# ifeq ($(shell getconf LONG_BIT), 32)
#         SGX_ARCH := x86
# else ifeq ($(findstring -m32, $(CXXFLAGS)), -m32)
#         SGX_ARCH := x86
# endif
if host_machine.cpu_family() == 'x86'
    sgx_arch = 'x86'
elif host_machine.cpu_family() == 'x86_64' and get_option('cpp_args').contains('-m32')
    sgx_arch = 'x86'
endif

if sgx_arch == 'x86'
    sgx_common_flags = ['-m32']
    sgx_library_path = join_paths(sgx_sdk, 'lib')
    sgx_enclave_signer = join_paths(sgx_sdk, 'bin', 'x86', 'sgx_sign')
    sgx_edger8r = join_paths(sgx_sdk, 'bin', 'x86', 'sgx_edger8r')
else
    sgx_common_flags = ['-m64']
    sgx_library_path = join_paths(sgx_sdk, 'lib64')
    sgx_enclave_signer = join_paths(sgx_sdk, 'bin', 'x64', 'sgx_sign')
    sgx_edger8r = join_paths(sgx_sdk, 'bin', 'x64', 'sgx_edger8r')
endif

sgx_common_flags += common_flags






build_mode = ''

if sgx_mode == 'HW'
    if sgx_debug == true
        build_mode = 'HW_DEBUG'
    elif sgx_prerelease == true
        build_mode = 'HW_PRERELEASE'
    else
        build_mode = 'HW_RELEASE'
    endif
endif

if sgx_mode == 'SIM'
    if sgx_debug == true
        build_mode = 'SIM_DEBUG'
    elif sgx_prerelease == true
        build_mode = 'SIM_PRERELEASE'
    else
        build_mode = 'SIM_RELEASE'
    endif
endif

fs = import('fs')
build_root = join_paths(meson.source_root(), 'build')


subdir('AppInitiator')
subdir('AppResponder')
subdir('EnclaveInitiator')
subdir('EnclaveResponder')
subdir('App')

run_target('print_build_mode',
    command: ['echo', 'The project has been built in ' + build_mode + ' mode.']
)
