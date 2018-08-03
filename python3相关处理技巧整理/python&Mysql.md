Mysql的安装：

网址： <http://dev.mysql.com/downloads/mysql/> 

解压缩版，下载：Windows (x86, 64-bit), ZIP Archive

以管理员身份打开命令行，进入解压缩后的bin目录，mysqld -install

若出现：缺失MSVCR120.dll的报错

下载 [VC redist packages for
x64](https://www.microsoft.com/en-us/download/details.aspx?id=40784),安装位置默认。重新mysqld
–install

F:\\Develop\\Mysql\\mysql-5.7.23-winx64\\bin\>mysqld -install

Service successfully installed.

F:\\Develop\\Mysql\\mysql-5.7.23-winx64\\bin\>net start mysql

MySQL 服务正在启动 .

MySQL 服务无法启动。

服务没有报告任何错误。

请键入 NET HELPMSG 3534 以获得更多的帮助。

在mysql5.7以上版本中默认没有一个data目录，即没有初始化服务。需要先初始化mysql才可以启动服务。正确的步骤是:先在mysql的bin目录下执行mysqld
--initialize-insecure （不设置root密码，建议使用）命令，若已经执行过net start
mysql，删除data文件，重新执行mysqld --initialize-insecure。

https://jingyan.baidu.com/article/da1091fb1a46a6027849d6a8.html

此时net start mysql 服务器启动成功

首次登陆：mysql –u root –p

第一次进入后要修改密码否则不能进行操作，否则会报错

修改密码（注意密码要求1.最少8个字符 2.包含大写 和 小写字母 3.要有标点符号）：SET
PASSWORD = PASSWORD('new password');

Python anacoda连接mysql

安装pymysql

Pycharm和anacoda prompt窗口无法安装成功

用pip install pymysql 安装到了如下目录

F:\\ProgramData\\Anaconda3\\pkgs\\sqlalchemy-1.2.7-py36ha85dd04_0\\Lib\\site-packages\\sqlalchemy\\dialects\\mysql

将pymysql.py拷贝至：

F:\\ProgramData\\Anaconda3\\envs\\Guangzhou\\Lib\\site-packages

提示：

ImportError: attempted relative import with no known parent package

Python 操作mysql示例：

import pymysql

\# 打开数据库连接

db = pymysql.connect("localhost", "root", "test123", "TESTDB")

\# 使用 cursor() 方法创建一个游标对象 cursor

cursor = db.cursor()

\# 使用 execute() 方法执行 SQL，如果表存在则删除

cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")

\# 使用预处理语句创建表

sql = """CREATE TABLE EMPLOYEE (

FIRST_NAME CHAR(20) NOT NULL,

LAST_NAME CHAR(20),

AGE INT,

SEX CHAR(1),

INCOME FLOAT )"""

cursor.execute(sql)

\# 关闭数据库连接

db.close()

[优化Python对MySQL批量插入的效率](https://www.cnblogs.com/hyace/p/4173831.html)

1、insert的时候尽量多条一起插，不要单条插。这样可以减少日志量，降低日志刷盘数据量和频率，效率提高很多。

这是文章提供的测试对比：

![https://images0.cnblogs.com/blog/316027/201412/191419430163905.jpg](media/ed14d97a4e85bf9ebe24e1c489e0c13a.jpg)

2、在事务中进行插入。也就是每次部分commit，否则每条insert commit
创建事务的消耗也是不小的。

![https://images0.cnblogs.com/blog/316027/201412/191419553293001.jpg](media/b3fd0097f02b4fa33767adadde643478.jpg)

3、
数据插入的时候保持有序。比如Innodb用的是B+树索引，对B+树的插入如果是在索引中间就会需要树节点分裂合并，这也会有一定的消耗。

![https://images0.cnblogs.com/blog/316027/201412/191420095488711.jpg](media/70df2f1817967966e1075331d6f9800d.jpg)

几种方法综合起来的测试数据如下：

![https://images0.cnblogs.com/blog/316027/201412/191420251412408.jpg](media/63265f4004e17f4b864d9a03566f64ce.jpg)

　　可以看到1000万以下数据的优化效果是明显的，但是单合并数据+事务的方式在1000万以上性能会有急剧下降，因为此时已经达到了innodb_buffer的上限，随机插入每次需要大量的磁盘操作，性能下降明显。而有序插入在1000万以上时也表现稳定，因为索引定位方便，不需要频繁对磁盘读写，维持较高性能。

\# def data_store(df,db):

\# db = pymysql.connect('localhost', 'root', 'password', 'guangzhouIC')

\# cur = db.cursor()

\# cur.execute('use guangzhouIC')

\# sql = "insert into ic_up (LOGCARDID,UPBST,TIME,DATE) values"

\# for i in np.arange(len(df)):

\# sql = sql + '('+
str(df.iloc[i].values[0])+','+df.iloc[i].values[1]+','+df.iloc[i].values[2].isoformat()+','+df.iloc[i].values[2].isoformat()+'),'

\# sql = sql[:-1]+';'

\# cur.execute(sql)

\# cur.close()

Pandas.dataframe直接存储Mysql数据库：

from sqlalchemy import create_engine

conn = create_engine('mysql+pymysql://root: password
\@localhost:3306/guangzhouIC?charset=utf8')

pd.io.sql.to_sql(df, 'ic_up', conn, schema='guangzhouIC', if_exists='append',
index=False)

'ic_up'：数据表

if_exists='append' 如果表存在，在后面追加，如果不存在，建立此表。

index=False :设置了自增的id，故无需导入index。

Schema：对应数据库*。*

<http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.to_sql.html?highlight=to_sql#pandas.DataFrame.to_sql>

查看库和表的信息：

<https://blog.csdn.net/bzfys/article/details/55252962>

<https://www.cnblogs.com/cymwill/p/8289367.html>

from sqlalchemy import create_engine

conn = create_engine('mysql+pymysql://root:root
\@localhost:3306/guangzhouIC?charset=utf8')

def select_data(date_,time_,conn):

sqlTable\_ = 'ic_up'

sql = "select \* from "+ sqlTable\_ + " where date between "+ date_[0] + " and "
+ date_[1]

if not time_:

sql = sql + ";"

else:

sql = sql + " and time between "+ time_[0] + " and " + time_[1] + ";"

print(sql)

frame = pd.read_sql(sql, conn, index_col="IC_up_id")

if not frame:

print("日期："+ str(date_) +", 时间段：" + str(time_) + " 查询结果为空！")

frame = pd.DataFrame()

return frame
