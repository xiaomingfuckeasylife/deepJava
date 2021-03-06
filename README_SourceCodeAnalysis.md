## ConcurrentHashMap源码分析

### 讨论点

*  这个是我在看AbstractMap的源码片段发现了大量的这样的类似的代码
```java
public boolean containsValue(Object value) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (value==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getValue()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (value.equals(e.getValue()))
                    return true;
            }
        } 
        return false;
}
```
给大家的第一感觉是不是很重复，想要将这两个while给合并起来。
```java
public boolean containsValue(Object value) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        // 这段代码我尝试这么改成这样，但是我发现在我的这段代码中，虽然更加清爽，但是其实效率很低，
        // 因为在循环中每次都需要进行一个三元判断这样和上面的代码相比就会差很大的效率问题。效率是最重要的，其次是重构代码
        while(i.hasNext()){
        	Entry<K,V> entry = i.next();
        	if(value == null ? value == entry.getValue() : value.equals(entry.getValue())){
        		return true;
        	};
        }
        return false;
    }
```
上面是修改后的代码，这样的代码看起来是不是清爽很多。但是其实这个写法的效率是很低的。我在注释部分已经详细说明了。

*  看下面的一段代码：还是很有意思的。
```
 public String toString() {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (! i.hasNext())
            return "{}";

        StringBuilder sb = new StringBuilder();
        sb.append('{');
        for (;;) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            // 这个地方和当前对象进行比较是为了防止进入死循环。infinite loop 
            sb.append(key   == this ? "(this Map)" : key);
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value);
            if (! i.hasNext())
                return sb.append('}').toString();
            sb.append(", ");
        }
    }
```

* ConcurrentHashMap是通过热分离的思想，将一块内存通过hash分为多个segment。然后每个segment通过锁（ReentrantLock）保证一致性。

* 保证一致性hash的算法之分离链接法。
```java
package com.itany.separatechainning;  
  
import java.util.LinkedList;  
import java.util.List;  
  
/* 
 *解决冲突的第一个方法时分离链接法 其做法是将散列到同一个值得所有元素保留到一个表中 
 *为执行一次查找 我们通过散列函数来决定究竟遍历哪一个链表，然后我们再在被确定的链表中执行一次查找 
 *散列表存储一个List链表数组  
 */  
public class SeparateChainingHashTable<T>  
{  
    private static final int DEFAULT_TABLE_SIZE=101;  
    //我们定义散列表的填装因子：散列表中元素个数和散列表大小的比  这里是1 即集合链表的平均长度是1  
    private int currentSize;  
    private List<T>[] theLists;  
    public SeparateChainingHashTable()  
    {  
        this(DEFAULT_TABLE_SIZE);  
    }  
    public SeparateChainingHashTable(int size)  
    {  
        //对数组中的List进行初始化  
        theLists=new LinkedList[nextPrime(size)];  
        for(int i=0;i<theLists.length;i++)  
        {  
            theLists[i]=new LinkedList<T>();  
        }  
    }  
    public void makeEmpty()  
    {  
        for (List<T> list : theLists)  
        {  
            list.clear();  
        }  
        currentSize=0;  
    }  
    public void insert(T t)  
    {  
        List<T> whichList=theLists[myHash(t)];  
        if(!contains(t))  
        {  
            whichList.add(t);  
            currentSize++;  
            if(currentSize>theLists.length)  
                reHash();  
        }  
    }  
    public boolean contains(T t)  
    {  
        //通过myHash找出是哪一个集合  
        List<T> whichList=theLists[myHash(t)];  
        return whichList.contains(t);  
    }  
    public void remove(T t)  
    {  
        List<T> whichList=theLists[myHash(t)];  
        if(contains(t))  
        {  
            whichList.remove(t);  
            currentSize--;  
        }  
    }  
    private int myHash(T t)  
    {  
        int hash=t.hashCode();  
        //对每一个t得到数组的序号 大小从0-theLists.length-1 进行分配  
        hash%=theLists.length;  
        //防止hash值为负数  
        if(hash<0)  
            hash+=theLists.length;  
        return hash;  
    }  
    //有一种情况是currentSize>theLists.length 需要对数组进行扩容 即再散列  
    //因为列表装的太满 那么操作时间将会变得更长，且插入操作可能失败  此时的方法时新建另外一个两倍大的表  
    //而且使用一个新的相关的散列函数(因为计算时tableSize变了)，扫描整个原始散列表，计算每一个元素的新散列值   并将它们插入到新表中  
    private void reHash()  
    {  
        List<T>[] oldLists=theLists;//复制一下一会要用    theLists在重新new一个  
        theLists=new LinkedList[nextPrime(2*theLists.length)];  
        for(int i=0;i<theLists.length;i++)  
        {  
            theLists[i]=new LinkedList<T>();  
        }  
        //把原来的元素复制到新的数组中  注意是把集合中的元素复制进去  
        for(int i=0;i<oldLists.length;i++)  
        {  
            for (T t : oldLists[i])  
            {  
                insert(t);  
            }  
        }  
          
    }  
    //表的大小是一个素数 这可以保证一个很好的分布  
    //是否是素数  
    private static boolean isPrime(int num)  
    {  
        int i=1;  
        while((num%(i+1))!=0)  
        {  
            i++;  
        }  
        if(i==num-1)  
        {  
            return true;  
        }  
        else   
        {  
            return false;  
        }  
    }  
      
      
    private static int nextPrime(int num)  
    {  
        while(!isPrime(num))  
        {  
            num++;  
        }  
        return num;  
    }  
     
}
```
* ConcurrentHashMap 实现高并发的总结基于通常情形而优化在实际的应用中，散列表一般的应用场景是：除了少数插入操作和删除操作外，绝大多数都是读取操作，而且读操作在大多数时候都是成功的。正是基于这个前提，ConcurrentHashMap 针对读操作做了大量的优化。通过 HashEntry 对象的不变性和用 volatile 型变量协调线程间的内存可见性，使得 大多数时候，读操作不需要加锁就可以正确获得值。这个特性使得 ConcurrentHashMap 的并发性能在分离锁的基础上又有了近一步的提高。总结 ConcurrentHashMap 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 HashTable 和用同步包装器包装的 HashMap（Collections.synchronizedMap(new HashMap())），ConcurrentHashMap 拥有更高的并发性。在 HashTable 和由同步包装器包装的 HashMap 中，使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。在使用锁来协调多线程间并发访问的模式下，减小对锁的竞争可以有效提高并发性。有两种方式可以减小对锁的竞争：减小请求 同一个锁的 频率。减少持有锁的 时间。ConcurrentHashMap 的高并发性主要来自于三个方面：
1. 用分离锁实现多个线程间的更深层次的共享访问。
2. 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
3. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。使用分离锁，减小了请求 同一个锁的频率。通过 HashEntery 对象的不变性及对同一个 Volatile 变量的读 / 写来协调内存可见性，使得 读操作大多数时候不需要加锁就能成功获取到需要的值。由于散列映射表在实际应用中大多数操作都是成功的 读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。通过减小请求同一个锁的频率和尽量减少持有锁的时间 ，使得 ConcurrentHashMap 的并发性相对于 HashTable 和用同步包装器包装的 HashMap有了质的提高。

