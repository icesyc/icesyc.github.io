PHP编码规范
==========

本文档描述了在php开发中应该遵守的规范，对代码风格的统一起到指导作用。

目录文件及编码
---------

文件名需要遵守yii2框架的约定，controller和model需要使用驼峰式(camelize)的命名规则，而view文件需要使用下划线(underscore)的命名规则，如：

```
控制器：TestController.php PayOrderController.php
模型: Test.php PayOrder.php
视图：test.php pay_order.php
```

注意：yii的generator在生成view时会给每个控制器生成一个对应view的目录，目录的命令规则是中间线(dash)的方式，如 pay-order，如果需要为控制器使用单独的目录，请遵守此规则。

文件的编码统一使用UTF-8编码

目录名必须全部小写

格式
---

###大括号

- 大括号与if,else,for,do,while一起使用时，即使只有一条语句，也要加应该把大括号加上。
- 左大括号与声明语句在同一行，即括号左边不需要换行，右边需要。
- 右大括号需要单独占一行(使用函数表达式时除外)。

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
if ($user->score > 100){
	$user->score = $user->score - 100;
}
$str = $str . 'another string';
```

函数的参数之间需要有空格

```
function addScore($user, $type, $status){
	...
}
```

类的方法和方法之间需要有空行

```
function func1(){
    ... 
}

function func2(){
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

- 接口和类的命名需要使用驼峰式，首字母大写，如 UserScore
- 变量及函数的命名需要使用驼峰式，首字母小写，如 $userScore, getUserScore($user)
- 常量的定义采用大写+下划线的方式，如 ORDER_STATUS
- mysql的库名、表名、和字段名都需要使用全部小写+下划线的规则，以避免在不同系统下产生问题。

编程实践
-------

###字符串

字符串优先使用单引号，如果字符串里包含变量，可以使用双引号或使用sprintf来进行格式化，如

```
$str = 'this is a test';
$str2 = sprintf('user name is %s, age is %d', $name, $age);
$str3 = "user name is $ice, age is $age";
```

###函数

单个函数的行数应该在100行以内，超过100行的要考虑拆分成多个函数，保持函数的功能尽量简单。

###常量定义
对于数据库表示状态或类别的定义，使用tinyint(1)来存储，并在Model中使用常量定义不同数字表示的含义，如表示订单状态的字段status可以表示如下：

```
const STATUS_NOT_PAY = 0; //未支付
const STATUS_PENDING = 1; //待处理
const STATUS_DONE = 2; //已完成
const STATUS_CANCEL = 3; //已取消
```

在代码中尽量避免直接使用常量来判断状态，而是提供对应的方法，如
```
class Order{
	public function isPending(){
		return $this->status == self::STATUS_PENDING;
	}
}
...
$order = Order::findOne($id);
if ($order->isPending()){
	...
}
```

常量的定义是为了使数字具有更明确的含义，而使用方法既可以使代码的可读性更好，也对外面隐藏了常量及判断的逻辑细节，这样即使判断的逻辑需要调整，也只需要改对应的函数。

###时间字段

对于mysql，日期和时间有对应的数据类型date和datetime，使用date和datetime类型时，我们需要存储格式化之后的日期，即2016-12-20这样的字符串，并可以对该字段进行比较查询，输出的时候也无须做任何转换。使用date类型在程序中比较时间时需要做一次转换，在判断空日期的时候也需要使用
```
if ($date == '0000-00-00'){
	...
}
```
这样的代码才能完成。

另外就是使用int(11)来保存unix timestamp，在程序中判断时由于是数字，可以直接比较，代码会显示更直观和整洁
```
if ($date > 0){
	...
}
```
但是用户在创建数据时一般会使用日期控件来输入，这时需要将输入日期转换成unix timestamp才能入库，通常有2种做法

使用Model rules的filterValidator来处理，在Model中增加新的rule
```
[['date', 'filter', function($value){
	return strtotime($value);	
}]]
```

在Model的beforeSave方法里处理

```
public function beforeSave($insert)
{
    if (parent::beforeSave($insert)) {
    	$this->date = strtotime($this->date);
        return true;
    } else {
        return false;
    }
}
```

至于哪种方案更好一些，我认为要看使用的场景，如果只是做为一种显示需要，使用mysql的date类型则更简单，如果需要对该字段进行很复杂的条件判断，我更倾向于unix timestamp。

###使用策略模式代替过多的if和else

当代码逻辑中出现很多个if elseif的判断，就要考虑这个问题了，比如：

```
if ($order->status == 1){
	...
} elseif ($order->status == 2){
	...
} elseif ($order->status == 3){
	...
} elseif ($order->status == 3){
	...
}
```

上面的逻辑可能是想根据订单的不同状态有不同的处理逻辑，如果状态很多就会让代码变得非常难看，这种情况下通常可以使用策略模式解决这个问题。

```
public $processors = [ 
	self::STATUS_PENDING => 'Pending',
	self::STATUS_DONE => 'Done',
	self::STATUS_CANCEL => 'Cancel'
];

public function process(){
	$processor = 'process' . $this->processors[$this->status];
	return $this->$processor();
}

public function processPending(){
	...
}

public function processDone(){
    ...
}

public function processCancel(){
    ...	
}
```

由于php的动态语言特性，我们不需要反射机制也可以动态的调用方法。

###参数验证

有的时候我们需要对很多参数进行验证，或对同一参数多次验证，这时候如果写的不好，也很能写成这样

```
if (Util::isEmail($_GET['email'])){
	return $this->error('email is error');
} elseif ($_GET['username'] == ''){
	return $this->error('username is empty');
} elseif ($_GET['password'] == ''){
	return $this->error('password is empty');
}
....
```
这时我们又陷入了大量if else的困局。想一下，能不能事先把规则和消息配置好，再统一循环验证？，其实这就是Yii框架数据验证的方式，Yii框架本身提供了大量的Validator，配合Model可以让验证变得非常容易

```
class UserForm extends Model{

	public function rules(){
	    return [
	        [['username', 'password'], 'required', 'message' => '用户名密码不能为空'], //不能为空
	        [['email'], 'email', 'message' => '邮箱格式不正确'], //email格式
	        [['age'], 'integer', 'min' => 0], //年龄必须是整数，不能是负数
	    ];
	}
}
```

使用定义好的验证逻辑

```
$form = new UserForm();
$form->load($_POST);
if(!$form->validate()){
	$errors = $form->errors;
	...
}
```

###避免代码嵌套层次过多

代码层次如果太深，会影响我们正常阅读，同时也会让逻辑变得非常混乱。  
在函数的章节时，我们建议应该让函数的代码尽量控制在100行以内，这也有助于减少代码的层次。  
通常函数内的代码嵌套不应该超过3层，比如下面这样是非常糟糕的风格：

```
if($a == 1){
	if($b == 2){
		foreach($users as $user){
			if($user->id > 10){
				if($user->age < 20){
					...
				}
			}
		}
	}
}
```

该如何避免呢？有三条建议你可以考虑：

1. 合并条件，看看能否将多个判断条件合并成一条
2. 将否定条件写在前面，条件不符合就直接返回
3. 在循环中使用continue和break来先处理否定条件

如上面的代码可以改写成：

```
if($a != 1){
	return false;
}
if($b != 2){
	return false;
}
foreach($users as $user){
	if($user->id <= 10 || $user->age >= 20){
		continue;
	}
	...
}
```

###精简代码

使用三元操作符来代替if else，如

```
if($user->isLogin()){
	$score = 10;
}else{
	$score = 20;
}
```

可以写成

```
$score = $user->isLogin() ? 10 : 20;
```






