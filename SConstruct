#
# SConstruct build file. See http://www.scons.org for details.
import os

class ConfigHeader:
    def __init__(self):
        self.defines = { }
        self.prefix = ""
        self.postfix = ""

    def SetPrefix(self, prefix):
        self.prefix = prefix

    def SetPostfix(self, postfix):
        self.postfix = postfix

    def Define(self, name, value = ""):
        self.defines[name] = value

    def Save(self, filename):
        file = open(filename, 'w')
        file.write("/* %s. Generated by SConstruct */\n" % (filename))
        file.write("\n")
        file.write(self.prefix + "\n")
        for key, value in self.defines.iteritems():
            file.write("#define %s \"%s\"\n" % (key, value))
        file.write(self.postfix + "\n")
        file.close()

def Glob(dirs, pattern = '*' ):
    import os, fnmatch
    files = []
    for dir in dirs:
        try:
            for file in os.listdir( Dir(dir).srcnode().abspath ):
                if fnmatch.fnmatch(file, pattern) :
                    files.append( os.path.join( dir, file ) )
        except Exception, e:
            print "Warning, couldn't find directory '%s': %s" % (dir, str(e))
        
    return files

# thanks to Michael P Jung
def CheckSDLConfig(context, minVersion):
    context.Message('Checking for sdl-config >= %s... ' % minVersion)
    from popen2 import Popen3
    p = Popen3(['sdl-config', '--version'])
    ret = p.wait()
    out = p.fromchild.readlines()
    if ret != 0:
        context.Result(False)
        return False
    if len(out) != 1:
        # unable to parse output!
        context.Result(False)
        return False
    # TODO validate output and catch exceptions
    version = map(int, out[0].strip().split('.'))
    minVersion = map(int, minVersion.split('.'))
    # TODO comparing versions that way only works for pure numeric version
    # numbers and fails for custom extensions. I don't care about this at
    # the moment as sdl-config never used such version numbers afaik.
    ret = (version >= minVersion)
    context.Result(ret)
    return ret

# Package options
PACKAGE_NAME = "SuperTux"
PACKAGE_VERSION = "0.2-cvs"
PACKAGE_BUGREPORT = "supertux-devel@lists.sourceforge.net"
PACKAGE = PACKAGE_NAME.lower()
PACKAGE_STRING = PACKAGE_NAME + " " + PACKAGE_VERSION

# User configurable options
opts = Options('build_config.py')
opts.Add('CXX', 'The C++ compiler', 'g++')
opts.Add('CXXFLAGS', 'Additional C++ compiler flags', '')
opts.Add('CPPPATH', 'Additional preprocessor paths', '')
opts.Add('CPPFLAGS', 'Additional preprocessor flags', '')
opts.Add('CPPDEFINES', 'defined constants', '')
opts.Add('LIBPATH', 'Additional library paths', '')
opts.Add('LIBS', 'Additional libraries', '')
opts.Add('DESTDIR', \
        'destination directory for installation. It is prepended to PREFIX', '')
opts.Add('PREFIX', 'Installation prefix', '/usr/local')
opts.Add(EnumOption('VARIANT', 'Build variant', 'optimize',
            ['optimize', 'debug', 'profile']))

env = Environment(options = opts)

# Create build_config.py and config.h
if not os.path.exists("build_config.py") or not os.path.exists("config.h"):
    print "build_config.py or config.h don't exist - Generating new build config..."

    header = ConfigHeader()
    header.Define("PACKAGE", PACKAGE)
    header.Define("PACKAGE_NAME", PACKAGE_NAME)
    header.Define("PACKAGE_VERSION", PACKAGE_VERSION)
    header.Define("PACKAGE_BUGREPORT", PACKAGE_BUGREPORT)
    header.Define("PACKAGE_STRING", PACKAGE_STRING)
        
    conf = Configure(env, custom_tests = {
        'CheckSDLConfig' : CheckSDLConfig
    })
    if not conf.CheckSDLConfig('1.2.4'):
        print "Couldn't find libSDL >= 1.2.4"
        Exit(1)
    if not conf.CheckLib('SDL_mixer'):
        print "Couldn't find SDL_mixer library!"
        Exit(1)
    if not conf.CheckLib('SDL_image'):
        print "Couldn't find SDL_image library!"
        Exit(1)
    if not conf.CheckLib('GL'):
        print "Couldn't find OpenGL library!"
        Exit(1)

    env = conf.Finish()

    env.ParseConfig('sdl-config --cflags --libs')
    env.Append(CPPDEFINES = \
        {'DATA_PREFIX':"'\"" + env['PREFIX'] + "/share/supertux\"'" ,
         'LOCALEDIR'  :"'\"" + env['PREFIX'] + "/locales\"'"})
    opts.Save("build_config.py", env)
    header.Save("config.h")
else:
    print "Using build_config.py"

if env['VARIANT'] == "optimize":
    env.Append(CXXFLAGS = "-O2 -g -Wall")
elif env['VARIANT'] == "debug":
    env.Append(CXXFLAGS = "-O0 -g3 -Wall -Werror")
    env.Append(CPPDEFINES = { "DEBUG":"1" })
elif env['VARIANT'] == "profile":
    env.Append(CXXFLAGS = "-pg -O2")

build_dir="build/" + env['PLATFORM'] + "/" + env['VARIANT']

env.Append(CPPPATH = ["#", "#/src", "#/lib"])
env.Append(LIBS = ["supertux"])
env.Append(LIBPATH=["#" + build_dir + "/lib"])

env.Export(["env", "Glob"])
env.SConscript("lib/SConscript", build_dir=build_dir + "/lib", duplicate=0)
env.SConscript("src/SConscript", build_dir=build_dir + "/src", duplicate=0)
Default("supertux")
