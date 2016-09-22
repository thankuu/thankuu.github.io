---
title: "PHP安全 - SQL防注入"
tags:
    - php
    - 安全
    - CodeIgniter
    
---
### 1、表单校验
这部分其实和安全关系不大，个人理解表单校验只是为了防止用户在前端直接调用接口来绕过前端的表单验证。而通常将针对获取到的表单进行校验，这部分功能目前一般在所有框架中都具备有，以`CodeIgniter`举例,官方给的示例是：


    public function index()
    {
        $this->load->helper(array('form', 'url'));

        $this->load->library('form_validation');

        $this->form_validation->set_rules('username', 'Username', 'callback_username_check');

    $this->form_validation->set_rules('password', 'Password', 'required');

        $this->form_validation->set_rules('passconf', 'Password Confirmation', 'required');

    $this->form_validation->set_rules('email', 'Email', 'required|is_unique[users.email]');

        if ($this->form_validation->run() == FALSE)
    {
        $this->load->view('myform');
    }
    else
    {
        $this->load->view('formsuccess');
        }
    }

这种方式的目的就是将前端传过来的数据进行初步校验，校验的方式一般有特殊校验（邮箱，密码，日期）和一般校验（是否必填）等等，通过这个方式可以对前端的数据进行格式以及类型的初步校验，检验其变量格式和类型，（预格式化数据）是否包含HTML代码等，对数据进行过滤。

### 2、过滤特殊符号
在所有攻击中，可以通过对表单数据过滤特殊符号` \ , ' , " `来预防SQL注入攻击。假设我们可以通过这样一个链接`http://dev.com/index.php?user_name=thankuu`向服务器请求这个用户的信息，如果`username`是合法的，那么服务器会执行这样的SQL语句：

    SELECT uid,user_info FROM table.user WHERE username=`thankuu`;

但是，如果我们在向服务器请求的内容中加入这样的内容：`thankuu' UNION SELECT username,password FROM table.user-- `，那么服务器最终会执行的SQL语句将会变为：

    SELECT uid,user_info FROM table.user WHERE username=`thankuu` UNION SELECT username,password FROM table.user-- ;

执行的结果就是将所有用户的用户名和对应的密码全部输出返回。同理，我们也可以使用类似的方法跳过网站的用户密码验证，只用知道正确的用户名就可以登录网站。

为了防范这种方式的注入以及插入`<script>`脚本代码，我们可以在php代码中使用`addslashes`函数，将以上特殊符号进行转意，并将原有SQL语句：


    SELECT * FROM table WHERE var={$var};

替换成

    SELECT * FROM table WHERE var='{$var}';

我们的SQL语句中带有单引号，所以要想用户执行注入时就必须反引号，但是我们在接受数据时，将原有数据中的特殊符号进行了转意，那么用户插入的特殊符号将会转意成`\\,\',\"`,，所以想要执行的注入攻击就会失效。

### 3、绑定变量，使用预编译语句
MySQL的mysqli驱动提供了预编译语句的支持，不同的程序语言，都分别有使用预编译语句的方法，现代PHP框架中，使用ORM（关系数据模型）都基本具备了预编译。CodeIgniter的AR防注入能力还是很不错的,使用ORM可以有效防止注入，如果执行的SQL较为复杂，需要书写原生SQL时可以采用参数绑定的方法：


    $sql = "SELECT * FROM table WHERE var=? and var2=?";

    $result = $this->db->query($sql,array($array1,$array2))->result_array();

更多使用方法可以参考官方使用手册。
