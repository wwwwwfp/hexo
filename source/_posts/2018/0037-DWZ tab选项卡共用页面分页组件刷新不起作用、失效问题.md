---
title: DWZ tab选项卡共用页面分页组件刷新不起作用、失效问题
index_img: /img/cover/07.jpg
categories:
  - 前端
tags:
  - DWZ
abbrlink: 8f473d70
date: 2018-11-02 17:20:15
---
**初用DWX，还是碰到好多问题，谨以此文记录tab 分页刷新问题**

+ **HTML页面(tab页面):**
   ![](1.png)
   注：这儿id不能省略
+ **列表页面**
    ![](2.png)
   注：id为固定的 pagerForm    rel为 pagerForm
+ **page页面：**
   ```html
    <div th:include="views/common/pager_local :: tab"></div>
    ```
   
   pager_local.html
   ```html
   <th:block th:fragment="tab">
       <div class="panelBar"  th:online="text">
           <div class="pages">
               <span th:text="#{TEXT_DISPLAY}">display</span>
               <select class="combox" name="numPerPage" th:onchange="'javascript:tabPageBreak('+'{numPerPage:'+this.value+'}'+','+${tabTagId}+');'">
                   <option value="20"  th:selected="${infos.numPerPage}  == 20 ? 'selected' "  >20</option>
                   <option value="50"  th:selected="${infos.numPerPage}  == 50 ? 'selected' ">50</option>
                   <option value="100" th:selected="${infos.numPerPage}  == 100 ? 'selected' " >100</option>
                   <option value="200" th:selected="${infos.numPerPage}  == 200 ? 'selected' " >200</option>
               </select>
               <span><span th:text="#{TEXT_ITEMS}">items</span> <span>&nbsp;,&nbsp;</span><span th:text="#{TEXT_TOTAL}">total</span><span><label style="color:blue;">&nbsp;[[${infos.totalCount}]]&nbsp;</label></span><span th:text="#{TEXT_ITEMS}">items</span></span>
           </div>
           <div class="pagination"  targetType="navTab" th:rel="${tabTagId}" th:attr="totalCount=${infos.totalCount},numPerPage=${infos.numPerPage},currentPage=${infos.pageNum}"   pageNumShown="10" ></div>
       </div>
   </th:block>
   ```
   注：tabTagId 为 tab中的id值
   ```javascript
   function tabPageBreak(args, div){
       var tabpage = $(div).attr("id");
       dwzPageBreak({targetType:"navTab", rel:tabpage, data:args});
   }
   ```
   注：这儿的 div 获取到的是 id为 div 的整个标签
   