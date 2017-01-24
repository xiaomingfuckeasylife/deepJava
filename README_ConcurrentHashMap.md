## ConcurrentHashMap源码分析

### 讨论点

**  这个是我在看AbstractMap的源码片段发现了大量的这样的类似的代码
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

**  看下面的一段代码：还是很有意思的。
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

