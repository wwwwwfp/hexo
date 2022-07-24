---
title: Salesforce Apex 触发器学习记录
index_img: /img/cover/22.jpg
categories:
  - Salesforce
tags:
  - Salesforce
  - Apex
  - Triggers
abbrlink: de06021c
date: 2020-07-01 20:06:54
---

> Apex 触发器（Apex Triggers）是一种特殊的 Apex 类。它的主要作用是在一条记录被插入、修改、删除之前或之后自动执行一系列的操作。每一个 Trigger 类必须对应一种对象。

1、直接在Trigger中写对应的逻辑。

2、通过Trigger 绑定 TriggerHandler ，在TriggerHandler中写对应的业务逻辑

注：在处理Trigger中的业务逻辑时，一定要考虑执行顺序和DML操作

参考：https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm?search_text=Trigger

+ Trigger 触发事件：
   ```text
   before insert：插入数据之前
   before update：更新数据之前
   before delete：删除数据之前
   after insert：插入数据之后
   after update：更新数据之后
   after delete：删除数据之后
   after undelete：恢复数据之后
   ```
+ Trigger 常用执行事件：
   ```text
   isInsert：是否是 insert 操作
   isUpdate：是否是 update 操作
   isDelete：是否是 delete 操作
   isBefore：是否是操作之前
   isAfter：是否是操作之后
   ```

**Trigger**

代码目录：在triggers文件夹下
![](1.png)

语法结构：
```java
/**
 * AccountTriggers  类名，自定义
 * Account 为Account 对象创建trigger  on后面为对象名称
 * 参数：事件类型，如删除前action   插入前action 等
 *      如果业务写了对应action  ，在参数中没有对应参数   ,后面业务逻辑也不会执行
 */
trigger AccountTriggers on Account(before delete, before insert, before update, 
                                    after delete, after insert, after update) {
	if (Trigger.isBefore) {
		// 事件前 处理的逻辑
		if (Trigger.isDelete) {
			//删除Account前逻辑
		}
		if(Trigger.isUpdate){
			// 更新前处理的逻辑
			// Trigger.new 可以获取到处理操作后的List<Account>
			List<Account> list_new_account = Trigger.new;
			// 获取修改之前的数据集
			List<Acclout> list_old_account = Trigger.old;
			// 返回对应原来数据的Map
			Map<Id,Contact> map_new_account = Trigger.newMap;
			// 返回对应新数据的Map
			Map<Id,Contact> map_old_account = Trigger.oldMap;
			
		}
		// ... 
	}else{
		// 事件后处理的逻辑
		// ... 事件处理同上
	}                                    
}
```

**TriggerHandler**

业务代码目录：在classe文件夹下，实质是一个Apex类

代码结构（Handler Class）

```java
/**
 * Triggers 源码
 * 本质是通过bind方法将对应的action和hanler 绑定对应的事件类型
 * 然后通过manage 方法执行对应的业务逻辑
 */
public class Triggers {
    /**
     * Enum representing each of before/after CRUD events on Sobjects
    */
    public enum Evt {
        afterdelete, afterinsert, afterundelete,
        afterupdate, beforedelete, beforeinsert, beforeupdate   
    }
    /**
     *  Simplistic handler to implement on any of the event. It doesn't requires or enforces any patter except the
     *  method name to be "handle()", a developer is free to use any Trigger context variable or reuse any other
     *  apex class here.
    */
    public interface Handler {
        void handle();          
    } 
    // Internal mapping of handlers
    Map<String, List<Handler>> eventHandlerMapping = new Map<String, List<Handler>>();
    /**
     *  Core API to bind handlers with events
    */
    public Triggers bind(Evt event, Handler eh) {
        List<Handler> handlers = eventHandlerMapping.get(event.name());
        if (handlers == null) {
            handlers = new List<Handler>();
            eventHandlerMapping.put(event.name(), handlers);
        }
        handlers.add(eh);
        return this;
    }
    /**
     * Invokes correct handlers as per the context of trigger and available registered handlers
    */
    public void manage() {
        Evt ev = null;
        if(Trigger.isInsert && Trigger.isBefore){
            ev = Evt.beforeinsert;
        } else if(Trigger.isInsert && Trigger.isAfter){
            ev = Evt.afterinsert;
        } else if(Trigger.isUpdate && Trigger.isBefore){
            ev = Evt.beforeupdate;
        } else if(Trigger.isUpdate && Trigger.isAfter){
            ev = Evt.afterupdate;
        } else if(Trigger.isDelete && Trigger.isBefore){
            ev = Evt.beforedelete;
        } else if(Trigger.isDelete && Trigger.isAfter){
            ev = Evt.afterdelete;
        } else if(Trigger.isundelete){
            ev = Evt.afterundelete;             
        }
        List<Handler> handlers = eventHandlerMapping.get(ev.name());
        if (handlers != null && !handlers.isEmpty()) {
            for (Handler h : handlers) {
                h.handle();
            }
        }
    } 
}


/**
 * 必须要实现 Triggers.Handler
 * 重新 handler 方法
 */
public with sharing class AccountHandler implements Triggers.Handler {
	/**
	 * 执行业务逻辑
	 */
	public static void handle(){
		//  业务逻辑
	    System.debug('....');
	    
	    if(Trigger.isInsert){
	    	// xxxxx
	    }
	    
	    if(Trigger.isUpdate){
	    	// xxxxx
	    }
	}
}

/**
 * 在 Trigger 中绑定对应的handler 并执行对应得业务逻辑
 */
 trigger AccoutTrigger on Account(before insert, before update, after insert) {
    new Triggers()
        .bind(Triggers.Evt.afterinsert,new AccountHandler())
        .bind(Triggers.Evt.beforeupdate,new AccountHandler())
        .manage();
}
```
