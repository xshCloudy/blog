---
title: Arrays.asList坑点记录
catalog: true
date: 2019-01-17 13:43:51
subtitle: "Arrays.asList坑点记录"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
## Arrays.asList坑点记录

### 坑点
```javascript
   public static void main( String[] args ){
        Integer [] a = {0,1,2,3};
        List<Integer> integers = Arrays.asList(a);
        integers.add(4);
    }
```
上述代码执行后结果：
```javascript
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.remove(AbstractList.java:161)
	at com.xinji.directpaymentclient.web.service.ArraysTest.main(ArraysTest.java:16

```
### 分析原因
进入Arrays.asList方法：
```javascript
 public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }

```
发现返回的ArrayList 是Arrays的内部类，并不是java.util包下的ArrayList 。
Arrays里面的ArrayList并没有重写父类AbstractList 中的add，remove方法。
而父类AbstractList中的add，remove方法是
```javascript
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }
```
所以才会报UnsupportedOperationException错误。

### 解决办法

如果不需要改变list长度，可以使用Arrays.asList.
但如果要改变长度就如下处理转成java.util包下的ArrayList 
```javascript
  public static void main( String[] args ){
        Integer [] a = {0,1,2,3,4,5};
        List<Integer> ints = new ArrayList<Integer>(Arrays.asList(a));
        ints.add(6);
        System.out.println(ints.size());

    }

```



