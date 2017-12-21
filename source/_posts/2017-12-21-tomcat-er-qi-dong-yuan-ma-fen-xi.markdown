---
layout: post
title: "tomcat-二-启动源码分析"
date: 2017-12-21 14:48:08 +0800
comments: true
categories: 
---
#一.Bootstrap类

启动tomcat时实际是调用Boostrap的main方法，在启动tomcat时在main方法中做了三件事

>* 初始化classLoader
>* 加载Catalina类
>* 调用Catalina的start方法

Bootstrap中有个static块，用于设置catalina.home和catalina.base

在关闭tomcat时调用Catalina的stop方法

##1.先看看main方法
```java
public static void main(String args[]) {

        if (daemon == null) {
            // Don't set daemon until init() has completed
            Bootstrap bootstrap = new Bootstrap();
            try {
                //1.初始化classLoader 2.加载Catania类
                bootstrap.init();
            } catch (Throwable t) {
                handleThrowable(t);
                t.printStackTrace();
                return;
            }
            daemon = bootstrap;
        } else {
            // When running as a service the call to stop will be on a new
            // thread so make sure the correct class loader is used to prevent
            // a range of class not found exceptions.
            Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
        }

        try {
            String command = "start";
            if (args.length > 0) {
                command = args[args.length - 1];
            }

            if (command.equals("startd")) {
                args[args.length - 1] = "start";
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stopd")) {
                args[args.length - 1] = "stop";
                daemon.stop();
            } else if (command.equals("start")) {
                //3.调用Catania的start方法
                daemon.setAwait(true);
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stop")) {
                daemon.stopServer(args);
            } else if (command.equals("configtest")) {
                daemon.load(args);
                if (null==daemon.getServer()) {
                    System.exit(1);
                }
                System.exit(0);
            } else {
                log.warn("Bootstrap: command \"" + command + "\" does not exist.");
            }
        } catch (Throwable t) {
            // Unwrap the Exception for clearer error reporting
            if (t instanceof InvocationTargetException &&
                    t.getCause() != null) {
                t = t.getCause();
            }
            handleThrowable(t);
            t.printStackTrace();
            System.exit(1);
        }

    }
```

##2.看看bootstrap.init()方法

```java
public void init() throws Exception {

        //初始化类加载器
        initClassLoaders();

        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // 加载Catania类
        if (log.isDebugEnabled())
            log.debug("Loading startup class");
        Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.getConstructor().newInstance();

        // 设置shareLoader为Catania的parentClassLoader
        if (log.isDebugEnabled())
            log.debug("Setting startup class properties");
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;

    }
```

#二.Catalina类

在Bootstrap的main方法中会调用Catalina的load、start方法。主要是初始化和启动server，具体做了以下几个事：

>* initDirs() : 初始化 java.io.tmpdir 目录
>* initNaming() : 初始化Java命名服务
>* createStartDigester() : 读取并解析server.xml
>* initStreams() : 设置标准输出和错误输出
>* server.init() : 调用server的init()方法
>* getServer().start() : 开始server
>* Runtime.getRuntime().addShutdownHook(shutdownHook) ： 注册shutdownHook

##1.Catalina的start方法
```java
public void start() {

        if (getServer() == null) {
            load();
        }

        if (getServer() == null) {
            log.fatal("Cannot start server. Server instance is not configured.");
            return;
        }

        long t1 = System.nanoTime();

        // 开始server
        try {
            getServer().start();
        } catch (LifecycleException e) {
            log.fatal(sm.getString("catalina.serverStartFail"), e);
            try {
                getServer().destroy();
            } catch (LifecycleException e1) {
                log.debug("destroy() failed for failed Server ", e1);
            }
            return;
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info("Server startup in " + ((t2 - t1) / 1000000) + " ms");
        }

        // 注册shutdownHook
        if (useShutdownHook) {
            if (shutdownHook == null) {
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);

            // If JULI is being used, disable JULI's shutdown hook since
            // shutdown hooks run in parallel and log messages may be lost
            // if JULI's hook completes before the CatalinaShutdownHook()
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager) logManager).setUseShutdownHook(
                        false);
            }
        }

        if (await) {
            //调用server的await方法
            await();
            stop();
        }
    }
```

##2.Catalina的load方法
```java
/**
     * Start a new server instance.
     */
    public void load() {

        long t1 = System.nanoTime();

        //初始化 java.io.tmpdir 目录
        initDirs();

        // 初始化Java命名服务
        initNaming();

        // 读取并解析server.xml
        Digester digester = createStartDigester();

        InputSource inputSource = null;
        InputStream inputStream = null;
        File file = null;
            //省略部分代码

            try {
                inputSource.setByteStream(inputStream);
                digester.push(this);
                digester.parse(inputSource);
            } catch (SAXParseException spe) {
                log.warn("Catalina.start using " + getConfigFile() + ": " +
                        spe.getMessage());
                return;
            } catch (Exception e) {
                log.warn("Catalina.start using " + getConfigFile() + ": " , e);
                return;
            }
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    // Ignore
                }
            }
        }

        getServer().setCatalina(this);
        getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
        getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());

        // 设置标准输出和错误输出
        initStreams();

        // 调用server的init()方法
        try {
            getServer().init();
        } catch (LifecycleException e) {
            if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE")) {
                throw new java.lang.Error(e);
            } else {
                log.error("Catalina.start", e);
            }
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info("Initialization processed in " + ((t2 - t1) / 1000000) + " ms");
        }
    }

```




