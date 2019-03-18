mysql版本 mysql-installer-community-8.0.15.0.msi ： 下载地址：https://dev.mysql.com/downloads/file/?id=484920

mysql安装：https://www.cnblogs.com/ayyl/p/5978418.html
mysql server/client protocol port:3306
mysql x protocol port:33060
root 密码:123456
windows service name:MySQL80
默认安装路径:C:\Program Files\MySQL
默认配置路径：C:\ProgramData\MySQL\MySQL Server 8.0

mysql官方：https://dev.mysql.com/
安装和环境配置文档 MySQL Connector/C++  8.0.15:
 https://dev.mysql.com/doc/connector-cpp/8.0/en/
 https://dev.mysql.com/doc/connectors/en/



- API使用文档:https://dev.mysql.com/doc/x-devapi-userguide/en/


- Mysql 手册:https://dev.mysql.com/doc/refman/8.0/en/


- Building Connector/C++ Applications on Windows with Microsoft Visual Studio:
https://dev.mysql.com/doc/connector-cpp/8.0/en/connector-cpp-apps-windows-visual-studio.html

Connect to a MySQL database from Visual Studio 2017:http://pinter.org/archives/6450

insidemysql博客:https://insidemysql.com/


win10  Access denied for user 'root'@'localhost'：
https://blog.csdn.net/chencaw/article/details/79597618 -> 文章中应该是输入:mysql> alter user'root'@'localhost' IDENTIFIED BY '123456';
https://blog.csdn.net/Carl_Qi/article/details/51469456

[《使用MySQL Workbench建立数据库，建立新的表，向表中添加数据》](https://www.cnblogs.com/jpfss/p/6647598.html)

无法定位序数 4233 于动态链接库：[链接](https://stackoverflow.com/questions/6534505/how-to-fix-libeay32-dll-was-not-found-error)

登陆Mysql 命令行:mysql -u root -p
查看MySql加载了哪些插件:show plugins;
查看插件目录:show variables like 'plugin_dir'

加载插件:https://www.cnblogs.com/Focus-Flying/p/9316506.html
https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_plugin-load
https://dev.mysql.com/doc/refman/8.0/en/plugin-loading.html#server-plugin-installing

查看端口:
netstat -ano | findstr "33060"
查看进程：
tasklist | findstr "4932"