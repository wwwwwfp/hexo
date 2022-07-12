---
title: Salesforce Apex 变量、集合基础知识
index_img: /img/cover/21.jpg
categories:
  - Salesforce
tags:
  - Apex
  - Salesforce
abbrlink: bbca4297
date: 2019-05-28 23:53:36
---
  
### Salesforce

salesforce 大概分成两个部分：Apex,VisualForce Page

其中Apex 语法和java 语法很相似其中：

常用的基本变量有：种常用的基本变量Integer,String,Decimal,Double,Long,Boolean,ID。

常用的集合对象：List,Set,Map

其他：Object 和 sObject

其中sObject 定义为：Salesforce中的标准对象或自定义对象在Apex中使用时被称作“sObject”。sObject对象的一个实例相当于Salesforce中的一条记录。

### 一、基本变量

+ Integer
    ```java
    // 主要方法：
    public String format() // Integer转String
    public static Integer valueOf(String stringToObject) //String转Integer
    ```
+ Long
    ```java
    // 主要方法：
    public String format()	//转String
    public Integer intValue()  //转Integer
    public static Long valueOf(String stringToLong)  //String 转Long
    ```
+ Decimal
    ```java
    // 主要方法：
    public Decimal abs()  //返回小数点的绝对值
    public Decimal divide(Decimal divisor, Integer scale)  // 除数为divisor 返回结果为scale位数
    public Double doubleValue() // 转double
    public String format() //  转string
    public Integer intValue()  //转Integr
    public Long longValue()
    public Integer precision()  //返回 值对应的数字个数  如 3.2   返回为2   即3和2
    public Long round() //转换Long 四舍五入
    public Integer scale() //返回小数点后的位数个数
    public Decimal setScale(Integer scale) //设置小数点后的位数个数
    public Decimal stripTrailingZeros() // 去除0以后的小数
    Decimal.valueOf(Object objectToDecimal)// 将Double  Long String 等转换成Decimal
    ```
+ Boolean

    需注意的一点 默认为null
+ Double
    ```java
    // 主要方法：
    public static Double valueOf(String stringToDouble)
    public Long round() // 四舍五入返回Long
    public Integer intValue() //转 Integer
    ```
+ ID

  ID类型可以用任何一个符合规则的18位字符表示，如果你设置ID字符为15位，则将字符自动扩展成18位。不符合规则的ID字符在运行时则运行时异常
    ```java
    主要方法：
    public static ID valueOf(String toID)   //将toId转换成ID
    public Boolean equals(String id)   //判断两个ID是否相同
    ```
