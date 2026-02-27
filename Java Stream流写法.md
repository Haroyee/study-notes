# Java Stream流写法

## 1、一般用法

```java
Integer nums[] = new Integer[]{1,2,3,4,5,6,7};
List<Integer> numList = Arrays.stream(nums).collect(Collectors.toList());
System.out.println(numList);
```

```java
Integer nums1[] = new Integer[]{1,2,3,4,5,6,7};
List<Integer> nums2 = new ArrayList<>(Arrays.stream(nums1).collect(Collectors.toList()));
List<Integer> numList = nums2.stream().collect(Collectors.toList());
System.out.println(numList);
```

输出结果均为：

```
[1, 2, 3, 4, 5, 6, 7]
```

## 2、获取指定元素

```java
#初始化类
List<User> users = new ArrayList<>();
User user =new User();
for(int i = 0;i <= 7;i++) {
    user.setId(i);
    users.add(user);
}
#stream流
List<Integer> numList = users.stream().map(u->u.getId()).collect(Collectors.toList());
System.out.println(numList);
```

输出

```
[0, 1, 2, 3, 4, 5, 6, 7]
```

其他类型

map

```java
List<User> users = new ArrayList<>();

        for(int i = 0;i < 7;i++) {
            User user =new User();
            user.setId(i);
            user.setUsername("user_"+i);
            users.add(user);
        }
        Map<String,Integer> idMap = users.stream().collect(Collectors.toMap(i -> i.getUsername(), i -> i.getId()));
        System.out.println(idMap);
```

```java
Set<Integer> set = nums.stream().map(i -> i.getId()).collect(Collectors.toSet());
```

## 3、收集为自定义对象（以LinkedList为例）

```java
LinkedList<Integer> linkedList = nums.stream().map(i -> i.getId()).collect(Collectors.toCollection(LinkedList::new));
```

## 4、调用构造函数用法

```

```

