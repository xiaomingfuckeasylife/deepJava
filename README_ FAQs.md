## 记录自己所遇到的一些工作中的问题以及寻找到的答案

### 1 如何备份oracle数据库

* 导出：exp userid=yourusername/youruserpassword@Connect_Identifier File=OSPath
* 导入：imp userid/password@Connect_identifier fromuser=user_name_you_have_data_unloaded_from touser=new_user_name file=Path_to_*.dmp file

[传送门](http://stackoverflow.com/questions/12419340/how-to-backup-and-restore-oracle-database-11g-like-sql2005-database)

### 2 proxy对象equals方法的陷阱

```java
package com.zwdai.cde.modules.test.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
/*
 * this is an proxy handler we can make a proxy connection out of it 
 */
public class Reporter implements InvocationHandler {

	private Connection conn;
	private static boolean logOn = false;
	
	public Reporter(Connection conn) {
		this.conn = conn;
	}
	
	public static void setLog(boolean on) {
		Reporter.logOn = on;
	}
	
	@SuppressWarnings("rawtypes")
	public Connection getConnection() {
		Class clazz = conn.getClass();
		return (Connection)Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{Connection.class}, this);
	}
	
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		try {
			if (method.getName().equals("prepareStatement")) {
				String info = "Sql: " + args[0];
				if (logOn){
					
				}
				else
					System.out.println(info);
			}
			return method.invoke(conn, args);
		} catch (InvocationTargetException e) {
			throw e.getTargetException();
		}
	}

	public Connection getOriginalObject(){
		return conn;
	}
	
}
// the test code would be something like this . 
public class Test{

  public static void main(String[] args){
    Connection conn = pool.getConnection()
    Connection conn1 = new Reporter(conn).getConnection();
    
    conn1.equals(conn1); // false .  this is actually the original connection compare with the proxy object's memory address. which is the same as conn == conn1 obvious it is not true. 
    
    // also there is something else you should be aware of the proxy object can be nested .which make a proxy out of a proxy.
  }

}
```


