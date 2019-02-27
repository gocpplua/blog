1、下载protoc-xxx-win32.zip:[protoc-XXX-win32.zip
](https://github.com/protocolbuffers/protobuf/tags) 和 [protobuf-cpp-XXX.zip](https://github.com/protocolbuffers/protobuf/tags)

> 目前使用的是:v3.6.1。protobuf-3.0.0-alpha-3之后的版本均不提供vs的工程protobuf.sln。我下载了[2.6.1版本](https://github.com/protocolbuffers/protobuf/releases/tag/v2.6.1)
> 
2、配置环境变量
将解压出来的protoc.exe放在一全英文路径下，并把其路径名放在windows环境变量下的path下

3、在.proto文件中定义消息格式
syntax = "proto3";
package tutorial;

message Student{
    uint64 id = 1;
    string name =2;
}

4、生成代码
在所使用的proto文件路径下打开cmd窗口执行以下命令(protoc的详细用法参见protoc -h)：
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/XXXX.proto，
例如：protoc student.proto --cpp_out=./，会生成下述文件：
student.pb.h：声明你生成的类的头文件。
student.pb.cc：你生成的类的实现文件。



10、扩展protobuf
如果希望向后兼容，必须遵循：
a、不必更改tag数
b、不必添加或删除任何required字段
c、可以删除optional或repeated字段
d、可以添加新的optional或repeated字段，但你必须使用新的tag数。

 
11、优化
c++的protobuf库，已经极大地优化了。合理使用可以改善性能。
a、如果可能，复用message对象。
b、关于多线程的内存分配器

参考文章:
[《windows下使用vs进行Proctocol Buffer开发》](https://www.cnblogs.com/ppzbty/p/5412014.html)

[《google protobuf学习笔记一：windows下环境配置》](https://blog.csdn.net/majianfei1023/article/details/45371743)

[《google protobuf学习笔记二：使用和原理》](https://blog.csdn.net/majianfei1023/article/details/45112415)

[《Protocol Buffers C++入门教程》](https://blog.csdn.net/K346K346/article/details/51754431) - 读写参考这个

[《protobuf详细的安装和使用（windows cpp）》  ](https://blog.csdn.net/program_anywhere/article/details/77365876) --有cmake，写的不错

[《ProtoBuf如何在c++中使用》](https://blog.csdn.net/qq_15267341/article/details/80107293) -- 生成protobuf库以后，新建工程使用库

[cmake下载](https://github.com/Kitware/CMake/releases/download/v3.13.4/cmake-3.13.4-win64-x64.msi)