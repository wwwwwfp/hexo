---
title: java 员工轮询值班排班 开发设计（mysql+redis）
index_img: /img/cover/10.jpg
categories:
  - Mysql
tags:
  - redis
  - Mysql
  - 轮询
abbrlink: 2e9888de
date: 2018-12-19 20:01:00
---

+ **设计一个值班历史纪录表 duty_employee_history(area_id,dept_id) 联合主键，存放已值班过的数据**
    ```sql
    area_id  int(11) NOT NULL   区域 
     
    dep_id  int(11) NOT NULL    部门
     
    employee_ids   varchar(2000) NOT NULL  已值班过的 ,号分隔 格式 ,12,22,3,45, 前后要有逗号 防止 45 匹配456 这样的数据
    ```
+ **值班表 duty**

  从值班表中拿到当前部门当前区域 正在值班的employee   List<Integer>   dutyEmpIdList   （可以存放到Redis）

+ **从duty_history 中拿出  当前排班部门区域的唯一一条数据DutyEmployeeHistory 、 一个部门区域只有一条数据，可以不担心效率问题**
    ```sql
    DutyEmployeeHistory dutyEmployeeHistory = dutyEmployeeHistoryDao.selectByPrimaryKey(key);
    //如果为null 插入一条新的  第一个值班人员从值班人员list中随便拿一个
    if(null == dutyEmployeeHistory){
         dutyEmployeeHistory = new DutyEmployeeHistory();
         dutyEmployeeHistory.setOrgAreaId(areaId);
         dutyEmployeeHistory.setDepId(deptId);
         dutyEmployeeHistory.setFlowTypeId(flowTypeId);
         dutyEmployeeHistory.setEmployeeIds(","+dutyEmpIdList.get(0)+",");  //第一个值班人员
         dutyEmployeeHistoryDao.insertSelective(dutyEmployeeHistory);
         return dutyEmpIdList.get(0); //返回第一个值班人员
    }else {
         //遍历值班员工，如果在已经值班历史中没有，则将该员工追加到历史 employeeIds 后面
         //返回追加的该员工  即当前值班的员工
         String employeeIds = dutyEmployeeHistory.getEmployeeIds();
         for (Integer empId:dutyEmpIdList) {
             boolean contains = employeeIds.contains("," + empId + ",");
             if(!contains){
                 //如果已值班员工中不存在 当前员工，追加后面返回即可
                 employeeIds+=empId+",";
                 dutyEmployeeHistory.setEmployeeIds(employeeIds);
                 dutyEmployeeHistoryDao.updateByPrimaryKeySelective(dutyEmployeeHistory);
                 return empId;
             }
         }
         //该列表员工都已值过班,比较先后顺序  最后面的为最新的值班数据。
         Integer result = dutyEmpIdList.get(0);
         int index = employeeIds.indexOf("," + result + ",");
         int temp_index = 0;
         //判断  该list中的值班员工在 历史中的顺序，顺序越靠前的优先值班
         for (int i=0;i<dutyEmpIdList.size();i++) {
             temp_index = employeeIds.indexOf("," + dutyEmpIdList.get(i) + ",");
             if(temp_index<index){
                index = temp_index;
                result = dutyEmpIdList.get(i);
             }
         }
        //修改 duty_employee_history 表中的值班数据。
        employeeIds = employeeIds.replaceAll(","+result+",",",");
        employeeIds+=result+",";
        dutyEmployeeHistory.setEmployeeIds(employeeIds);
        dutyEmployeeHistoryDao.updateByPrimaryKeySelective(dutyEmployeeHistory);                   
        return result;   //该result为当前值班的员工
    }
    ```
+ 有好方案  还望骚扰   共同进步！！！！