import os
import os.path
import ConfigParser
from subprocess import call

available_mcus = [  'stm32f1',
                    'stm32f2',
                    'stm32f4']

toolchains = {"CodeSourcery":'arm-none-eabi-',
              "Linaro":'arm-linux-gnueabi-'}

def set_linker_header():
    call(['clear'])
    print "Please enter the name of the linker script containing the memory map for your MCU (usually xxx-xxx.ld)"
    return raw_input(">>")

#TODO: build in automatic detection
def select_toolchain():
    valid_tc = False
    tcindex = [i for i in toolchains]

    while not valid_tc:
        call(['clear'])
        print("Please select from the following toolchains:")

        for tc in range(len(tcindex)):
            print "{0}: {1}".format(tc + 1, tcindex[tc])

        tc_sel = raw_input(">>")

        try:
            tc_sel = int(tc_sel)
            if tc_sel in range(1, len(tcindex) + 1):
                return toolchains[tcindex[tc_sel - 1]]
            else:
                raw_input("Please select from the available options")
        except:
            raw_input("Please enter a valid integer value")

def select_mcu():
    valid_mcu = False
    while not valid_mcu:
        call(['clear'])
        print "Please select from the following MCUs:"
        for i in range(1, len(available_mcus) + 1):
            print "{0}. {1}".format(i, available_mcus[i - 1])

        try:
            choice = int(raw_input(">> ")) - 1
            if choice not in range(len(available_mcus)):
                raise ValueError
            else:
                return available_mcus[choice]
        except ValueError:
            raw_input("Please enter a valid choice from the list [press enter]")

def locate_opencm3():
    valid_cm3 = False
    while not valid_cm3:
        call(['clear'])
        cm3_root = raw_input("Please enter the path to the root of libopencm3\n>> ")
        if os.path.exists(os.path.join(cm3_root, "HACKING")):
            valid_cm3 = True
        else:
            raw_input("Invalid Path [press enter]")
    return cm3_root

def create_config_file():
    cfg = ConfigParser.RawConfigParser()
    cfg.add_section('Build')
    cfg.set('Build', "cm3_root", locate_opencm3())
    cfg.set('Build', "mcu", select_mcu())
    cfg.set('Build', "link_header", set_linker_header())
    cfg.set('Build', "toolchain_prefix", select_toolchain())
    with open('build.cfg', 'wb') as configfile:
        cfg.write(configfile)

    return

def load_config_file(build_options):
    cfg = ConfigParser.RawConfigParser()
    cfg.read('build.cfg')
    build_options['cm3_root'] = cfg.get('Build', 'cm3_root')
    build_options['mcu'] = cfg.get('Build', 'mcu')
    build_options['link_header'] = cfg.get('Build', 'link_header')
    build_options['toolchain_prefix'] = cfg.get('Build', 'toolchain_prefix')
    return

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

build_options = dict()

if not os.path.exists('build.cfg'):
    create_config_file()

call(['clear'])
load_config_file(build_options)

libopencm3_root = build_options['cm3_root']
mcu = build_options['mcu']
target_linker_script = build_options['link_header']
prefix = build_options['toolchain_prefix']

env = Environment(ENV = {'PATH' : os.environ['PATH']},
                  #CPPPATH = libopencm3_root + '/include',
                  CPPDEFINES = [mcu.upper(),'BOARD_DEV_KIT', 'CONFIG_SHELL'],
                  CC =      prefix + 'gcc',
                  LD =      prefix + 'ld',
                  LINK =    prefix + 'gcc',
                  OBJCOPY = prefix + 'objcopy',
                  OBJDUMP = prefix + 'objdump',)

base_linker_script = "libopencm3_" + mcu + ".ld"
cmsis_lib = "libopencm3_" + mcu + '.a'

base_linker_script_path = libopencm3_root + "/lib/" + base_linker_script
cmsis_lib_path = libopencm3_root + "/lib/" + cmsis_lib

if not os.path.exists(base_linker_script):
    os.symlink(base_linker_script_path, base_linker_script)

if not os.path.exists(cmsis_lib):
    os.symlink(cmsis_lib_path, cmsis_lib)

arch_flags = ' -mthumb -mcpu=cortex-m3 -msoft-float '
linker_flags = ''.join([' -T ', target_linker_script,
                        ' -Wl,--start-group -lc -lgcc -Wl,--end-group ',
                        ' -nostartfiles -Wl,--gc-sections',
                        arch_flags,])

cc_flags = ''.join([' -Os ',
                    ' -g ',
                    ' -Waddress ',
                    ' -Warray-bounds ',
                    ' -Wchar-subscripts',
                    ' -Wenum-compare',
                    ' -Wimplicit-int',
                    ' -Wimplicit-function-declaration',
                    ' -Wcomment',
                    ' -Wformat',
                    ' -Wmain',
                    ' -Wmissing-braces',
                    ' -Wnonnull',
                    ' -Wparentheses',
                    ' -Wpointer-sign',
                    ' -Wreturn-type',
                    ' -Wsequence-point',
                    ' -Wsign-compare',
                    ' -Wstrict-aliasing',
                    ' -Wstrict-overflow=1',
                    ' -Wswitch',
                    ' -Wtrigraphs',
                    ' -Wuninitialized',
                    ' -Wunknown-pragmas',
                    ' -Wunused-function',
                    ' -Wunused-label',
                    ' -Wunused-value',
                    ' -Wvolatile-register-var',
                    ' -Wextra ',
                    ' -fno-common',
                    arch_flags,
                    ' -MD ',])
env['LINKFLAGS'] =  linker_flags
env['CCFLAGS'] = cc_flags

source_files = list()
find_source_files(source_files)

include_dirs = list()
include_dirs.append(libopencm3_root + '/include')
include_dirs.append('./include')
include_dirs.append('.')
#find_include_dirs(include_dirs)

env['CPPPATH'] = include_dirs

env.Program(target = 'cantaloupe.elf', source = source_files, LIBS=[cmsis_lib], LIBPATH='.')
env.Command('cantaloupe.bin', 'cantaloupe.elf', prefix + 'objcopy -Obinary cantaloupe.elf cantaloupe.bin')

flash = env.Alias('flash', 'cantaloupe.bin', './flash.py -ar')
AlwaysBuild(flash)

dumpasm = env.Alias('dumpasm', 'cantaloupe.elf', prefix + "objdump --disassemble cantaloupe.elf > cantaloupe.asm")
AlwaysBuild(dumpasm)

dfu = env.Alias('dfu', 'cantaloupe.elf', 'sudo ./flash.py -d')
AlwaysBuild(dfu)

autoflash = env.Alias('autoflash', 'cantaloupe.bin', 'sudo ./flash.py -dar')
AlwaysBuild(autoflash)
