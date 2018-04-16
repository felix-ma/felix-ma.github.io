---
layout:		post
title:		"测试"
subtitle:	"个人测试博客发布"
date:		2018-04-16
author:		"felix"
header-img:	"img/in-post/post-bg-universe.jpg"
catalog:	true
tags:
	- SpringBoot
---

# 控制台输出启动地址

```java
public static void main(String[] args) throws UnknownHostException {
    SpringApplication app = new SpringApplication(XXXXXX.class);
    Environment env = app.run(args).getEnvironment();
    log(env);
}

/**
 * 格式化运行成功后输出项目地址
 *
 * @param env
 */
private static void log(Environment env) {
    String name = env.getProperty("spring.application.name");
    String port = env.getProperty("server.port");
    String path = env.getProperty("server.context-path");
    String address = null;
    try {
        address = InetAddress.getLocalHost().getHostAddress();
    } catch (UnknownHostException e) {
        address = "127.0.0.1";
        e.printStackTrace();
    }
    name = StringUtils.isEmpty(name) ? "" : name;
    port = StringUtils.isEmpty(port) ? "8080" : port;
    path = StringUtils.isEmpty(path) ? "" : path;
    log.info(
            "\n----------------------------------------------------------\n\t"
                    + "Application '{}' is running! Access URLs:\n\t"
                    + "Local: \t\thttp://localhost:{}{}\n\t"
                    + "External: \thttp://{}:{}{}"
                    +"\n----------------------------------------------------------",
            name, port, path, address, port, path);
}

```