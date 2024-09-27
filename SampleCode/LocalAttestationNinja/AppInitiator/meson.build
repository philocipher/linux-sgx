
if sgx_mode != 'HW'
  urts_lib_name = 'sgx_urts_sim'
else
  urts_lib_name = 'sgx_urts'
endif

# Include directories
inc_dirs = [
  sgx_sdk / 'include',
  '../Include'
]

# Libraries
libs = [
  urts_lib_name,
  'pthread'
]

links =  ['-L' + sgx_sdk / 'lib64']

cxx_flags = []
c_flags = []
if get_option('SGX_DEBUG')
  cxx_flags += ['-DDEBUG', '-UNDEBUG', '-UEDEBUG']
  c_flags += ['-DDEBUG', '-UNDEBUG', '-UEDEBUG']
elif get_option('SGX_PRERELEASE')
  cxx_flags += ['-DEDEBUG', '-DNDEBUG', '-UDEBUG']
  c_flags += ['-DEDEBUG', '-DNDEBUG', '-UDEBUG']
else
  cxx_flags += ['-DNDEBUG', '-UEDEBUG', '-UDEBUG']
  c_flags += ['-DNDEBUG', '-UEDEBUG', '-UDEBUG']
endif



# Custom target to generate EnclaveInitiator_u.c
enclave_initiator_u = custom_target('enclave_initiator_u',
    output : ['EnclaveInitiator_u.c', 'EnclaveInitiator_u.h'],
    input : '../EnclaveInitiator/EnclaveInitiator.edl',
    command : [sgx_edger8r, '--untrusted', '@INPUT@', '--search-path', sgx_sdk / 'include', '--untrusted-dir', join_paths(build_root, 'AppInitiator')]
)

# Define your sources
sources = [
    'App.cpp',
    'UntrustedEnclaveMessageExchange.cpp',
    '../util/fifo.cpp',
    enclave_initiator_u
]


# Define your executable
appinitiator =executable('appinitiator',
    sources,
    include_directories : inc_dirs,
    cpp_args: cxx_flags,
    c_args: c_flags,
    link_args: links,
    dependencies: [
    meson.get_compiler('cpp').find_library(urts_lib_name),
    meson.get_compiler('cpp').find_library('pthread')
    ],
    install : true
  )

# Move the built executable to the parent folder
custom_target('move_executable',
    output : 'move_executable',
    command : ['cp', appinitiator, build_root],
    depends : appinitiator,
    build_by_default : true
)