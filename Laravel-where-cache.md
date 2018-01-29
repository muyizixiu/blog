---
title: Laravel where 条件缓存
date: 2018-10-28
categories:
- 技术
- php
---

# Laravel where 条件缓存

在Laravel中，使用门面提供的DB类，以QueryBuilder的形式去操作数据库是一种很常见的形式。由DB::table($tablename)提供的对象即是QueryBuilder类的。这里面值得注意的一点是：在获取完数据之后，QueryBuilder 对象并不会释放其缓存内容。这里的缓存内容指的是查询条件，例如：where,join,groups等条件。这有可能会导致一个问题，下面是问题代码的示范:

```
<?php
\DB::listen(function($query){
    echo "\n",$query->sql;
});
\DB::table('seeed_user')->select('*')->where('id','=','1')->get();//正常查询
\DB::table('seeed_user')->select('*')->where('id','!=','1')->get();//正常查询
$table = \DB::table('seeed_user');//缓存QueryBuilder对象
$table->select('*')->where('id','=','1')->get();//正常查询
$table->select('*')->where('id','!=','1')->get();//这里的sql查询不正常,受上一条查询sql的影响
```

其打印内容为:

```
select * from `seeed_user` where `id` = ?
select * from `seeed_user` where `id` != ?
select * from `seeed_user` where `id` = ?
select * from `seeed_user` where `id` = ? and `id` != ?
```
前两条sql明显是正确的，在缓存QueryBuilder对象之后，最后一条sql明显收到了前一条sql的影响。

这里的测试是基于lumen框架(laravel框架的简化版)和phpunit，完整代码如下:

```
<?php

class ExampleTest extends TestCase
{

    public function testQueryBuilder()
    {
        \DB::listen(function($query){
            echo "\n",$query->sql;
        });
        \DB::table('seeed_user')->select('*')->where('id','=','1')->get();
        \DB::table('seeed_user')->select('*')->where('id','!=','1')->get();
        $table = \DB::table('seeed_user');
        $table->select('*')->where('id','=','1')->get();
        $table->select('*')->where('id','!=','1')->get();
    }
}
```
使用phpunit --verbose 选项可以观察到sql的打印。

另外两个值得注意的源码知识点是：

1. 在laravel中开启了withEloquent（在bootstrap/app.php中），实际上就有了class_alias(类别名)，就如同上面的DB类，可以直接在使用根命名空间\DB;
2. Laravel的Database的实现如果想迁移到其余框架里面，除了向laravel一样注入配置，还可以利用里面实现的Capsule类
