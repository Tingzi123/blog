title: java-map遍历
---

Map集合：即 接口Map<K,V> 

map集合的两种取出方式：
    1.Set<k> keyset: 将map中所有的键存入到set集合（即将所有的key值存入到set中）， 因为Set具备迭代器，可以进行迭代遍历。 所有可以迭代方式取出所有的链，再根据get方法。获取每一个键对应的值。

        Map 集合的取出原理： 将map集合转成set集合。 再通过迭代器取出
    2. set<Map.Entry<k,v>>  entrySet: 将map集合中的映射关系（即键值对的方式存入到set中）存入到set集合中,而这个关系的数据类型就是：map.entry