+ String
    ```java
    // 主要方法：
    //返回简化字符串，maxWidth>自身长度？自身：maxWidth长度的字符串加上省略号,省略号占3个字符
    public String abbreviate(Integer maxWidth)  
    //注意：maxWidth如果小于4，则抛出Runtime Exception
    public String abbreviate(Integer maxWidth,Integer offset) //返回简化字符串，maxWidth为需要简化长度，offset为字符串简化起点
    //如果max太小，则抛出Runtime Exception
    public String capitalize()     //返回当前字符串，其中第一个字母改为标题（大写）。
    public String center(Integer size)  //返回指定大小字符串使原字符串位于中间位置（空格填充左右），如果size<字符串长度，则不起作用
    public String center(Integer size,String paddingString)  //返回指定大小字符串使原字符串处于中间位置。参数1：字符串显示长度；参数2：填充的字符样式
    public Integer charAt(Integer index)   //返回对应值得ASC码值
    public Integer codePointAt(Integer index)   //返回指定位置的值对应的Unicode编码
    public Integer codePointBefore(Integer index)   //返回指定位置的值前一个对应的Unicode编码
    public Integer codePointCount(Integer beginIndex,Integer endIndex)  //返回起始位置到截至位置字符串的Unicode编码值
    public Integer compareTo(String secondString)  //基于Unicode比较两个字符串大小,如果小于比较值返回负整数，大于返回正整数，等于返回0
    public Boolean contains(String substring) //判断是否包含某个字符串，包含返回true，不包含返回false    
    public Boolean containsAny(String inputString)   //判断是否包含inputString任意一个字符，包含返回true，不包含返回false
    public Boolean containsIgnoreCase(String inputString)    //判断是否包含inputString（不区分大小写）,包含返回true，不包含返回false
    public Boolean containsNone(String inputString)   //判断是否不包含inputString，不包含返回true，包含返回false
    public Boolean containsOnly(String inputString)  //当前字符串从指定序列只包括inputString返回true，否则返回false
    public Boolean containsWhitespace()  //判断字符串是否包含空格，包含返回true，不包含返回false
    public Integer countMatches(String substring)  //判断子字符串在字符串中出现的次数
    public String deleteWhitespace()  移除字符串中所有的空格  //返回两个字符串之间不同，如果anotherString为空字符串，则返回空字符串，如果anotherString为null，则抛异常     比较结果以anotherString为基准，从第一个字符比较，不相同则返回anotherString与源字符串不同之处
    public Boolean endsWith(String substring)  //判断字符串是否已substring截止，如果是返回true，否则返回false
    public Boolean endsWithIgnoreCase(String substring)  //判断字符串是否已substring截止（不区分大小写），如果是返回true，否则返回false
    public Boolean equals(Object anotherString)  //判断字符串是否和其他字符串相同
    public Boolean equalsIgnoreCase(String anotherString)  //判断字符串是否和其他字符串相同（不区分大小写）
    public static String format(String stringToFormat, List<String> formattingArguments)  //将转换的参数替换成相同方式的字符串中
    public static String fromCharArray(List<Integer> charArray)   //将char类型数组转换成String类型
    public List<Integer> getChars()   //返回字符串的字符列表
    public static String getCommonPrefix(List<String> strings)  //获取列表共有前缀
    public Integer hashCode()   //返回字符串的哈希值
    public Integer indexOf(String substring)  //返回substring第一次在字符串中出现的位置，如果不存在返回-1
    public Integer indexOf(String substring,Integer index)  //返回substring第一次在字符串中出现的位置，起始查询字符串的位置为index处
    public Integer indexOfAny(String substring)  //substring任意一个字符第一次在字符串中出现的位置
    public Integer indexOfAnyBut(String substring)  //返回substring任意一个字符不被包含的第一个位置，无则返回-1
    public Integer indexOfChar(int char)  //返回字符串中char字符最先出现的位置
    public Integer indexOfDifference(String compareTo)  //返回两个字符串第一个不同位置的坐标
    public Integer indexOfIgnoreCase(String substring)  //返回substring在字符串中第一个出现的位置（不考虑大小写）
    public Boolean isAllLowerCase()  //字符串是否均为小写，如果是返回true，否则返回false
    public Boolean isAllUpperCase()  //字符串是否均为大写，如果是返回true，否则返回false
    public Boolean isAlpha()   //如果当前所有字符均为Unicode编码，则返回true，否则返回false
    public Boolean isAlphanumeric()  //如果当前所有字符均为Unicode编码或者Number类型编码，则返回true，否则返回false
    public Boolean isAlphanumericSpace()  //如果当前所有字符均为Unicode编码或者Number类型或者空格，则返回true，否则返回false
    public Boolean isAlphaSpace()  //如果当前所有字符均为Unicode或者空格，则返回true，否则返回false
    public Boolean isAsciiPrintable()  //如果当前所有字符均为可打印的Asc码，则返回true，否则返回false
    public Boolean isNumeric()  //如果当前字符串只包含Unicode的位数，则返回true，否则返回false
    public Boolean isWhitespace()  //如果当前字符只包括空字符或者空，则返回true，否则返回false
    public static String join(Object iterableObj, String separator)   //通过separator连接对象，通用于数组，列表等
    public Boolean lastIndexOf(String substring)   //substring在字符串中最后出现的位置，不存在则返回-1
    public Boolean lastIndexOfIgnoreCase(String substring)  //substring在字符串中最后出现的位置（忽略大小写），不存在则返回-1
    public String left(Integer index)    //获取从零开始到index处的字符串
    public String leftPad(Integer length)   //返回当前字符串填充的空间左边和指定的长度
    public Integer length()    //返回字符串长度
    public String mid(Integer startIndex,Integer length);  //返回新的字符串，第一个参数为起始位，第二个字符为字符的长度//类似于substring
    public String normalizeSpace()    //前后删除空白字符
    public String remove(String substring)   //移除所有特定的子字符串，并返回新字符串
    public String removeIgnorecase(String substring)   //移除所有特定的子字符串(忽略大小写)，并返回新字符串
    public String removeEnd(String substring)   //当且仅当子字符串在后面移除子字符串
    public String removeStatrt(String substring)   //当且仅当子字符串在后面移除子字符串
    public String repeat(Integer numberOfTimes)   //重复字符串numberOfTimes次数
    public String repeat(String separator, Integer numberOfTimes)   //重复字符串numberOfTimes次，通过separator作为分隔符
    public String replace(String target, String replacement)   //将字符串中的target转换成replacement
    public String replaceAll(String target,String replacement)
    public String reverse()  //字符串倒序排列
    public String right(Integer length)   //从右查询返回length的个数的字符串
    public String[] split(String regExp)   //通过regExp作为分隔符将字符串分割成数组
    public String[] split(String regExp, Integer limit)   //通过regExp作为分隔符将字符串分割成数组，limit显示数组个数
    public Boolean startsWith(string substring)   //判断字符串是否以substring开头，如果是返回true，不是返回false
    public String substring(Integer length)    //截取字符串固定长度
    public String toLowerCase()   //将字符串转换成小写
    public String toUpperCase()   //将字符串转换成大写
    public String trim()   //去字符串左右空格
    public String uncapitalize()    //将字符串第一个转换成小写
    public static String valueOf(Object objectToConvert)   //将Object类型转换成String类型，其中Object类型包括  Date,DateTime,Decimal,Double,Integer,Long,Object
    ```
