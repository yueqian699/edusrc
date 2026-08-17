先看三种型和对应后端代码  
数字型  
select * where users from id = 2-1  


字符型  
select * where users from id = '1'  


搜索型  
SELECT * FROM users WHERE username LIKE '%a%';


$id=$_GET['id'];  

 
$sql="SELECT * FROM users WHERE id='$id' ";  

 
$result=mysql_query($sql);  

