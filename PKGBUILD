# Maintainer: Jens Staal <staal1978@gmail.com>

pkgname=llvmlinux-toolchain-svn-git
pkgver=20120714
pkgrel=2
pkgdesc="a bundle of LLVM, compiler-rt, lldb and clang patched to compile Linux"
url="http://llvm.linuxfoundation.org/index.php/Main_Page"
arch=('i686' 'x86_64')
license=('UIUC' 'MIT')
depends=('ocaml' 'swig' 'libedit')
makedepends=('subversion' 'git' 'cmake' 'python2' 'libffi' 'clang')
optdepends=('libc++-svn') #optional initial make dependency when it starts working...
provides=('llvm' 'compiler-rt' 'clang' 'llvm-ocaml' 'clang-analyzer' 'lldb' 'libc++-svn' 'libc++' 'libc++abi')
replaces=('llvm' 'clang' 'llvm-ocaml' 'clang-analyzer' 'libc++-svn')
conflicts=('llvm' 'clang' 'llvm-ocaml' 'clang-analyzer' 'libc++-svn')

#this checkout contain LLVMlinux-specific patches
_llvm_gitroot=('http://git.linuxfoundation.org/llvm-setup.git')
_llvm_gitname=('llvm-setup')

if [ ${CARCH} == x86_64 ]; then
    _target=$CARCH
else
    _target=x86
fi

