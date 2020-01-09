shiro缓存
===
* 什么是shiro缓存
    * shiro中提供了对认证信息和授权信息的缓存。
    * 默认是关闭认证信息缓存的，对于授权信息的缓存shiro默认开启的(因为授权的数据量大)
* AuthenticatingRealm 及 AuthorizingRealm 分别提供了对AuthenticationInfo 和 AuthorizationInfo 信息的缓存。
* 使用Map存储数据
```java
package org.apache.shiro.cache;

import java.util.Collection;
import java.util.Set;

public interface Cache<K, V> {
    V get(K var1) throws CacheException;

    V put(K var1, V var2) throws CacheException;

    V remove(K var1) throws CacheException;

    void clear() throws CacheException;

    int size();

    Set<K> keys();

    Collection<V> values();
}
```
```java
package org.apache.shiro.cache;

import java.util.Collection;
import java.util.Collections;
import java.util.Map;
import java.util.Set;

public class MapCache<K, V> implements Cache<K, V> {
    private final Map<K, V> map;
    private final String name;

    public MapCache(String name, Map<K, V> backingMap) {
        if (name == null) {
            throw new IllegalArgumentException("Cache name cannot be null.");
        } else if (backingMap == null) {
            throw new IllegalArgumentException("Backing map cannot be null.");
        } else {
            this.name = name;
            this.map = backingMap;
        }
    }

    public V get(K key) throws CacheException {
        return this.map.get(key);
    }

    public V put(K key, V value) throws CacheException {
        return this.map.put(key, value);
    }

    public V remove(K key) throws CacheException {
        return this.map.remove(key);
    }

    public void clear() throws CacheException {
        this.map.clear();
    }

    public int size() {
        return this.map.size();
    }

    public Set<K> keys() {
        Set<K> keys = this.map.keySet();
        return !keys.isEmpty() ? Collections.unmodifiableSet(keys) : Collections.emptySet();
    }

    public Collection<V> values() {
        Collection<V> values = this.map.values();
        return (Collection)(!this.map.isEmpty() ? Collections.unmodifiableCollection(values) : Collections.emptySet());
    }

    public String toString() {
        return "MapCache '" + this.name + "' (" + this.map.size() + " entries)";
    }
}
```
