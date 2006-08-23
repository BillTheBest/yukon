

import SCons.Defaults
import SCons.Tool
import SCons.Util


envBase = Environment(
	CC = 'gcc',
	CPPPATH = ['#include', '/usr/include/ffmpeg'],
	CCFLAGS = ['-std=c99', '-pipe', '-O3'],
	LINKFLAGS = ['', ''],
	LIBS = ['dl', 'pthread', 'GL', 'GLU', 'X11'],
	AS = 'yasm'
)

static_obj, shared_obj = SCons.Tool.createObjBuilders(envBase)

shared_obj.add_action('.asm', SCons.Defaults.ASAction)
shared_obj.add_emitter('.asm', SCons.Defaults.SharedObjectEmitter)

envArray = {}

envArray['x86'] = envBase.Copy()
envArray['x86'].Append(
	CCFLAGS = ['-m32'],
	LINKFLAGS = ['-m32'],
	LIBPATH = ['/emul/linux/x86/usr/lib/'],
	ASFLAGS = '-f elf32'
)

envArray['amd64'] = envBase.Copy()
envArray['amd64'].Append(
	CCFLAGS = ['-m64'],
	LINKFLAGS = ['-m64'],
	ASFLAGS = '-f elf64'
)

yukonCoreSource		= [
	'yukonCore/yBuffer.c',
	'yukonCore/yTime.c',
	'yukonCore/yConfig.c',
	'yukonCore/yCompressor.c'
]


yukonPreloadSource 	= [
	'yukonPreload/yEngine.c',
	'yukonPreload/yHooks.c',
] + yukonCoreSource

yukonReplaySource	= [
	'yukonReplay/main.c',
	'yukonReplay/colorspace.c'
] + yukonCoreSource

yukonConvertSource	= [
	'yukonConvert/main.c'
] + yukonCoreSource


for arch, env in envArray.items():
	env.BuildDir('build/'+arch+'/', 'src')
	mmEnv = env.Copy()
	mmEnv.Append(LIBS = [ ])
	
	asm = [ 'build/'+arch+'/asm/'+arch+'/huffman.asm' ]
	lib = env.SharedLibrary(target = 'build/'+arch+'/yPreload.'+arch+'.so', source = [ 'build/'+arch+'/'+s for s in yukonPreloadSource ] + asm, SHLIBPREFIX='')
	replay = mmEnv.Program(target = 'build/'+arch+'/yReplay', source = [ 'build/'+arch+'/'+s for s in yukonReplaySource ] + asm)
	server = env.Program(target = 'build/'+arch+'/yServer', source = 'build/'+arch+'/yukonServer/main.c')
	env.Alias(arch, [ lib, replay, server ])