build(){
    cd ${srcdir}
    
    msg "checking out LLVM-specific linux patches and config(s)"
    if [ -d $_llvm_gitname ] ; then
	cd $_llvm_gitname && git pull $_llvm_gitroot
	msg2 "The local files are updated."
    else
	git clone $_llvm_gitroot $_llvm_gitname
    fi
	msg2 "done"
	  
    cd ${srcdir}
    
    msg "setting up llvm source code tree"
    if [ -d llvm-svn/.svn ]; then
	(cd llvm-svn && svn up)
    else
      svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm-svn
    fi
    msg2 "done"
    
    msg "setting up clang source code tree"
    cd ${srcdir}/llvm-svn/tools
    if [ -d clang/.svn ]; then
	(cd clang && svn up)
    else
      svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
    fi
    msg2 "done"
    
    msg "setting up lldb source code tree"
    cd ${srcdir}/llvm-svn/tools
    if [ -d lldb/.svn ]; then
	(cd lldb && svn up)
    else
      svn co http://llvm.org/svn/llvm-project/lldb/trunk lldb
    fi
    msg2 "done"
        
    cd $srcdir/llvm-svn/projects
    msg "setting up compiler-rt sources"
    if [ -d compiler-rt/.svn ]; then
        (cd compiler-rt && svn up)
    else
        svn co http://llvm.org/svn/llvm-project/compiler-rt/trunk compiler-rt
    fi
    
    msg "setting up libc++abi sources"
    if [ -d libcxxabi/.svn ]; then
	(cd libcxxabi && svn up)
    else
      svn co http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi
    fi

    msg "setting up libc++ sources"
    if [ -d libcxx/.svn ]; then
	(cd libcxx && svn up)
    else
      svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx
    fi
    

    msg "SVN checkout done or server timeout"
    
    msg "setting up build directory sources and patching"
    rm -rf ${srcdir}/llvm #start fresh
    cp -ar ${srcdir}/llvm-svn ${srcdir}/llvm
    cd ${srcdir}/llvm
    msg2 "Arch package-derived sed -driven modifications"
    # Fix installation directories, ./configure doesn't seem to set them right
    sed -i -e 's:\$(PROJ_prefix)/etc/llvm:/etc/llvm:' \
         -e 's:\$(PROJ_prefix)/lib:$(PROJ_prefix)/lib/llvm:' \
         -e 's:\$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in
    sed -i '/ActiveLibDir = ActivePrefix/s:lib:lib/llvm:' \
    tools/llvm-config/llvm-config.cpp
    sed -i 's:LLVM_LIBDIR="${prefix}/lib":LLVM_LIBDIR="${prefix}/lib/llvm":' \
    autoconf/configure.ac \
    configure

    # Fix insecure rpath (http://bugs.archlinux.org/task/14017)
    sed -i 's:$(RPATH) -Wl,$(\(ToolDir\|LibDir\|ExmplDir\))::g' Makefile.rules
    
    _patchdir="${srcdir}/llvm-setup/toolchain/clang/patches"
    if [ -d ${_patchdir}/clang/x86 ] ; then
	  msg2 "patching clang..."
	  cd ${srcdir}/llvm/tools/clang
	  for i in ${_patchdir}/clang/x86/*.patch; do
	      msg2 "applying ${i}"
	      patch -p1 < ${i}
	  done
    else
	  msg2 "no clang patches..."
    fi
    if [ -d ${_patchdir}/llvm/x86 ] ; then
	msg2 "patching llvm..."
	cd ${srcdir}/llvm 
	for i in ${_patchdir}/llvm/x86/*.patch; do
	    msg2 "applying ${i}"
	    patch -p1 < ${i}
	done
    else
	msg2 "no llvm patches..."
    fi
    msg2 "done"
    
    msg "setting up clang as default compiler"
#    export CC=clang
#    export CXX=clang++ 
    export CFLAGS="-march=native -mtune=native -O2 -pipe"
    export CXXFLAGS="-march=native -mtune=native -O2 -pipe" #fortify option can break clang-self-compilation...
#    export AR=llvm-ar
#    export AS=llvm-as
#    export LD=llvm-link
#    export NM=llvm-nm
#    export OBJDUMP=llvm-objdump
#    export RANLIB=llvm-ranlib
    
# getting it to build with libc++ is still a work in progress...
#    CXXFLAGS="$CXXFLAGS -stdlib=libc++ -I/usr/include/c++/v1 -I$srcdir/llvm/projects/libcxxabi/include"
#    LDFLAGS="$LDFLAGS -Wl,-lstdc++" 
# official upstream build explicitly uses GCC...
    export CC=gcc
    export CXX=g++

# Include location of libffi headers in CPPFLAGS
    export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"

# remove lldb from build as long as compiler isn't clang.
    if [ ${CXX} == g++ ]; then
	 msg "lldb requires clang to build"
	 rm -rf ${srcdir}/llvm/tools/lldb
    else
	 msg "OK compiler is Clang, building lldb"
    fi
    
# remove "sample" from projects... it does not do anything useful
    msg "removing 'sample' project"
    rm -rf ${srcdir}/llvm/projects/sample
    

    # Use Python 2
    rm -rf  "$srcdir/python2-path" # start fresh
    mkdir -p "$srcdir/python2-path"
    ln -s /usr/bin/python2 "$srcdir/python2-path/python"
    export PATH="$srcdir/python2-path:$PATH"
          
    msg "Starting make..."
    
# several undocumented configure options that were used by someone else to build LLVM/Clang with clang and c++...
# those that fail are commented out again.
#build only enables host target.
    cd ${srcdir}/llvm
    ./configure \
    --prefix=/usr \
    --libdir=/usr/lib/llvm \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-pic \
    --enable-polly \
    --enable-jit \
    --enable-libffi \
    --disable-libcpp \
    --enable-targets=host \
    --with-cxx-include-arch=$_target \
    --enable-ltdl-install \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --enable-optimized \
    --enable-threads \
    --disable-multilib \
    --with-optimize-option=-O2 \
    --with-binutils-include=/usr/include

#    --with-clang \
#    --with-built-clang \
#    --disable-bootstrap \
#    --no-recursion \
#    --no-create \
#    --with-c-include-dirs=
# due to linking errors with libc++, I have currently explicitly disabled it.
# to test compiling with LLVM libc++. change --disable-libcpp to --enable-libcpp.

#fixing packaging problems with libc++ makefile
#essentially the same functions can be performed by toolchain.install
 sed -i 's/chown -R root:wheel $(HEADER_DIR)/#placeholder: chown -R broke packaging/b' $srcdir/llvm/projects/libcxx/Makefile

 
    make -j1 REQUIRES_RTTI=1 || return 1
    make DESTDIR=${pkgdir} install || return 1 
    #getting some ld paths OK
    mkdir -p $pkgdir/etc/ld.so.conf.d
    echo "/usr/lib/llvm" > $pkgdir/etc/ld.so.conf.d/llvm.conf
    echo "/usr/lib/ocaml" > $pkgdir/etc/ld.so.conf.d/llvm-ocaml.conf
} 
