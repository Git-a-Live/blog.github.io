对海外开发者们而言，使用Android Studio几乎不会有什么大问题。然而对于中国大陆的开发者们来说， 得益于中国国家防火墙的存在，使用Android Studio需要进行一些额外的本不必要的配置，从而在某种程度上提高了入门的门槛。

配置Android Studio，主要就是配置Gradle文件，尤其是项目的build.gradle文件， 在这个文件当中有两处地方需要做修改，分别是`buildscript`和`allprojects`当中的：

```
repositories {
    google()
    jcenter()
}
```

这两个地方需要在`google( )`前加上

```
maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
maven { url 'https://maven.aliyun.com/repository/google' }
```

这三行代码表示的是Gradle在中国大陆的阿里云镜像仓库，加上之后，Gradle就可以直接从镜像仓库进行同步，从而提高同步速度。 

Gradle同步成功的标志，就是`activity_main.xml`文件加载完成，并出现一个带有“Hello World”字样文本框的UI界面， 这时就可以开始摆放控件设计UI，在`MainActivity`文件中编写源代码了。

注意：`google( )`和`jcenter( )`不一定要注释掉，因为有时候就算注释掉，同步Gradle也还是有可能发生无法连接之类的错误。 配置完之后再同步Gradle，通常可以解决Gradle同步失败的问题。