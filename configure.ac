AC_INIT([Haskell LLVM data interoperability], [0.1.0], [travitch@cs.wisc.edu], [llvm-data-interop])

AC_LANG(C++)

AC_CONFIG_SRCDIR([src/LLVM/Internal/Interop.chs])

AC_CONFIG_FILES([llvm-data-interop.buildinfo])

AC_PROG_CXX

AC_ARG_WITH(compiler,
  [AS_HELP_STRING([--with-compiler],
    [use the given Haskell compiler])],
  compiler="$withval",
  compiler=ghc)dnl

AC_ARG_WITH(llvm_prefix,
  [AS_HELP_STRING([--with-llvm-prefix],
    [use the version of LLVM at the given location])],
  llvm_prefix="$withval",
  llvm_prefix="$prefix")dnl

AC_ARG_WITH(llvm_bindir,
  [AS_HELP_STRING([--with-llvm-bindir],
    [use LLVM binaries at the given location])],
  llvm_bindir="$withval",
  llvm_bindir="$llvm_prefix/bin")dnl

AC_PATH_PROGS(llvm_config, [llvm-config-3.2 llvm-config-3.1 llvm-config],
  [AC_MSG_ERROR(could not find llvm-config in $llvm_bindir)],
  ["$llvm_bindir:$PATH"])

dnl * Choose target platform
dnl
dnl We don't use the standard autoconf macros for this, but instead
dnl ask GHC what platform it is for.  Why?  We need to generate a library
dnl matching the compiler.
dnl NB: This code is from GHC's configure (where the corresponding code for
dnl guessing host and build variables can be found, too)

dnl Guess target platform if necessary.
m4_divert_once([HELP_CANON],
[[
System types:
  --target=TARGET   configure for building compilers for TARGET [guessed]]])dnl

if test "$target" = ""
then
    if test "${compiler}" != ""
    then
        target=`${compiler} +RTS --info | grep '^ ,("Target platform"' | sed -e 's/.*, "//' -e 's/")//' | tr -d '\r'`
        echo "Target platform inferred as: $target"
    else
        echo "Can't work out target platform"
        exit 1
    fi
fi

dnl Determine target-specific options
dnl This is important as Snow Leopard (Mac OS X 10.6) defaults to generating
dnl 64-bit code.
case $target in
i386-apple-darwin)
    TARGET_CPPFLAGS="-m32"
    TAGRET_LDFLAGS="-m32"
    ;;
x86_64-apple-darwin)
    TARGET_CPPFLAGS="-m64"
    TAGRET_LDFLAGS="-m64"
    ;;
esac

llvm_version="`$llvm_config --version`"
llvm_cppflags="`$llvm_config --cppflags`"
llvm_includedir="`$llvm_config --includedir`"
llvm_ldflags="`$llvm_config --ldflags`"

llvm_all_libs="`$llvm_config --libs all`"
llvm_target="`$llvm_config --libs engine | sed 's/.*LLVM\(.[[^ ]]*\)CodeGen.*/\1/'`"

dnl We need to separate libraries that need to be linked from other linker options.
llvm_extra_libs=""
llvm_extra_libdirs=""
llvm_ldoptions=""
for opt in $llvm_all_libs $llvm_ldflags; do
  case $opt in
    -l*) llvm_extra_libs="$llvm_extra_libs `echo $opt | sed 's/-l//'`";;
    -L*) llvm_extra_libdirs="$llvm_extra_libdirs `echo $opt | sed 's/-L//'`";;
    *)   llvm_ldoptions="$llvm_ldoptions $opt";;
  esac
done

llvm_libdir="`$llvm_config --libdir`"

CPPFLAGS="$llvm_cppflags $CPPFLAGS $TARGET_CPPFLAGS"
LDFLAGS="$llvm_ldflags $LDFLAGS $TARGET_LDFLAGS"

AC_CHECK_HEADERS([llvm/Module.h], [],
  [AC_MSG_ERROR(could not find LLVM headers)])

# We have to link using the C++ compiler.
CC=$CXX

save_LIBS="$LIBS"

AC_LINK_IFELSE(
  [AC_LANG_PROGRAM([#include <llvm/Module.h>],
    [llvm::Module *dummy])],
  [LIBS="-lLLVM-${llvm_version} $LIBS"] [HAVE_LLVM=1],
  [AC_MSG_ERROR(["The llvm-${llvm_version} shared library is not installed"])])

AC_SUBST([llvm_version])
AC_SUBST([llvm_cppflags])
AC_SUBST([llvm_extra_libs])
AC_SUBST([llvm_extra_libdirs])
AC_SUBST([llvm_target])
AC_SUBST([llvm_includedir])
AC_SUBST([llvm_ldoptions])
AC_SUBST([llvm_libdir])

AC_OUTPUT
