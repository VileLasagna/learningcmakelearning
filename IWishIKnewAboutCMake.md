

# Some stuff I wish I was taught straight away about CMake

It was a long time coming but finally I had an excuse to push CMake up my
learning priority queue. So why the heck CMake, then? Which sounds like a silly
question to be asked of anyone actually reading through something like this,
maybe.

## Foreword: Do I actually want to use CMake?

Well, if you're still debating whether it's worth or not to learn CMake and use
it for your projects, be them professional or hobbyist, I will just go ahead and
recommend it.  CMake is damn bloody powerful and not too clunky to use. I, for
the longest time stuck to QMake, as the thing I'd learnt and knew how to wrestle
pretty efficiently and as much I will still defend QMake to be much better and
more versatile than people give it credit for (i.e: It's a perfectly fine multi
platform build system for projects even if you do not use Qt itself) it still
has some annoying quirks and shortcomings (like how some crazy people keep
insisting there are IDEs they'd like to use over QtCreator or how QMake is
actually tied to the specific version of Qt you're using (I mean, this is
proper weird, as there's a binary for Qt 5.7, another for 5.8...)). Regardless,
not only it is being phased out itself in favour of Qbs, but we CAN do much
better.

Though not free of occasional rage and confusion my time with CMake has
been pretty good and this is a build system that will go pretty far in giving
you tons of power over your build. The multi-platform support is great and you
are given the power to make use of native tools. And it's crazy extensible,
something other people have already made use of and made your life easier.
Probably one of the things that most attract me is CMake's management of
"packages" which can be created and "found" to help you manage dependencies.
Many of which through built-in modules which helps you keep your paths and
linking somewhat sane. I'll talk a bit more about them later but, TL;DR:
 > Oh no! Now I have to sort out THIS and link it in?
 >
 >  ```
 > find_package(SomeMonster)
 > target_link_libraries(MyLovelyProject SomeMonster)
 >  ```
 >
 > Congratulations! You have you library linking, your includes pointed to
 > your language standard requirement notified and any compile flags or defines
 > needed already forwarded

It's a level of power I haven't found anything else and it's fantastic. The more
complex your projects become, the more you feel this is saving you from all the
nightmare that might be managing all that.

I am, however, a firm believer that as long as some magic like that remains
magical it actually means you're not understanding what's going on and, thus,
are unable to make any use of if other than just downloading someone else's
project, typing the commands and just hoping it works. And this is what this is
about.

## How is *THIS* document any special?

CMake has good documentation.

This is a fact, if you disagree you are way too blessed. Go spend half an hour
trying to understand any Windows API function through MSDN... you'll agree.

CMake's documentation though, is good reference. It doesn't do too great a job
in "teaching" you how to use it. It mostly assumes you need to know how each
individual thing works and does a good job at that. And it is pretty extensive
and comprehensive.

Likewise, most of the CMake learning material elsewhere fall in one of two
categories, by and large.
On the one hand you have some step by step tutorials which get you building some
come, all right. There's a decent official one, even.
And on the other hand there's a pretty good number of _StackOverflow_ questions
with often good answers that will help you with a particular trick or situation.

Where all that is lacking, I feel, is in teaching you _the mindset_ of writing
CMake, in a way. A lot of the concepts and jargon that underlie how this works.
And to me, that is central to demystifying a lot of what CMake does for you, and
then getting CMake to do it when you want it to.

### TL;DR

I intend to go on a bit over these core principles to try and help you
understand how CMake is doing things or how it expects you to do those things.
Also how other people who are more experienced with CMake will expect them. So
a mix of basic introduction with a splash of _Good Practices_ for good measure.
Though there are some good documents on this, I still think there are some
things I had to sort of piece together on my own and would have liked to have
seen before I started doing my own stuff. With that in mind, on we go.


## The Basics

#### What IS CMake?

We normally call CMake a build system but that is not entirely accurate, though
you can say it's not wrong either. CMake is a _meta build system_. This is, in
the end, technicality to an extent, but still useful to keep in mind. A normal
__Build System__ takes a bunch of instructions on what tools to call and a list
of the stuff you want built, makes the necessary calls and delivers you the
finished product. Be it a library or an executable...

CMake doesn't actually do that.

Because CMake is a __Meta build system__ and those take the same sort of input
and, instead, generate instructions for those build systems, which you may then
call to get your stuff built.

So CMake won't call `/usr/bin/clang` or `cl.exe`, instead it will generate the
appropriate build files and call `/usr/bin/make` or `nmake.exe` which then will
do all the actual calls to the compilers and linkers.

This is part of what makes CMake so flexible across different platforms. In the
end it will leverage whatever native tools you specify and they will get that
work done. So keep in mind that there is "another layer" hidden, so to speak.

#### CMakeLists.txt - The source of it all

One of the practical things about CMake is that all your build system code
is defined in one (or more) plain text file that is pretty human-readable.
This file is always named __CMakeLists.txt__ and is what CMake always looks for.
Different components in your project will generally have each their own
CMakeLists.txt and when you use `add_subdirectory(SomeDir)`, that's exactly
what CMake is looking for in that directory.

#### CMakeCache.txt - Your own little sandbox

When CMake goes and processes your CMakeLists it creates a file called
**CmakeCache.txt**. This file contains all the different CMake variables as they are
configured specifically for that build of your project. So paths that might have
been found, library names of detected dependencies, compiler flags, deployment
paths, etc.

This is the file you might have to tweak when, for instance, CMake tells you
"I can't find OpenCV. Can you tell me where it is?". The main pane of the
**cmake-gui** tool will show you precisely variables on that cache. And will
write there the variables and flags you set.


#### A function based build system

If you compare CMake to QMake (which I often do given QMake is a system I've
used for quite a while) there is a "philosophical" difference between them.
Where as in QMake you'd say something like:

```
SOURCES += main.cpp
TARGET = MyProj
LIBS += -lSomeLib
DEFINES += MyDef
```

in CMake, the equivalent is something more like:

```
set(MYPROJ_SOURCES main.cpp)
add_executable(MyProj ${MYPROJ_SOURCES})
target_link_libraries(MyProj SomeLib)
target_compile_definitions(MyProj MyDef)
```

In CMake every statement is a function or a macro (which works mostly like functions, but there are some [differences](https://cmake.org/cmake/help/v3.0/command/macro.html), even loops. This can be a bit strange
in the beginning but most of them aren't too arcane once you get a feel for the
mindset behind them. So rather than tweaking something, more likely than not,
each CMake statement is giving a more direct command.

#### Casing conventions

Once upon a time it was considered good practice to have all your CMake commands
in capitals, so... rewriting the previous code block

```
SET(MYPROJ_SOURCES main.cpp)
ADD_EXECUTABLE(MyProj ${MYPROJ_SOURCES})
TARGET_LINK_LIBRARIES(MyProj SomeLib)
TARGET_COMPILE_DEFINITIONS(MyProj MyDef)
```

CMake doesn't really care either way but it was decided that doing things like
that is ugly and reeks of macro hell. Instead, the more modern conventions are
that function names are all in lower case, using_underscores_where_appropriate
and then variables are left in UPPER_CASE in order to make them more easily
distinguishable. Keeping this in mind might be an easy way to spot very old
code if you come across it.

#### Server Mode

With the growing popularity of CMake a lot of interest grew on using CMake as a
more general development-ready build system. Traditionally if you want to work
on a CMake project rather than just getting it to build, you'd probably want
CMake to generate you either a Code::Blocks project or (eeeeewwww) a Visual
Studio solution, as those IDEs give you a lot more "interaction" so to speak,
helping you add and remove files, parse code, walk through debugging....

CMake recently, from version 3.7, CMake, working with some IDE vendors (at least
the people from Qt, that I know of) added a new feature called _Server Mode_.
**cmake-server** is a special mode that provides additional information for IDEs
and helps them interpret all that through JSON. As it's recent, it's a work in
progress, but it's bound to make it much easier for IDEs to support CMake
natively.

### Integrated Development Environments

And with that, we can talk a bit about IDEs themselves. Going all kate and bash
is fine but a good IDE does help a lot in managing all the things and without
some good IDE support, even a good system will still be painful to use. So I'm
using this section to talk about some of the strong and weak points of IDEs I've
tried which have CMake support (or for which CMake has unilateral support).

> This DID turn out pretty long though so if you want a TL;DR: _"Qt Creator is
> amazing, use it. Clion is also really good, do consider it. Visual Studio is...
> geez, no need for martyrdom guise."_

#### Qt Creator

This is what I've been using by and large. I'm a long fan of the IDE and
in general it is the measuring stick I apply to every other thing I come
across (_"hurr durr, I can go and download QtC for free. Are you even as
good as that?"_)

A disclaimer that this is what I use so it'll be overwhelmingly the one of
which I have the most to say. 100% objective and unbiased opinion, if you
use anything else you're just wrong and should use this instead.

Qt Creator has long had support for CMake but I will say that my initial
contact, way back, was pretty disappointing. I felt a bit that the support
was kinda lack lustre and it was one of the things which kept me to QMake
(I guess Qt never made enough noise about Qbs to push me into learning that)
It's support has been steadily getting better and although some things still
have me disappointed such as the fact that if I want to create a new class
I have to pop Guake down and `touch newclass.{h,c}pp` and
then manually add them to the CMakeLists.

It's still disappointing but a small detail and the rest of what Qt Creator
offers has made me stick to it.

If you're not familiar, Qt Creator works with the concept of "kits". A kit
is a "collection of settings" which include
 - A Qt version
 - A C compiler
 - A C++ compiler
 - A debugger backend
 - A deploy device (desktop, android phone...)
 - A Cmake executable
 - Some CMake configurations
 - Possibly some alternative QMake configurations

That's a whole bunch of stuff and in those CMake configurations you can
specify which generators you want to use with that kit as well as any
additional defines you want to pass CMake. By default QT adds its own root
to the CMAKE_PREFIX_PATH as well as the exectuables for the compilers you've
set up in the kit and for the QMake executable associated with it.

That already gives you some good help _but wait, there's more!_

In addition to that Qt Creator has, on it's _Projects_ tab, a visual
parser of the CMakeCache which is pretty helpful when tweaking variables
and options for your build. It is very reminiscent (possibly most of the
very same code, I'd guess) of the _cmake-gui_ editor.
Finally, something that QtCreator always makes me miss on other IDEs is
the ability to set up custom build and deploy steps. Adding either
`cmake -- install` or `make package` (if you're using CPack) as a deploy
step is generally super helpful and something I myself may have got way too
used to as it's always something I look for (and often don't find) in other
environments. Combined with it being a pretty strong debugging frontend,
having native valgrind integration, great code completion, very configurable
editor with the best code highlighting I've seen anywhere... yeah, this IDE
is pretty amazing. It IS a very tall stick to measure things by.

That said it's not without its own flaws and annoyances.

I've mentioned CMake Server Mode earlier and Qt is jumping on it. So if you
intend to use Qt Creator to work with CMake, you DO want a version later
than **4.3** which, right now is in beta. You can look for the Qt blog post
on this if you're curious, but the support has got drastically better with
this version.

That gui-like cache editor too, as helpful as it is, will not show up if
your CMake "configure" step fails, even if it'd be the easiest way to get
it working. It's driven at least one decision I've made regarding my own
CMake practices.

And... maybe stretching, but it does seem to have some annoying bug with
Clang+C2 toolchain on Windows (because of course) where kits configured
with that compiler will sometimes lose their configuration and you'll
notice the wrong compiler being called in the terminal. I don't know why
this happens but it is very much a Qt Creator bug in some way, not a CMake
related thing (it'll happen to your QMake projects as well, as the kit
itself needs to be fixed). For me that seems to happen every other time I
run the IDE. And it fails silently before by apparently failing to run
`vcvarsall.bat`. It's an easy fix, you just restart the IDE, make sure the
kits show as having no compiler set up, re-select the compiler from the
drop down menu and it'll work again as it should have the whole time. Again
this is specific to that toolchain in Windows so... I dunno. Additionally,
because QtCreator doesn't consider Clang+C2 a C compiler if that's what you
want to use (which you should if the other option is VC) you might want to
go in the CMake Options and tweaking the CMAKE_C_COMPILER option to point to
the same thing as the CXX compiler. QtCreator will tell you that looks weird
but will do it all the same, as it should. And you should be fine

####JetBrains Clion

Clion is a favourite among quite a few CMake veterans. It's yet another
IDE JetBrains built on top of IntelliJ, their Java IDE and they seem to
know their stuff because their IDEs seem to always be rated among the best
available for each one. Clion is no exception there. It is more closely
integrated with CMake than QtCreator. It has better completion and more
awareness of how the project should related to the CMakeLists. It is
definitely worth a check. From what I understand, JetBrains puts a lot
of value in making sure their IDE is helpful in getting tons of code
refactoring done easy. Things like adding inheritance on the fly, renaming
functions... If you're curious it might be interesting to search YouTube
for a "Clion Tips and Tricks" talk that happened at CppCon2015. It's 15
minutes long and looks pretty impressive if that's the way you like working.
JetBrains are also the people behind _ReSharper_ so their code completion
is pretty strong. I have considered this IDE pretty strongly a few times
but it unfortunately couldn't win me over yet. The points where it lacks,
either by itself or by comparison, are the following

 - Cache editor

    Whereas Qt Creator has that nifty tool, Clion just gives you quick
    access to open _CMakeCache.txt_ in their text editor quickly and work
    from there. It's not bad, but knowing there can be much more, I keep
    wanting more.

 - Custom build and deploy steps

    This is the deal breaker for me. Namely for not giving me access to
    calling "install" from the IDE. It keeps sounding small but it's a thing
    I find myself less and less willing to give away. _THAT SAID_ JetBrains
    are aware people want this and there's a feature request being looked
    at referring to this. So it may very well come in a version in the near

 - No Support for Clang+C2

    VC is a terrible compiler and no one should ever have to use it. So far,
    so good, as Clion doesn't normally support VC. You have to enable it in
    a hidden menu to remind you of the terrible mistake you're making. What
    Clion gives you instead is the option of using either MinGW or Cygwin.
    As far as I'm concerned that is fine and the way I want MY stuff to
    happen when I need to deploy them to Windows. However the world is a big
    place and sometimes you need to link to standard VC. Clion has support
    for the VC toolchain, yes, but just the standard one. It goes ahead
    and detects `cl.exe` and sets that up as your compiler and there's not
    much you can do about it (that I have found, at least) which works, but
    then you're using a horrible compiler that will make your life harder
    and produce worse code.

 - Eeeewwww, Java!

    Jokes aside, the fact that IntelliJ, is built on Java, makes that a
    feature of every JetBrains IDE which might mean some systems may
    struggle a bit with it. I say this not from a theoretical point of
    view but from an empyrical one. Currently I call my  main development
    machine at home _"Crappy Lappy"_ and that name is pretty adequate (if
    you wonder how crappy, [It's one of these](https://support.hp.com/nz-en/document/c05243218 "Official specs") )
    It seems to struggle quite a bit with Clion parsing all the things.
    Not sensible on a properly powerful dev machine but, apparently we
    can't always have one handy. Again, comparatively, Qt Creator seems
    to be easier on it.

 - It costs real moneys

    Clion is not crazy expensive and the from what I've used of the IDE,
    it's money very well spent. When I've looked at it it hasn't been the
    greatest factor in me not going for it, but it's there, every penny it
    costs is a penny it costs over what QtCreator costs, and as I said, that
    is an excellent IDE you're getting. So... there's that factor as well.
    They do offer open source licenses if you contribute to Open projects
    and have a 30 day trial I've probably taken two or three times now. So
    it's easy to check it and see if it works for you.

#### Visual Studio

Have a penny, go download a good IDE.

Jokes about VS being horrible aside (it's not, it's just pretty bad) if you
do want to use it, CMake helps you a lot. One of the available generators
for Windows is called _Visual Studio <someVer> <SomeYear>_ and what it does
for you is generate a Visual Studio solution for you to work on. I can't
tell you much about it because once I knew Qt Creator I just could never
go back to Visual Studio without being completely horrified at it so there's
about 6 years I don't really use that IDE much any more but the generated
solution is actually really good, I've been told. It tracks changes to the
CMakeLists and helps a bit with that. I've also heard rumours that _VS 2017_
might have actual CMake support and I would imagine that is pretty good news
as far as I'm concerned since solutions and msbuild are one of the things I
abhor in the IDE. But, again, given I avoid it as much as I can, I'm
potentially not qualified to talk meaningful details about this, nor confirm
the truth in this.

So if you want to insist on this, you probably want to open your project
with **cmake-gui** and picking the right version of Visual Studio from the
available generators and then open the solution with the IDE.

#### *Code::Blocks* and *KDevelop*

VS is an IDE I have barely touched in years. CodeBlocks is an IDE I have
barely used at all so, again this is something I can't comment deeply on
but if it's something you've used and/or liked, this another "project"
generator that CMake provides.

There is also one for KDevelop on Linux, so another possibility to be
checked out if you're comfortable there.

There are a few more also listed but these keep slipping more
and more distant from my own familiarity. I'm mentioning them more on this
idea that, they're a thing, if you like it, you might want to check it out.
You can look for more information on [CMake's official docs](https://cmake.org/cmake/help/v3.8/manual/cmake-generators.7.html "Cmake-generators")

## Lexicon - all the mumbo-jumbo

So a lot of what's difficult about learning any new tech really is getting
around the specifics of the lexicon around it. It may not seem at first but then
you remember that in C++ you have _structs_ and _classes_ which are the same
thing but not really except exactly that. And if you're learning Java, you come
across _Primitive Types_ which have their own special rules and for some reason
are treated differently than other variables.

CMake, like any proper tool has it's own language which seems made to confuse
any new users. And I feel that behind them are some of the important concepts to
wrapping your head around how CMake works.

### Generators

Generator are, in a way, the "templates" for CMake. CMake works in two steps:
**Configure** and **Generate**.
In the _Configure_ step, CMake reads through your CMakeLists and creates a
bunch of temporary build files, includind _CMakeCache.txt_. Afterwards, when
you've tweaked all you need, if anything, once you _Generate_, cmake writes the
output build files for the Generator you've chosen.

The most common ones are either **Unix Makefiles** or **NMake Makefiles** but
as I mentioned in the IDE section, there are also some that generate project
files for specific IDEs. CMake can do both, as some of these generators (such
as the _CodeBlocks_ generator) are considered "Extra Generator" so you have your
build instructions for whatever native tool you're using AND some project files
to check things out in an IDE.

If you use **CPack**, CMake's own packaging tool, that also has its own set of
generators, each outputting a different type of package such as _7Z_, which
creates a zipped package with files you've chosen or _IFW_, which creates a
graphical installer built on the Qt Installer Framework (basically exactly like
the ones you get to install Qt itself from their website). You can check
Cpack's own documentation and `cpack --help` for more info on those.


### Build Tree

Way back I mentioned a CMake command called `add_subdirectory(SomeProj)`.
That command does what it says on the tin, so speak, but it also exposes a
concept upon which CMake is build, the **Build Tree**.
When you call CMake, it looks for a _CMakeLists.txt_ and sets that as the root
of the build tree. Projects which you add as subdirectories may or may not be
actually independent projects on their own.

Lets imagine a project

```
<SomePath>/OurProject
              CMakeLists.txt
              /myApp
                  CMakeLists.txt
              /myLib
                  CMakeLists.txt
```

now, assuming myApp and myLib are both added as subdirectories and that myApp
uses myLib and links it in. Let's say you want to reference, in myLib, the root
folder for it, for wahtever reason, let's say defining include paths. CMake has
a variable you're likely to come across very soon in examples,
`CMAKE_CURRENT_SOURCE_DIR` which points to the directory you're reading the
source code from, where you have your CMakeLists is. If, in myLib/CMakeLists
you enter `message(${CMAKE_CURRENT_SOURCE_DIR})` you will get
`<SomePath>/OurProject` as an output.

That might be strange at first, but your Source Dir is the root of your build
tree. What you'd be looking for in this case is `CMAKE_CURRENT_LIST_DIR`, which
gives you the path to the _CMakeLists.txt_ that's currently being processed,
regardless of the build tree structure.

This becomes more relevant when you're talking about variables since variables
set for a father node will be inherited by the child nodes etc... So projects
don't necessarily exist in isolation.

### Target - The building block

This was probably one of the most "revealing" realizations for me personally.
CMake revolves around **targets**. They are the fundamental block of how CMake
wants you to think about your project. Basically _nothing means much to CMake
unless it's tied to a target_. And while this is not strictly true, it's a good
way to start thinking about this. Let's go back to QMake, my old comparison. In
QMake, you're assumed to have a target per .pro file. A **target** here, as it
is for both systems, is generally either an _executable_ or a _library_ you're
going to ask it to build. CMake has special _custom targets_ which are used for
custom operations or a bit of magic here and there, but libs and bins are the
important ones.

So, in QMake you might want to do something like

```
INCLUDEPATH += <SomePath>/include
LIBS += -L<SomePath>/deps -lsomeDep
DEFINES += USE_DEP

```
But in CMake, the correct way to do this is to tie them to a target. So instead
you should use

```
add_executable(MyApp ${MYAPP_SOURCES})

target_include_directories(MyApp <SomePath>/include)
target_link_libraries(MyApp someDep)
target_compile_definitions(MyApp "USE_DEP")

```

These includes, libs, defines... they don't belong to "our project", they
belong to _MyApp_. And for CMake this is crucial in getting things to work
properly as those commands will populate properties in that target that CMake
will use.

Spoiler warning that the last bit on this section is about something in CMake
called _"Property transitivity"_ and that's where this will hit for real. But
for now, keep in mind that your CMakeLists pretty much doesn't start until you
declared your **target**, as that's what you'll need to tie all the things to.

### Library Types

Aside from the standard `STATIC` and `SHARED` libraries we've come to expect,
CMake understands some additional types of libraries which you may want to be
aware of.

 - `INTERFACE`
    Interface libraries are your header-only libraries. And CMake treats them
    specially because with no source files, they can't normally be a target
    since they have no output. But you may still want to have them so you can
    easily use them. So this is a convenience type for that.
 - `OBJECT`
    An Object library is an ever cruder form of a static library. Sometimes
    you have code that is like a library but not really. You just want to get
    that partial compile in and then use it. That's where an Object library
    comes in. You do that compile and then, instead of linking it in, you can
    just tell CMake to "add these object files to the target too".
 - `ALIAS`
    That's what it sounds. You can use it if you imported an alternative version
    of a library or if you intend to have your library used in lieu of a more
    common one.

You can find more information about these on CMake's documentation,for
[the add_library() command]("https://cmake.org/cmake/help/v3.8/command/add_library.html")

### Packages

One of the first bits of "arcane magic" you may come in contact with when getting
to grips and using CMake will probably be along the lines of something like this:

```
find_package(SDL)
target_link_libraries(MyApp ${SDL_LIBRARIES})
target_include_directories(MyApp ${SDL_INCLUDE_DIRS})

```
The whole `find_package()` thing is a very powerful and very common CMake thing.

What that line does is call one of two CMake scripts. It'll look for either a
`FindSDL.cmake` or a `SDL-config.cmake`

The former is a legacy format. Though still used, it is being phased out. In
CMake documentation this is known as a _MODULE_ package. CMake actually ships
with a ton of these by default. You can check for them in [CMake's
documentation]("https://cmake.org/cmake/help/v3.8/manual/cmake-modules.7.html")
and have a look at the individual files in either CMake's installed folder or
`/usr/share/cmake-<version>/Modules` depending on your OS.

What these are in rough terms are scripts that in general work through calls of
`find_library()` and  `find_path()` in order to try and locate the includes and
library files related to the _package_ you're looking for. These files generally
will set variables such as those in the example, so you can add the relevant
commands to your own code.

More modern however will be in the latter of those two formats. Those are called,
conveniently, _CONFIG_ packages and in general do a bit more work. Some libraries
will install their config scripts to your system when installed, for my system,
for example (Arch Linux) the standard OpenCV package does this, and those files
are located in `/usr/share/opencv`. I know that OpenCV also distributes these
with their Windows binaries so that's somewhere you can look for a production
example, so to speak.

There are two main reasons why _CONFIG_ packages are better than _MODULE_
packages, and that's not even going into the fact that they're much cleaner in
your code.

The first reason is that _CONFIG_ packages are constituted of a few different
files on one of them will be called something like `OpenCV-config-version.cmake`
This _config-version_ file gives you power to control version compatibility. That
file enables you to tell users of your package whether they can expect to have
compatibility broken never, on each different major version, for ever single
version.... It also allows users to ask for a specific version and CMake will
try to find it according to these rules.

Second reason is more teaser material, sorry. I mentioned, back when I was
talking about _targets_ that there's a thing called _property transitivity_ and
_CONFIG_ packages are made to make use of that, and make your life much easier.
What I'll say for now is that they make the process of using them much less
error prone and simpler.

Probably more exciting is the fact that it isn't too difficult for you to provide
such packages for libraries of your own. CMake has a [CMakePackageConfigHelpers]("https://cmake.org/cmake/help/v3.8/module/CMakePackageConfigHelpers.html")
module you can include to precisely help you with that. It's worth a look once
you're ready to dive into this.

### *Source Folder* and *Binary Folder* (*List Folder* makes a return cameo)

I jumped the gun a little bit and talked some about the source folder before
but I'm putting everything together right now just for some perspective.

When a developer hears **Source Folder** the pretty obvious response is

> "OBVIOUSLY the folder with my source files"

And for **Binary Folder**

> "Pretty evidently the folder for my compiled binaries"

And both are predictably wrong in this case.

When CMake hears something like `CMAKE_SOURCE_DIR` what it means is,
as mentioned before, the directory containing the _CMakeLists.txt_ for the root
of the project currently opened, whereas `CMAKE_CURRENT_SOURCE_DIR` means the
directory containing the _CMakeLists.txt_ that is currently being processed:

And while `CMAKE_BINARY_DIR` is a bit closer to what we'd expect, what
it means is _"That folder where I'm going to write CMakeCache.txt and build all
the things! ALL THE THINGS!"_.
`CMAKE_CURRENT_BINARY_DIR` follows the came rules of `CMAKE_CURRENT_SOURCE_DIR`.
For every directory included with `add_subdirectory()`, a subfolder is created
inside the `CMAKE_BINARY_DIR`, that folder is `CMAKE_CURRENT_BINARY_DIR`.

As we mentioned before too, the last one of these is `CMAKE_CURRENT_LIST_DIR`
which is similar to current source dir, but relates to the CMakeLists file it
is used in.

A bit confusing, yes? So let's just hack out a quick mock-up example.

```
projDir $ ls -R

CMakeLists.txt

projDir/lib/
    CMakeLists.txt

    projDir/lib/cmake
        someScript.cmake

build/
    CMakeCache.txt
    Makefile
    build/lib/
        Makefile

```
Okay, now for these files, let's imagine

projDir/CMakeLists.txt has a line that reads `add_sudirectory(lib)`

projDir/lib/CMakeLists.txt has one that says `include(cmake/somescript.cmake)`

and

```
projDir $ cat ./lib/cmake/someScript.cmake

message("Current Binary dir = ${CMAKE_CURRENT_BINARY_DIR}" )
message("Current Source dir = ${CMAKE_CURRENT_SOURCE_DIR}" )
message("Current List dir = ${CMAKE_CURRENT_LIST_DIR}" )

message("Binary dir = ${CMAKE_BINARY_DIR}" )
message("Source dir = ${CMAKE_SOURCE_DIR}" )

```

When we run CMake and it reaches _someScript.cmake_  it'll process these
messages and the output to the console will be:

```

Current Binary dir = projDir/build/lib
Current Source dir = projDir/lib
Current List dir = projDir/lib/cmake

Binary dir = projDir/build
Source dir = projDir

```

It's a little strange at first to wrap your head around this and it's a result
of CMake's build tree structure. Entering different _CMakeLists.txt_ will update
your *CURRENT* Build and source folders, but you'll always have a reference to
the root of the tree and *CMAKE_CURRENT_LIST_DIR* always points to the
location of the current file being processed, even if it is a module, or script.
Incidentally, you also have *CMAKE_CURRENT_LIST_FILE* as well as a
*CMAKE_CURRENT_LIST_LINE*, this time, giving you the information they sound
like they should


### In-Source Builds vs Out-of-Source Builds

> You want to build your projects _Out-of-source_. Always. Every project. FIN~

That said AND being overwhelmingly true, what DOES it mean, though. Let's imagine

```
~ $ cd projects
~/projects $ git clone <Google Protobuf's repo here> protobuf
~/projects $ cd protobuf
~/projects/protobuf $ cmake .

```
Besides the fact that you may be "oof, building protobuf. Coffee time..." that
is pretty reasonable, especially considering CMake IS in fact protobuf's official
build system. And while that is all fine, when I joked "ALL THE THINGS" that was
sadly somewhat real. CMake will produce a TON of intermediate files and if what
you did looks like that example, all of that will go right together with your
project files and source code.

This is what is called an **In-source build**, and you can find a project you
know and build it and you'll see just how big of a mess it is. Some projects
attempt to prevent in-source builds, but CMake will always generate some things
before you can tell it to stop. So... do it once. On a controlled environment
where you can marvel at the horror of what you've done and never ever do it
again.

Instead, what you want is something more like:

```
~ $ cd projects
~/projects $ git clone <Google Protobuf's repo here> protobuf
~/projects $ cd protobuf
~/projects/protobuf  $ mkdir build
~/projects/protobuf  $ cd build
~/projects/protobuf/build $ cmake ..

```

What this achieves is that all that garbage gets put inside that _build_ folder.
Keep in mind to do this, pain will ensue if you don't. I know both **QtCreator**
and **Clion** do try and help you do this by default and **cmake-gui** does
display a field for you to choose a build folder which should be separate.
Just... keep in mind... no in-source if you intend to retain your sanity.

### install(EXPORT)  export(TARGETS)

These two similarly-named commands mean annoyingly different things.
You'll come across the first one a lot as it's a key command when generating
packages. It creates the file that describes the targets you're exporting
through said packages.

`export(TARGETS)` works in a similar way except the file generated is specific t
o the source tree it was generated in. So it's the same, except the
complete opposite. Most of the time you want `install(EXPORT)` but [docs]("https://cmake.org/cmake/help/v3.8/command/export.html")
give a few examples on where you might need the other. Just keep in mind that
**export** will not provide you with files for creating packages

### PUBLIC PRIVATE INTERFACE - The Magic Words of transitivity

I've teased it a bit and now it's time to talk about this. It's really not
particularly complicated but it ties up some of the stuff we mentioned before.

Way back, when I mentioned targets, I had some example snippets like so:

```
add_executable(MyApp ${MYAPP_SOURCES})

target_include_directories(MyApp <SomePath>/include)
target_link_libraries(MyApp someDep)
target_compile_definitions(MyApp "USE_DEP")

```

Well, the thing is, those are not 100% correct. Let's change this up a bit, and
let's make _MyApp_ into _MyLib_ so that some of this can make better sense:

```
add_library(MyLib ${MYLIB_SOURCES})

target_include_directories(MyLib PRIVATE <SomePath>/include)
target_link_libraries(MyLib PUBLIC someDep)
target_compile_definitions(MyLib INTERFACE "USE_DEP")

```

Now, what these keywords control is the **transitivity** of these properties.
And the way this comes into play is when we re-introduce My app

```
add_executable(MyApp ${MYAPP_SOURCES})

target_link_libraries(MyApp MyLib)

```

Now, when that `target_link_libraries()` happens, if _MyLib_ is a CMake
**target**, either native or imported (say through a *find_package(CONFIG)*),
CMake is going to make some magic happen.

For us right here, it'll make sure that MyApp is built with `-DUSE_DEP` and also
that it links in _someDep_. Properties on CMake targets live in two spaces, so to
speak. There's the `PRIVATE` space which applies to your target when it is being
built and the `INTERFACE` space which applies to projects that use it, say for
MyApp when it "links in" MyLib.

When something is declared `PUBLIC` it means it applies to both spaces. The target
needs it and targets linking it in also need it.

This mechanism drives a lot of the power behind how config-packages work, and
you can set them for custom or imported targets as well. Once you got that set up
smoothly, you can have other dependencies, your include paths, compile defines,
compile options (/BigObj, -mavx2), language standard requirements ( must have
cxx_constexpr)... all forwarded to targets that depend on your project, all set
with just a `target_link_libraries()` call.