```java
package jodd.db.pool;

import jodd.db.DbSqlException;
import jodd.db.connection.ConnectionProvider;
import jodd.log.Logger;
import jodd.log.LoggerFactory;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;

/**
 * A class for pre-allocating, recycling, and managing JDBC connections.
 * <p>
 * It uses threads for opening a new connection. When no connection
 * available it will wait until a connection is released.
 */
public class CoreConnectionPool implements Runnable, ConnectionProvider {

	private static final Logger log = LoggerFactory.getLogger(CoreConnectionPool.class);

	// ---------------------------------------------------------------- properties

	private static final String DEFAULT_VALIDATION_QUERY = "select 1";

	private String driver;
	private String url;
	private String user;
	private String password;
	private int maxConnections = 10;
	private int minConnections = 5;
	private boolean waitIfBusy;
	private boolean validateConnection = true;
	private long validationTimeout = 18000000L;		// 5 hours
	private String validationQuery;

	public String getDriver() {
		return driver;
	}

	/**
	 * Specifies driver class name.
	 */
	public void setDriver(String driver) {
		this.driver = driver;
	}

	public String getUrl() {
		return url;
	}

	/**
	 * Specifies JDBC url.
	 */
	public void setUrl(String url) {
		this.url = url;
	}

	public String getUser() {
		return user;
	}

	/**
	 * Specifies db username.
	 */
	public void setUser(String user) {
		this.user = user;
	}

	public String getPassword() {
		return password;
	}

	/**
	 * Specifies db password.
	 */
	public void setPassword(String password) {
		this.password = password;
	}

	public int getMaxConnections() {
		return maxConnections;
	}

	/**
	 * Sets max number of connections.
	 */
	public void setMaxConnections(int maxConnections) {
		this.maxConnections = maxConnections;
	}

	public int getMinConnections() {
		return minConnections;
	}

	/**
	 * Sets minimum number of open connections.
	 */
	public void setMinConnections(int minConnections) {
		this.minConnections = minConnections;
	}

	public boolean isWaitIfBusy() {
		return waitIfBusy;
	}

	/**
	 * Sets if pool should wait for connection to be freed when none
	 * is available. If wait for busy is <code>false</code>
	 * exception will be thrown when max connection is reached.
	 */
	public void setWaitIfBusy(boolean waitIfBusy) {
		this.waitIfBusy = waitIfBusy;
	}

	public long getValidationTimeout() {
		return validationTimeout;
	}

	/**
	 * Specifies number of milliseconds from connection creation
	 * when connection is considered as opened and valid.
	 */
	public void setValidationTimeout(long validationTimeout) {
		this.validationTimeout = validationTimeout;
	}

	public String getValidationQuery() {
		return validationQuery;
	}

	/**
	 * Specifies query to be used for validating connections.
	 * If set to <code>null</code> validation will be performed
	 * by invoking <code>Connection#isClosed</code> method.
	 */
	public void setValidationQuery(String validationQuery) {
		this.validationQuery = validationQuery;
	}

	/**
	 * Sets default validation query (select 1);
	 */
	public void setDefaultValidationQuery() {
		this.validationQuery = DEFAULT_VALIDATION_QUERY;
	}

	public boolean isValidateConnection() {
		return validateConnection;
	}

	/**
	 * Specifies if connections should be validated before returned.
	 */
	public void setValidateConnection(boolean validateConnection) {
		this.validateConnection = validateConnection;
	}

	// ---------------------------------------------------------------- init

	private ArrayList<ConnectionData> availableConnections, busyConnections;
	private boolean connectionPending;
	private boolean initialised;

	/**
	 * {@inheritDoc}
	 */
	@Override
	public synchronized void init() {
		if (initialised) {
			return;
		}
		if (log.isInfoEnabled()) {
			log.info("Core connection pool initialization");
		}
		try {
			Class.forName(driver);
		}
		catch (ClassNotFoundException cnfex) {
			throw new DbSqlException("Database driver not found: " + driver, cnfex);
		}

		if (minConnections > maxConnections) {
			minConnections = maxConnections;
		}
		availableConnections = new ArrayList<>(maxConnections);
		busyConnections = new ArrayList<>(maxConnections);

		for (int i = 0; i < minConnections; i++) {
			try {
				Connection conn = DriverManager.getConnection(url, user, password); 
				availableConnections.add(new ConnectionData(conn));
			} catch (SQLException sex) {
				throw new DbSqlException("No database connection", sex);
			}
		}
		initialised = true;
	}

	// ---------------------------------------------------------------- get/close

	/**
	 * {@inheritDoc}
	 */
	@Override
	public synchronized Connection getConnection() {
		if (availableConnections == null) {
			throw new DbSqlException("Connection pool is not initialized");
		}
		if (!availableConnections.isEmpty()) {
			int lastIndex = availableConnections.size() - 1;
			ConnectionData existingConnection = availableConnections.get(lastIndex);
			availableConnections.remove(lastIndex);
			
			// If conn on available list is closed (e.g., it timed out), then remove it from available list
			// and repeat the process of obtaining a conn. Also wake up threads that were waiting for a
			// conn because maxConnection limit was reached.
			long now = System.currentTimeMillis();
			boolean isValid = isConnectionValid(existingConnection, now);
			if (!isValid) {
				if (log.isDebugEnabled()) {
					log.debug("Pooled connection not valid, resetting");
				}

				notifyAll();				 // freed up a spot for anybody waiting
				return getConnection();
			} else {
				if (log.isDebugEnabled()) {
					log.debug("Returning valid pooled connection");
				}

				busyConnections.add(existingConnection);
				existingConnection.lastUsed = now;
				return existingConnection.connection;
			}
		}
		if (log.isDebugEnabled()) {
			log.debug("No more available connections");
		}

		// no available connections
		if (((availableConnections.size() + busyConnections.size()) < maxConnections) && !connectionPending) {
			makeBackgroundConnection();
		} else if (!waitIfBusy) {
			throw new DbSqlException("Connection limit reached: " + maxConnections);
		}
		// wait for either a new conn to be established (if you called makeBackgroundConnection) or for
		// an existing conn to be freed up.
		try {
			wait();
		} catch (InterruptedException ie) {
			// ignore
		}
		// someone freed up a conn, so try again.
		return getConnection();
	}

	/**
	 * Checks if existing connection is valid and available. It may happens
	 * that if connection is not used for a while it becomes inactive,
	 * although not technically closed.
	 */
	private boolean isConnectionValid(ConnectionData connectionData, long now) {
		if (!validateConnection) {
			return true;
		}
		
		if (now < connectionData.lastUsed + validationTimeout) {
			return true;
		}

		Connection conn = connectionData.connection;

		if (validationQuery == null) {
			try {
				return !conn.isClosed();
			} catch (SQLException sex) {
				return false;
			}
		}
		
		boolean valid = true;
		Statement st = null;
		try {
			st = conn.createStatement();
			st.execute(validationQuery);
		} catch (SQLException sex) {
			valid = false;
		} finally {
			if (st != null) {
				try {
					st.close();
				} catch (SQLException ignore) {
				}
			}
		}
		return valid;
	}

	/**
	 * You can't just make a new conn in the foreground when none are
	 * available, since this can take several seconds with a slow network
	 * conn. Instead, start a thread that establishes a new conn,
	 * then wait. You get woken up either when the new conn is established
	 * or if someone finishes with an existing conn.
	 */
	private void makeBackgroundConnection() {
		connectionPending = true;
		Thread connectThread = new Thread(this);
		connectThread.start();
	}

	public void run() {
		try {
			Connection connection = DriverManager.getConnection(url, user, password);
			synchronized(this) {
				availableConnections.add(new ConnectionData(connection));
				connectionPending = false;
				notifyAll();
			}
		} catch (Exception ex) {
			// give up on new conn and wait for existing one to free up.
		}
	}

	@Override
	public synchronized void closeConnection(Connection connection) {
		ConnectionData connectionData = new ConnectionData(connection);
		busyConnections.remove(connectionData);
		availableConnections.add(connectionData);
		notifyAll();		// wake up threads that are waiting for a conn
	}


	// ---------------------------------------------------------------- close

	/**
	 * Close all the connections. Use with caution: be sure no connections are in
	 * use before calling. Note that you are not <i>required</i> to call this
	 * when done with a ConnectionPool, since connections are guaranteed to be
	 * closed when garbage collected. But this method gives more control
	 * regarding when the connections are closed.
	 */
	@Override
	public synchronized void close() {
		if (log.isInfoEnabled()) {
			log.info("Core connection pool shutdown");
		}
		closeConnections(availableConnections);
		availableConnections = new ArrayList<>(maxConnections);
		closeConnections(busyConnections);
		busyConnections = new ArrayList<>(maxConnections);
	}

	private void closeConnections(ArrayList<ConnectionData> connections) {
		try {
			for (ConnectionData connectionData : connections) {
				Connection connection = connectionData.connection;
				if (!connection.isClosed()) {
					connection.close();
				}
			}
		} catch (SQLException sex) {
			// Ignore errors; garbage collect anyhow
		}
	}

	// ---------------------------------------------------------------- conn data

	/**
	 * Connection data with last used timestamp.
	 */
	class ConnectionData {
		final Connection connection;
		long lastUsed;

		ConnectionData(Connection connection) {
			this.connection = connection;
			this.lastUsed = System.currentTimeMillis();
		}

		@Override
		public boolean equals(Object o) {
			if (this == o) {
				return true;
			}
			if (o == null || getClass() != o.getClass()) {
				return false;
			}
			ConnectionData that = (ConnectionData) o;
			return connection.equals(that.connection);
		}

		@Override
		public int hashCode() {
			return connection.hashCode();
		}
	}

	// ---------------------------------------------------------------- stats

	/**
	 * Returns connection stats.
	 */
	public synchronized SizeSnapshot getConnectionsCount() {
		return new SizeSnapshot(availableConnections.size(), busyConnections.size());
	}

	/**
	 * Just a statistic class.
	 */
	public static class SizeSnapshot {
		final int totalCount;
		final int availableCount;
		final int busyCount;

		SizeSnapshot(int availableCount, int busyCount) {
			this.totalCount = availableCount + busyCount;
			this.availableCount = availableCount;
			this.busyCount = busyCount;
		}

		/**
		 * Returns total number of connections.
		 */
		public int getTotalCount() {
			return totalCount;
		}

		/**
		 * Returns number of available connections.
		 */
		public int getAvailableCount() {
			return availableCount;
		}

		/**
		 * Returns number of busy connections.
		 */
		public int getBusyCount() {
			return busyCount;
		}

		@Override
		public String toString() {
			return "Connections count: {total=" + totalCount +
					", available=" + availableCount +
					", busy=" + busyCount + '}';
		}
	}

}
```