### 二、时间日期
+ Datetime

  Datetime类型声明一个日期时间的对象，包含两部分：日期，时间。因为salesforce一般制作global项目，所以日期时间一般取格林时间。Datetime无构造函数，如果实例化只能通过其静态方法初始化。
    ```java
    //以下为Datetime的部分主要方法：
    Datetime nowDatetime = Datetime.now();
    Datetime datetime1 = Datetime.newInstance(2015,3,1,13,26,0);   //初始化
    Datetime datetime2 = Datetime.parse(datetimeString);
    Datetime datetime3 = Datetime.valueOf(datetimeString);
    String datetimeString = '2016-3-1 PM14:38';
    datetime1.format('yyyy-MM-dd HH:mm:ss'));   //格式化
    年月日时分秒操作
    datetime1 = datetime1.addDays(1);
    datetime1 = datetime1.addMonths(1);
    datetime1 = datetime1.addYears(1);
    datetime1 = datetime1.addHours(1);
    datetime1 = datetime1.addMinutes(1);
    datetime1 = datetime1.addSeconds(1);
    Date date1 = datetime1.date();
    Date dateGmt = datetime1.dateGmt();
    //对应时间获取
    Integer year = datetime1.year();
    Integer yearGmt = datetime1.yearGmt();
    Integer month = datetime1.month();
    Integer monthGmt = datetime1.monthGmt();
    Integer day = datetime1.day();
    Integer dayGmt = datetime1.dayGmt();
    Integer dayOfYear = datetime1.dayOfYear();
    Integer dayOfYearGmt = datetime1.dayOfYearGmt();
    Integer hour = datetime1.hour();
    Integer hourGmt = datetime1.hourGmt();
    Integer minute = datetime1.minute();
    Integer minuteGmt = datetime1.minuteGmt();
    Integer second = datetime1.second();
    Integer secondGmt = datetime1.secondGmt();
    ```

+ Date

    Date类型声明一个日期的对象，Date可以和Datetime相互转换，主要需要掌握二者关系以及相互转换。
    ```java
    //以下为Date部分主要方法：
    Date date2 = Date.today();
    Date date3 = Date.newInstance(2016,3,1);
    String dateString = '2016-3-1';
    Date date4 = Date.parse(dateString);
    Date date5 = Date.valueOf(dateString);
    date3.format() //格式化
    System.debug('通过newInstance实例化：' + date3.format());
    System.debug('通过parse实例化：' + date4.format());
    System.debug('通过valueOf实例化：' + date5.format());
    Integer daysBetween = date3.daysBetween(date4);    //  相差天数 date4-date3
    date4和date5是否相同日期   date4.isSameDay(date5)
    date3和date4相差月数  date3.monthsBetween(date4)
    date3.toStartOfMonth().format()); //调用toStartOfMonth执行值    //返回本月第一天
    public Date toStartOfWeek()   //返回本月第一个周日，如果本月1日非周日，则返回上月最晚的周日
    ```

