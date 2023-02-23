## 何为Flutter

[Flutter](https://flutter.dev/)是Google于2017年发布的一款开源框架，其设计目标为使用一套代码，为不同平台构建出好看且使用原生代码编译的应用。换句话说，Flutter就是为实现跨平台应用而打造的。

Flutter实现跨平台的技术方案，是类似Java那样采用自己的虚拟机——Dart虚拟机（跟Android早期的Dalvik虚拟机可不是同一个东西），并且提供一定的桥接技术，完成页面渲染和平台服务调用等功能。尤其值得注意的是，Flutter有自己的图形渲染引擎[Skia](https://github.com/google/skia)，这就是它的UI得以实现跨平台的重要基础。

Flutter采用的开发语言是[Dart](https://dart.dev/)，这是一门几乎完全为Flutter进行深度适配，并与其牢牢绑定的编程语言。

## Flutter开发环境搭建