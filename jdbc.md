JDBC:Java DateBase Connectinity

java访问数据库

问题：如何访问U盘数据？

java访问数据库的步骤：

1.加载驱动  Class.forName("com.mysql.jdbc.Driver");

2.注册驱动，打开连接对象  Conection con=DriverManager.getConnectoin("jdbc:mysql://l27.0.0.1:3306?atm","root","root");

3.由连接对象产生语句处理对象Statement stmt=con.createStatement();

4.操作数据，若是查询，创建结果集对象

5.关闭链接，释放资源



注意：针对于增删改操作：executeUpdate()

​           针对于查询操作：executeQuery();

```java
public class BaseDao {
    public static void main(String[] args) {
        String driver="com.mysql.jdbc.Driver";
        String url="jdbc:mysql://localhost:3306/flower?useSSL=false";
        String user="root";
        String pwd="root";
        Connection con=null;
        Statement stmt=null;
        ResultSet rs=null;
        try {
            Class.forName(driver);//加载驱动
            con= DriverManager.getConnection(url,user,pwd);//创建连接对象
            System.out.println(con);
            //创建语句处理对象
            stmt=con.createStatement();//创建语句处理对象
            //insert,update,delete 都是用executeUpdate()
            //操作数据，如果是查询创建结果集对象
//            String sql="insert into users values('1','王五','123456','男','1999-1-1','15891358888',1)";
//            stmt.executeUpdate(sql);
//            System.out.println("添加成功！");
//            String sql="update users set username='赵六',sex='女' where userid='1'";
//            stmt.executeUpdate(sql);
//              String sql="delete from users where userid='1'";
//              stmt.executeUpdate(sql);
            //query---->executeQuery()
//            String sql="select * from users where userid='1001'";
//            rs=stmt.executeQuery(sql);
//            //获取字段的值：根据字段名称
//            if(rs.next()){
//                System.out.println("用户编号："+rs.getString("userid")+"用户姓名："+rs.getString("username")+
//                        "密码："+rs.getString("password")+"性别："+rs.getString("sex"));
//            }
            String sql="select * from users";
            rs=stmt.executeQuery(sql);
            //获取字段的值：根据字段名称
            while(rs.next()){
                System.out.println("用户类型："+rs.getInt("type")+"\t用户编号："+rs.getString("userid")+"用户姓名："+rs.getString("username")+
                        "密码："+rs.getString("password")+"性别："+rs.getString("sex"));
            }
        }catch(ClassNotFoundException e){
            e.printStackTrace();
        }catch(SQLException e){
            e.printStackTrace();
        }finally {//关闭链接，释放资源
            try {
                if (rs != null) {
                    rs.close();
                }
                if(stmt!=null){
                    stmt.close();
                }
                if(con!=null||!con.isClosed()){
                    con.close();
                }

            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}

```



sql注入：你写的sql语句有漏洞。

‘or’1'='1

```
String sql = "select * from users where user_card='" + kh + "' and user_pwd='" + mm + "'";
```

PreparedStatement :

使用预编译的好处：

1、可以防止sql注入

2.多次执行同一sql效率高

3.可阅读性高

预编译使用步骤：

1.带参sql

2.给参数赋值

3.执行

```java
public class Atm {
    public static void main(String[] args)
    {
        String driver="com.mysql.jdbc.Driver";
        String url="jdbc:mysql://localhost:3306/atm?useSSL=false";
        String user="root";
        String pwd="root";
        Connection con=null;
//        Statement stmt=null;
        PreparedStatement psmt=null;//声明预编译处理对象
        ResultSet rs=null;
        Scanner sc=new Scanner(System.in);
        try{
            Class.forName("com.mysql.jdbc.Driver");
            con= DriverManager.getConnection(url,user,pwd);
            System.out.println("欢迎来到中国银行！");
//            stmt=con.createStatement();
            int count=0;
            String kh=null;
            String mm=null;
            while(true) {
                count++;
                System.out.println("请输入卡号：");
                kh = sc.next();
                System.out.println("请输入密码：");
                mm = sc.next();
                //String sql = "select * from users where user_card='" + kh + "' and user_pwd='" + mm + "'";
                String sql="select * from users where user_card=? and user_pwd=?";
//                System.out.println(sql);
                psmt=con.prepareStatement(sql);
                psmt.setString(1,kh);
                psmt.setString(2,mm);
                rs=psmt.executeQuery();
//                rs = stmt.executeQuery(sql);
                if (rs.next()) {
                   break;
                } else {
                    if(count!=3) {
                        System.out.println("卡号密码不正确！");
                        continue;
                    }
                    else
                        {
                            System.out.println("您的输入次数过多，卡已吞，请到柜台办理！");
                            System.exit(0);
                        }
                }

            }
            while(true) {
                System.out.println("请选择操作：1.取款2.查询余额3.退卡.........");
                int n = sc.nextInt();
                if (n == 1) {
                    while(true){
                    System.out.println("请输入取款金额：");
                    int q_money = sc.nextInt();
                    if (q_money % 100 == 0) {
                        String sql1 = "select user_money from users where user_card='" + kh + "'";
                        rs = psmt.executeQuery(sql1);
                        int y_money = 0;
                        if (rs.next()) {
                            y_money = (int) (rs.getFloat("user_money"));
                        }
                        if (y_money >= q_money) {
                            String sql2 = "update users set user_money=user_money-'" + q_money + "' where user_card='" + kh + "'";
                            psmt.executeUpdate(sql2);
                            System.out.println("取款成功！");
                            break;
                        } else {
                            System.out.println("取款不足！");
                        }

                    } else {
                        System.out.println("本机只能取整《100的整数倍》");
                    }
                }
                } else if (n == 2) {
                    String sql3="select user_money from users where user_card='"+kh+"'";
                    rs=psmt.executeQuery(sql3);
                    int y_money=0;
                    if(rs.next()){
                        y_money=(int)rs.getFloat("user_money");
                    }
                    System.out.println("您的余额："+y_money);
                } else if (n == 3) {
                    System.out.println("谢谢使用");
                    System.exit(0);

                }

            }
        }catch(Exception e){
            e.printStackTrace();
        }finally {

        }






    }
}

```





jdbc封装：