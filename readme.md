PHP代码规范
==========

本文档描述了在php开发中应该遵守的规范，对代码风格的统一起到指导作用。

文件及编码
---------

文件名需要遵守yii2框架的约定，controller和model需要使用驼峰式(camelize)的命名规则，而view文件需要使用下划线(underscore)的命名规则，如：

```
控制器：TestController.php PayOrderController.php
模型: Test.php PayOrder.php
视图：test.php pay_order.php
```

注意：yii的generator在生成view时会给每个控制器生成一个对应view的目录，目录的命令规则是中间线(dash)的方式，如 pay-order，如果需要为控制器使用单独的目录，请遵守此规则。

文件的编码统一使用UTF-8编码

格式
---

###大括号

- 大括号与if,else,for,do,while一起使用时，即时只有一条语句，也要加应该把大括号加上。
- 左大括号前面不需要换行
- 左大括号后面需要换行
- 右大括号需要单独占一行(使用函数表达式时除外)

如

```
    if ($user->isLogin()){
        $user->addScore($score);
    } else {
        $this->error('用户');
    }
```

使用函数表达式的情况

```
    usort($array, function($a, $b){
    	return $a - $b;
	});
```

###空白

赋值、比较、逻辑、运算、字符串操作符的两端需要有空格

```
    $user = Yii::$app->user;
    if($user->score > 100){
    	$user->score = $user->score - 100;
    }
    $str = $str . 'another string';
```
函数有参数列表中间需要有空格

```
    function addScore($user, $type, $status){
    	...
    }
```

###缩进

统一使用4个空格，不要使用tab来缩进

###数组

数组使用[]来定义，不要使用array()的语法。  
数组的定义如果超过80个字符，需要使用多行分隔，数组的最后一个元素后在也要添加逗号，这样在添加新元素的时候会比较方便。

```
    $config = [
    	'host' => 'localhost',
    	'username' => 'mysql',
    	'password' => 'mysql',
    ];
```

命名规则
-------

- 接口、类和trait的命名需要使用驼峰式，首字母大写，如 UserScore
- 变量及函数的命名需要使用驼峰式，首字母小写，如 $userScore, getUserScore($user)
- 常量的定义采用大写+下划线的方式，如 ORDER_STATUS

mysql的库名、表名、和字段名都需要使用全部小写+下划线的规则，以避免在不同系统下产生问题。

编程实践
-------


