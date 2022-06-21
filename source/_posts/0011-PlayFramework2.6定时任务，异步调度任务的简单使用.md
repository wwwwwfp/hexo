---
title: play framework 2.6 定时任务，异步调度任务的简单使用
index_img: /img/cover/11.jpg
categories:
  - Play Framework
tags:
  - Play Framework
abbrlink: ce0341e0
date: 2018-03-09 18:17:57
---
#### 1. 创建并启用模块
```java
public class ZTasksModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(CompositeImageTask.class).asEagerSingleton();//绑定CompositeImageTask任务
    }
}
```
#### 2. 然后在application.conf中通过添加以下行来启用该模块
    play.modules.enabled += "tasks.ZTasksModule"

#### 3. 任务demo  CompositeImageTask.java
```java
public class CompositeImageTask {
    private final ActorSystem actorSystem;
    private final ExecutionContext executionContext;

    @Inject
    public CompositeImageTask(ActorSystem actorSystem, ExecutionContext executionContext) {
        this.actorSystem = actorSystem;
        this.executionContext = executionContext;
        this.initialize();
    }

    private void initialize() {
        this.actorSystem.scheduler().schedule(
                Duration.create(1, TimeUnit.MINUTES), // initialDelay  项目启动后该任务多长时间执行
                Duration.create(10, TimeUnit.MINUTES), // interval     该任务执行周期
                () -> composite(),
                this.executionContext
        );
    }

    //项目启动后1分钟执行该任务，然后每隔10分钟执行一次
    public  void composite()  {
        /***任务逻辑***/
    }
}
```
