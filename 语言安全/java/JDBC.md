
常见api在 java.sql包里，需要与数据库版本相对应的jar包，加载到web project里的lib文件中
## 1.加载驱动
Class.forName("com.mysql.jdbc.Driver");
// mysql8 com.mysql.cj.jdbc.Driver
## 2.建立连接

String url="jdbc:mysql:///数据库名称?参数键值对";
Connection con=DriverManager.getConnection(url);
## 3.创建预处理语句对象
sql="insert into student(sname,age,major) values (?,?,?)"
PreparedStatement psmt=con.preparedStatement(sql);
psmt.setString(1,"张三");//插入第一个参数
pstm.setInt(2," 18");//插入第二个参数
psmt.setString(3,"网络工程");//插入第3个参数
## 4.执行sql语句
Statement类的方法
executeQuery  执行select
executeUpdate 执行 insert update delete 和数据定义语言 SQL DDL
## 5.关闭连接
pstm.close();
con.close();

## 6.打印数据库
ResultSet rs=psmt.executeQuery();

while(rs.next（）){
int studentid=rs.getInt(" studentid");
String sname=rs.getString(" sname");
int age=rs.getInt(" age");
 String major=rs.getString("major");

}
