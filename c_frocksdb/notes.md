1. 如何enable clockcache。
   一般情况下下载一个tbb就行
   sudo apt-get install libtbb-dev
  
2. 如何测试tbb有没有被安装
   bash build_tools/build_detect_platform tmp.txt
   看看tmp.txt里面PLATFORM_LDFLAGS和JAVA_LDFLAGS有没有加入 -ltbb
   因为这个build_detect_platform就是连接tbb的，而tbb是enable clockcache必须的库

3. 重新编译即可：env PORTABLE=1 make jclean rocksdbjava -j8
   这个时候在编译的时候会出现提示：#pragma message: TBB is defined and ROCKSDB_LITE is not defined. SUPPORT_CLOCK_CACHE is defined.表示ClockCache可以用了
   这个提示是我放在ClockCache.h里的
   然后把java/target/里的jar，搬到.m2即可
   cp rocksdbjni-6.20.3-linux64.jar ~/.m2/repository/com/ververica/frocksdbjni/6.20.3-ververica-1.0/frocksdbjni-6.20.3-ververica-1.0.jar

4. 一些小知识：c的projects可以通过makefile直接make编译，也可以通过CMakeLists.txt从cmake编译。如果是cmake的话：
   mkdir build
   cd build
   % cmake先生成makefile，再通过make编译
   cmake ./
   make
