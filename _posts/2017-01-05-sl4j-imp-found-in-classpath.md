---
layout: post
title: sl4j神奇的log实现类自动发现
tags: xgame MicroService java classpath sl4j
category: xgame
---

# sl4j有点神奇

[http://www.slf4j.org](http://www.slf4j.org)

sl4j对于每个java开发者应该都不陌生，一个logger api，没有具体的实现，我们可以选择各种不同的实现，比如log4j/logback-classic/java.util.Logging/simple/no-operation，在runtime的时候，不用配置修改任何配置。

很神奇吧，不用配置比如指明哪具体使用那个具体的实现class，只需要把这个jar包，在运行java时添加到classpath，sl4j就会自动的发现并使用这个log实现。

# 自动发现的实现

多数同学对应程序的热情或许和我一样，*好奇！！！*


## SourceCode
	private final static void bind() {
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            // skip check under android, see also
            // http://jira.qos.ch/browse/SLF4J-328sl4j-api-pom
            if (!isAndroid()) {
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding
            StaticLoggerBinder.getSingleton();
            INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
            reportActualBinding(staticLoggerBinderPathSet);
            fixSubstituteLoggers();
            replayEvents();
            // release all resources in SUBST_FACTORY
            SUBST_FACTORY.clear();
        } catch (NoClassDefFoundError ncde) {
            String msg = ncde.getMessage();
            if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
                INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
                Util.report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
                Util.report("Defaulting to no-operation (NOP) logger implementation");
                Util.report("See " + NO_STATICLOGGERBINDER_URL + " for further details.");
            } else {
                failedBinding(ncde);
                throw ncde;
            }
        } catch (java.lang.NoSuchMethodError nsme) {
            String msg = nsme.getMessage();
            if (msg != null && msg.contains("org.slf4j.impl.StaticLoggerBinder.getSingleton()")) {
                INITIALIZATION_STATE = FAILED_INITIALIZATION;
                Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
                Util.report("Your binding is version 1.5.5 or earlier.");
                Util.report("Upgrade your binding to version 1.6.x.");
            }
            throw nsme;
        } catch (Exception e) {
            failedBinding(e);
            throw new IllegalStateException("Unexpected initialization failure", e);
        }
    }

    private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class";

    static Set<URL> findPossibleStaticLoggerBinderPathSet() {
        // use Set instead of list in order to deal with bug #138
        // LinkedHashSet appropriate here because it preserves insertion order
        // during iteration
        Set<URL> staticLoggerBinderPathSet = new LinkedHashSet<URL>();
        try {
            ClassLoader loggerFactoryClassLoader = LoggerFactory.class.getClassLoader();
            Enumeration<URL> paths;
            if (loggerFactoryClassLoader == null) {
                paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
            } else {
                paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
            }
            while (paths.hasMoreElements()) {
                URL path = paths.nextElement();
                staticLoggerBinderPathSet.add(path);
            }
        } catch (IOException ioe) {
            Util.report("Error getting resources from path", ioe);
        }
        return staticLoggerBinderPathSet;
    }

奥秘就在于：
1. findPossibleStaticLoggerBinderPathSet中通过查找classpath中StaticLoggerBinder类的列表，每个log实现都会生成org/slf4j/impl/StaticLoggerBinder.class文件，来确定有多少个sl4j实现报警。

2. StaticLoggerBinder中有static方法返回具体实现的类

## 另有玄机

但是有个很奇怪的问题，bind中我们用到了，StaticLoggerBinder.getSingleton()函数，如果在sl4j-api中没有这个实现，sl4j-api本身应该是不能编译通过的，如果sl4j-api中，本身有这个实现，在我们classpath中添加具体实现时，findPossibleStaticLoggerBinderPathSet必然会发现两个StaticLoggerBinder？

在添加logback-classic依赖的实测调试中，findPossibleStaticLoggerBinderPathSet这发现一个StaticLoggerBinder, TAT！！!

好吧，代码面前无秘密! [https://github.com/qos-ch/slf4j.git](https://github.com/qos-ch/slf4j.git)，上图：

图1. sl4j-api代码结构
![sl4j-api 源码结构]({{site.url}}/img/sl4j-api-imp-structure.png)

图2. sl4j-api pom
![l4j-api pom 去除imp]({{site.url}}/img/sl4j-api-pom.png)

1. 图1, sl4j-api代码中有一个StaticLoggerBinder的实现，确保编译OK
2. 图2, maven生成jar包的时候,删除了sl4j-api中org/slf4j/impl的所有class，所以发布的jar中是不包含StaticLoggerBinder。

sl4j-api中imp就是一个幌子，逗咱玩呢！！！


# 美丽的姑娘却曾困扰着我的青春

runtime添加到classpath，就自动配置了使用的某种logger实现，非常的方便，但是在项目具体的使用中也遇到了一下问题。

有一次，我遇到一个非常怪异的问题，应用部署到linux上面，正常的生成了日志，测试同学在win上面运行，没有生成日志。真是活见鬼！！！

原因：

为了免于手动修改java运行的classpath，我们的做法是所有依赖的第三方包放到app目录的lib文件夹下，服务器运行脚本，用sh/bat自动扫描lib下面全部的jar，添加到classpath中。sl4j是一个非常流行并被广泛采用的logger lib，我们依赖的其他第三方库也依赖了sl4j，并依赖了slfj-simple，所以在lib目录下面其实是有两个可用的sl4j实现，由于os的不用，和不断的加入第三方依赖，间接依赖的sl4j-simple和项目设定的logback实现添加到classpath中的顺序是随机的。如果simple在前，就使用了sl4j-simple,所以就没有了日志文件输出。

解法：

删除lib目录下的sl4j-simple jar。

	“Class path contains multiple SLF4J bindings.”

发现这个bug不难，方法就是仔细看应用启动日志就行，*日志很重要，认真看！！！*

延伸：

虽然手动删除了lib下面的sl4j-simple.jar，但是我们再次自动编译的时候，他又出现了！！！

由于非常多的第三方库使用了sl4j，我们有发现很多个版本的sl4j和他的具体实现出现在lib目录，太招人爱也是烦！！！

必须从依赖中排查sl4j，对必须！！！ 方法如下

1. Maven

[http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html](http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html)

	<dependency>
      <groupId>sample.ProjectB</groupId>
      <artifactId>Project-B</artifactId>
      <version>1.0-SNAPSHOT</version>
      <exclusions>
        <exclusion>
          <groupId>sample.ProjectE</groupId> <!-- Exclude Project-E from Project-B -->
          <artifactId>Project-E</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

2. Sbt（scala项目中用，我一直写scala）

[http://www.scala-sbt.org/0.13/docs/Library-Management.html](http://www.scala-sbt.org/0.13/docs/Library-Management.html)

	libraryDependencies += "log4j" % "log4j" % "1.2.15" exclude("javax.jms", "jms")

愉快的结束吧！