+ Time

    Time 类型声明一个时间的对象，对于时间需要考虑的是：因为中国时间和格林时间相差8小时，所以具体项目时如果是global项目需要考虑使用格林时间，即GMT时间。

### 三、集合

+ List

    List代表一类的有序数据列表。数据序号从0开始。与JAVA不同的是：List是一个类，并且不存在ArrayList等子类。即实例化

    eg:List list1 = new List();

    List可以通过自身构造函数实例化，也可以通过数组进行实例化。

    以下为List主要方法：

    ```java
    //注：set()方法在设置插入位置以前应确保长度大于需要插入的位置，否则将抛出异常。
    List<String> lists = new String[]{'1','3'};
    List<String> list1 = new String[] {'5','4'};
    lists.set(0,'a');
    lists.add(0,'b');
    lists.add('2');
    lists.addAll(list1);
    //lists.sort();
    for(String item : lists) {
    }
    Iterator<String> iterator = lists.iterator();
    while(iterator.hasNext()) {
       String item = iterator.next();
    }
    if(lists.size() > 0) {
       Integer listSize = lists.size();
       lists.remove(listSize-1);
    }
    List<String> cloneList = lists.clone();
    for(Integer i=0;i<cloneList.size();i++) {
       System.debug('cloneListItem : ' + cloneList.get(i));
    }
    lists.clear();
    if(lists.size() > 0) {
       for(String item : lists) {
           System.debug('set item : ' + item);
       }
    } else {
       System.debug('lists has already clear');
    }
    ```

+ Set

    ```java
    Set代表一类数据的无序列表。与JAVA不同的是：Set是一个类，不存在HashSet等子类。即实例化
    eg:Set<String> set1 = new Set<String>();
    Set主要方法如下：
    Set<String> set1 = new Set<String>();
    set1.add('1');
    set1.add('2');
    Set<String> set2 = set1.clone();
    Boolean isEquals = set1.equals(set2);
    Boolean isContains = set1.contains('1');
    Integer setSize = set1.size();
    Iterator<String> iterator2 = set1.iterator();
    while(iterator2.hasNext()) {
          System.debug('set item:' + iterator2.next());
    }
    Boolean isEmpty = set1.isEmpty();
    //set1.remove('1');
    List<String> anotherList = new String[] {'1','3'};
    //public Boolean retainAll(List<Object> list)
    //译：set值为list中和set重复的内容，如果没有重复的内容，则set为空
    set1.retainAll(anotherList);
    Iterator<String> iterator3 = set1.iterator();
    while(iterator3.hasNext()) {
        System.debug('set item via retainAll:' + iterator3.next());//1,List与Set公共部分
    }
    ```

+ Map<K,V>

    Map代表着键值对，与JAVA用法类似，区别为Map是一个类，不是接口，不存在HashMap<K,V>等子类
    ```java
    Map主要方法如下
    Map<String,Object> map1 = new Map<String,Object>();
    map1.put('key1','value1');
    map1.put('key2','value2');
    map1.put('key3','value3');
    Boolean isContainsKey = map1.containsKey('key1');
    Map<String,Object> map2 = map1.clone();
    Boolean isMapEquals = map1.equals(map2);
    Set<String> keySet = map1.keySet();
    String value1 = (String)map1.get('key1');
    List<String> valuesList = (List<String>)map1.values();
    Integer mapSize = map1.size();
    ```

### 四、sObject

+ 所有的对象都是sObject类型，所以当创建任何一个对象时，可以声明为sObject类型。
    ```java
    sObject obj1 = new Account();
    sObject obj2 = new Student__c();
    ```

    上面的代码建立了一个“Account”（标准对象）和“Student__c”（自定义对象）实例。

    sObject类型可以转换为某一对象类型，反之则不行。

    另外，新建sObject类型的实例只能通过函数newSObject()，而不能通过new关键字
    
    如：sObject sObj = Schema.getGlobalDescribe().get(‘Account’).newSObject();
