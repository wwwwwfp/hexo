---
title: Salesforce 进程生成器Process Builder Apex操作类型
index_img: /img/cover/21.jpg
categories:
  - Salesforce
tags:
  - Salesforce
  - Apex
  - 进程生成器
abbrlink: 97812e5a
date: 2020-06-30 20:57:08
---

> 进程生成器的操作类型有好多，该文只记录Apex类型

![](1.png)

1. 创建Apex类
    ```java
    /**
     * 注：进程执行器的类 必须要有@InvocableMethod 注解
     * 且 仅有一个
     */
    public  class PurchaseOrderProcessBuilder {
        
        /**
         * label 进程执行器名称，在平台选择的时候显示的名称
         * description 描述信息
         * 参数：自定义 可以到平台通过不同类型进行设置，如：字段引用/全局常量/公式/ID 等 进行设置
         */
        @InvocableMethod(label='update Approval condition' description='update Approval condition') 
        public static void test(List<Id> ids){
            System.debug('=============================================');
            System.debug('ids: ' + ids);
            // 操作逻辑
        }
    }
    ```
2. 在系统平台添加相关配置
   1. 入口
      ![](2.png)
   2. 新建一个Process Builder
   
      配置完对应逻辑和参数等之后启用即可
      ![](3.png)
        
3. 测试结果
   ![](4.png)