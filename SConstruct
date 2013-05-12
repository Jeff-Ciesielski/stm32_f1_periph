import os

defines = []
mcu_families = ['STM32F10X_LD',
                'STM32F10X_LD_VL',
                'STM32F10X_MD',
                'STM32F10X_MD_VL',
                'STM32F10X_HD',
                'STM32F10X_HD_VL',
                'STM32F10X_XL',
                'STM32F10X_CL']

#these are in respect to the 'Libraries' folder

prefix = 'arm-none-eabi-'
arch_flags = ' -mthumb -mcpu=cortex-m3 -msoft-float '

cc_flags = ''.join([' -Os ',
                    ' -g ',
                    ' -Wall ',
                    ' -fno-common ',
                    arch_flags,
                    ' -MD ',])

envs = list()
for i in range(len(mcu_families)):
    family = mcu_families[i]
    envs.append(Environment(ENV = {'PATH' : os.environ['PATH']},
                            CC =      prefix + 'gcc',
                            LD =      prefix + 'ld',
                            LINK =    prefix + 'gcc',
                            AS =      prefix + 'as',
                            OBJCOPY = prefix + 'objcopy',
                            OBJDUMP = prefix + 'objdump',
                            CCFLAGS = cc_flags))
    envs[i]['MCU_FAMILY'] = family
    envs[i]['CPPDEFINES'] = defines + [family]

for env in envs:
    env.SConscript('Libraries/SConscript', variant_dir='libs/.{0}'.format(env['MCU_FAMILY'].lower()),
                   duplicate=0, exports='env prefix')
