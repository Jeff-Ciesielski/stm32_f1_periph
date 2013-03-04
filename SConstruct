import os
import os.path
import ConfigParser
from subprocess import call

libname = 'stmstandardperiph'

defines = ['STM32F10X_CL']

include_dirs = ['./Libraries/STM32F10x_StdPeriph_Driver/inc',
                './Libraries/CMSIS/CM3/DeviceSupport/ST/STM32F10x',]

autoscan_includes = False

def find_source_files(file_list):
    for obj in os.walk('.'):
        for fil in obj[2]:
            if fil[-2:] == '.c':
                file_list.append("{0}/{1}".format(obj[0],fil))

def find_include_dirs(dir_list):
    for obj in os.walk('.'):
        for fil in obj[2]:
            if fil[-2:] == '.h':
                include_dirs.append(obj[0])

call(['clear'])

prefix = 'arm-none-eabi-'

env = Environment(ENV = {'PATH' : os.environ['PATH']},
                  #CPPPATH = libopencm3_root + '/include',
                  CPPDEFINES = defines,
                  CC =      prefix + 'gcc',
                  LD =      prefix + 'ld',
                  LINK =    prefix + 'gcc',
                  OBJCOPY = prefix + 'objcopy',
                  OBJDUMP = prefix + 'objdump',)


arch_flags = ' -mthumb -mcpu=cortex-m3 -msoft-float '

cc_flags = ''.join([' -Os ',
                    ' -g ',
                    ' -Wall ',
                    ' -fno-common ',
                    arch_flags,
                    ' -MD ',])

env['CCFLAGS'] = cc_flags

source_files = list()
find_source_files(source_files)

if autoscan_includes:
    find_include_dirs(include_dirs)

env['CPPPATH'] = include_dirs

full_lib_name = "lib{0}.a".format(libname)

env.Library(target = libname, source = source_files)

dumpasm = env.Alias('dumpasm',
                    full_lib_name,
                    "{0}objdump --disassemble {1} > {2}.asm".format(prefix,
                                                                    full_lib_name,
                                                                    libname))
AlwaysBuild(dumpasm)
