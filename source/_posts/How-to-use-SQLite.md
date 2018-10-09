---
title: How to use SQLite
date: 2017-07-25 15:39:17
tags:
---

<br>*本文作者 Pantiy , 转载请注明出处*
>SQLite 是一种轻量级的数据库，实现了大多数 SQL 标准。它使用动态的、弱类型的 SQL 语法。在本地/客户端存储数据的常见选择。在 Android 的开发过程中，与 SQLite 打交道可以说是不可避免的。

**常规的使用方法**  

通常在使用过程中需要定义以下3种类：  

- XXXDatabase
- XXXSQLiteOpenHelper
- XXXCursorWrapper  

**1.XXXDatabase**  

这个类用于定义在 SQLite 中所需要创建的表单的 “名称”、“字段” 。例如，需要为班内同学的 "姓名" 、“学号” 进行储存，则创建一个名为 ClassmateInfoDatabase 的静态类，代码如下：
```
public final class ClassmateInfoDatabase {

    public static final String NAME = "classmate_info";

    public static String getSql() {
        return "create table " + NAME + "(" +
                Table.STUDENT_NUM + "," +
                Table.STUDENT_NAME + ")";
    }

    public static final class Table {
        public static final String STUDENT_NUM = "student_num";
        public static final String STUDENT_NAME = "student_name";
    }
}
```
- NAME 定义了表单的名字
- Table 定义了表单中的各个字段   

为什么需要用一个类来进行定义？因为表单的名称、字段都会在操作数据库时频繁被使用，这样做是为了最大程度地进行复用以及日后的维护。  

**2.XXXSQLiteOpenHelper**  

这个类与帮助你获取 Database 实例对数据库进行操作，这个类需要继承自 SQLiteOpenHelper 。例如，为上述的 ClassmateInfoDatabase 创建一个名为 ClassmateInfoSQLiteOpenHelper 类，代码如下：
```
public class ClassmateInfoDatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "classmateInfo.db";
    private static final int VERSION = 1;

    public ClassmateInfoDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(ClassmateInfoDatabase.getSql());
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
```
- DATABASE_NAME 顾名思义，表示数据库的名称
- VERSION 代表当前数据库的版本号
- onCreate()   
该方法会在第一次创建数据库时进行调用，db.execSQL(String sql) 方法所需要的参数就是标准的 SQL 语句，用于创建表单
- onUpgrade()  
该方法会在升级数据库时进行调用。例如，你发布的下一个 App 版本中 VERSION 值变大，表示有新版本数据库，则系统会调用该方法

**3.XXXCursorWrapper**  

这个类用于将数据库查询的结果包装成所定义的 POJO 类，用于数据填充，该类需要继承自 CursorWrapper 类。例如，创建一个名为 ClassmateInfoCursorWrapper 的类，代码如下：  
```
public class ClassmateInfoCursorWrapper extends CursorWrapper {

    public ClassmateInfoCursorWrapper(Cursor cursor) {
        super(cursor);
    }

    public ClassmateInfo getClassmateInfo() {

        String studentNum = getString(getColumnIndex(ClassmateInfoDatabase.Table.STUDENT_NUM));
        String studentName = getString(getColumnIndex(ClassmateInfoDatabase.Table.STUDENT_NAME));

        return new ClassmateInfo(studentNum, studentName);
    }
}
```
- getClassmateInfo()  
该方法是自定义的，用于将数据库查询的结果转换成所需的 POJO 类的实例

**4.总结**  

以上内容展示了实现 SQLite 本地化储存所必须的3个类。XXXDatabase 类中对表单的名称、字段进行了定义；XXXSQLiteOpenHelper 类实现了数据库的初始化创建，以及实例化 SQLiteDatabase 对象，用于操作数据库；XXXCursorWrapper 类实现了将数据库查询结果转换成 POJO 类的功能。  

--------------------------

*欢迎关注我的个人微信订阅号「 PantiyShare 」，我会在上面不定时地分享一些原创文章，不只有技术文章呦。*
