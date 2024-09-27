


if get_option('SGX_MODE') != 'HW'
  trts_library_name = 'sgx_trts_sim'
  service_library_name = 'sgx_tservice_sim'
else
  trts_library_name = 'sgx_trts'
  service_library_name = 'sgx_tservice'
endif

crypto_library_name = 'sgx_tcrypto'

enclave_name = 'libenclave_initiator.so'
signed_enclave_name = 'libenclave_initiator.signed.so'
enclave_test_key = 'EnclaveInitiator_private_test.pem'

enclave_security_link_flags = ['-Wl,-z,relro,-z,now,-z,noexecstack']

# To generate a proper enclave, it is recommended to follow below guideline to link the trusted libraries:
#    1. Link sgx_trts with the `--whole-archive' and `--no-whole-archive' options,
#       so that the whole content of trts is included in the enclave.
#    2. For other libraries, you just need to pull the required symbols.
#       Use `--start-group' and `--end-group' to link these libraries.
# Do NOT move the libraries linked with `--start-group' and `--end-group' within `--whole-archive' and `--no-whole-archive' options.
# Otherwise, you may get some undesirable errors.
enclave_link_flags = enclave_security_link_flags + [
    '-Wl,--no-undefined',
    '-nostdlib',
    '-nodefaultlibs',
    '-nostartfiles',
    '-L' + sgx_library_path,  # Add SGX_LIBRARY_PATH here
    '-Wl,--whole-archive',
    '-l' + trts_library_name,
    '-Wl,--no-whole-archive',
    '-Wl,--start-group',
    '-lsgx_tstdc',
    '-lsgx_tcxx',
    '-l' + crypto_library_name,
    '-l' + service_library_name,
    '-Wl,--end-group',
    '-Wl,-Bstatic',
    '-Wl,-Bsymbolic',
    '-Wl,--no-undefined',
    '-Wl,-pie,-eenclave_entry',
    '-Wl,--export-dynamic',
    '-Wl,--defsym,__ImageBase=0',
    '-Wl,--gc-sections',
    '-Wl,--version-script=' + meson.current_source_dir() + '/EnclaveInitiator.lds'
]

sgx_common_flags += [
    '-Wall', '-Wextra', '-Winit-self', '-Wpointer-arith', '-Wreturn-type',
    '-Waddress', '-Wsequence-point', '-Wformat-security',
    '-Wmissing-include-dirs', '-Wfloat-equal', '-Wundef', '-Wshadow',
    '-Wcast-align', '-Wconversion', '-Wredundant-decls'
]

sgx_common_cflags = sgx_common_flags + [
    '-Wjump-misses-init', '-Wstrict-prototypes', '-Wunsuffixed-float-constants'
]

sgx_common_cxxflags = sgx_common_flags + ['-Wnon-virtual-dtor', '-std=c++11']

enclave_include_paths = include_directories(
    join_paths(sgx_sdk, 'include'),
    join_paths(sgx_sdk, 'include/tlibc'),
    join_paths(sgx_sdk, 'include/libcxx'),
    '../Include'
)

enclave_c_flags = [
    '-nostdinc',
    '-fvisibility=hidden',
    '-fpie',
    '-ffunction-sections',
    '-fdata-sections'
]

cc = meson.get_compiler('c')
cc_version = cc.version()
cc_below_4_9 = cc_version.version_compare('<4.9')
# 3. Conditional stack protection flag
if cc_below_4_9
    enclave_c_flags += '-fstack-protector'
else
    enclave_c_flags += '-fstack-protector-strong'
endif

enclave_cpp_flags = enclave_c_flags + ['-nostdinc++']

if get_option('LAv2')
  enclave_c_flags += '-DSGX_USE_LAv2_INITIATOR'
  enclave_cpp_flags += '-DSGX_USE_LAv2_INITIATOR'
endif


# Custom target to generate EnclaveInitiator_t.c and EnclaveInitiator_t.h
enclave_initiator_t = custom_target('enclave_initiator_t',
    output : ['EnclaveInitiator_t.c', 'EnclaveInitiator_t.h'],
    input : 'EnclaveInitiator.edl',
    command : [sgx_edger8r, '--trusted', '@INPUT@', '--search-path', sgx_sdk / 'include', '--trusted-dir', join_paths(build_root, 'EnclaveInitiator')]
)

# Define sources
enclave_cpp_files = [
    'EnclaveInitiator.cpp',
    'EnclaveMessageExchange.cpp',
    'Utility_E1.cpp',
    enclave_initiator_t
]


# Define the Enclave target
enclave_initiator = shared_library(
    'enclave_initiator',
    enclave_cpp_files,
    include_directories: enclave_include_paths,
    c_args: sgx_common_cflags + enclave_c_flags,
    cpp_args: sgx_common_cxxflags + enclave_cpp_flags,
    link_args: enclave_link_flags,
    name_prefix: 'lib',
    name_suffix: 'so'
)


if not fs.exists(enclave_test_key)
    message('There is no enclave test key<EnclaveInitiator_private_test.pem>.')
    message('The project will generate a key<EnclaveInitiator_private_test.pem> for test.')
    run_command('openssl', 'genrsa', '-out', join_paths(build_root, 'EnclaveInitiator',enclave_test_key), '-3', '3072', check: true)
endif

signed_enclave = custom_target('signed_enclave_initiator',
    input: enclave_initiator,
    output: signed_enclave_name,
    command: [
        sgx_enclave_signer,
        'sign',
        '-key', './EnclaveInitiator/'+enclave_test_key,
        '-enclave', '@INPUT@',
        '-out', signed_enclave_name,
        '-config', '../EnclaveInitiator/EnclaveInitiator.config.xml'
    ],
    build_by_default: true
)


