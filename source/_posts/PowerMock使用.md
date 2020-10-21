---
title: PowerMock使用
date: 2020-10-20 11:35:13
category: code
tags: 
    - mock
toc: true
---

## 简介

​	可用于本地单元测试的框架。可模拟公有，私有方法返回数据；也可模拟静态属性数据，实例属性数据；模拟Spring Bean 实例及注入等。

<!-- more -->

## 与Mockito的区别

PowerMock较Mockito强大一点，可以模拟私有方法，静态方法，final 类。



## maven依赖

```
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito2</artifactId>
            <scope>test</scope>
        </dependency>
```



## 说明

### @Mock 和 @Spy 区别

@Mock：模拟出来的对象，其所有方法是空的，调用都返回空，真实的方法不会执行。

@Spy：模拟出来的对象，其所有方法都是真实的，调用都按真实方法去调用，因此很可能依赖真实的环境。除非mock 对应的方法，才会覆盖。

### doReturn()...when() 和when()...thenReturn() 的区别

mock 出来的对象： 没有区别。

spy出来的对象：doReturn()...when() 和when()...thenReturn()  是有区别的









## 使用

### 模拟公有方法

1.	设置Runner环境为`PowerMockRunner`

```java
// FoodMenuServiceTest.class ...

@RunWith(PowerMockRunner.class)
public class FoodMenuServiceTest{
	// ...
}
```



2.	定义Mock属性和需要注入的属性。

```java
	// FoodMenuServiceTest.class ...

	// 需要注入（Mock的属性）的属性,此处需要new具体类
 	@InjectMocks
    private FoodMenuService foodMenuService = new FoodMenuServiceImpl();

	// Mock 的属性
    @Mock
    private MaterialService materialService;

	// ...
```



3.	构造数据，模拟方法，运行

```java
   	// FoodMenuServiceTest.class ...

	@Test
    public void mockPublicMethod() {

        // 数据
        Material fish = Material.builder().name("鱼").build();
        Material crayFish = Material.builder().name("小龙虾").build();
        ArrayList<Material> materials = CollUtil.newArrayList(fish, crayFish);

        // 模拟方法 list
        PowerMockito.when(materialService.list()).thenReturn(materials);

        // 调用
        List<Food> list = foodMenuService.list();

        // 打印结果
        // [Food(name=红烧鱼), Food(name=水煮鱼), Food(name=麻辣小龙虾), Food(name=清蒸小龙虾)]
		System.out.println(list);	
    }

	// ...
```



### 模拟私有方法

1.	设置runner环境为PowerMockRunner,并设置PrepareForTest为需要测试的类PrepareForTest

```java
// MockPrivateMethodTest.class
@RunWith(PowerMockRunner.class)
@PrepareForTest(MaterialServiceImpl.class)
public class MockPrivateMethodTest {
    // ...
}
```



2.	定义需要Spy的属性

```java
// MockPrivateMethodTest.class
	@Spy
    private MaterialService materialService = new MaterialServiceImpl();
```



3.	构造数据，模拟方法，运行

```java
 // MockPrivateMethodTest.class
	@Test
    public void mockPrivateMethod() throws Exception {

        // 数据
        Material fish = Material.builder().name("鱼").build();
        Material crayFish = Material.builder().name("小龙虾").build();
        ArrayList<Material> materials = CollUtil.newArrayList(fish, crayFish);

        // 模拟方法
        // 区别于 PowerMockito.when(materialService, "pri").thenReturn(materials);
        PowerMockito.doReturn(materials).when(materialService, "pri");
        
        // 执行
        List<Material> list = materialService.listForPri();

        // 打印
        // [Material(name=鱼), Material(name=小龙虾)]
        System.out.println(list);

    }
```



### 同时模拟共有和私有方法，并注入

1.	设置Runner和PrepareForTest

```java
// MockPublicAndPrivateMethodTest.class
@RunWith(PowerMockRunner.class)
@PrepareForTest(MaterialServiceImpl.class)
public class MockPublicAndPrivateMethodTest {
    // ...
}
```

2.	定义属性

```java
    @InjectMocks
    private FoodMenuService foodMenuService = new FoodMenuServiceImpl();

	// 因为要调用 MaterialService.listForPubAndPri的实际方法，顾使用 @Spy，如不需可使用@Mock
    @Spy
    private MaterialService materialService = new MaterialServiceImpl();
```

3.	构造数据，模拟方法，运行

```java
	@Test
    public void mockPublicAndPrivateMethod() throws Exception {

        // 构造数据
        Material fish = Material.builder().name("鱼").build();
        Material crayFish = Material.builder().name("小龙虾").build();
        ArrayList<Material> materialsA = CollUtil.newArrayList(fish);
        ArrayList<Material> materialsB = CollUtil.newArrayList(crayFish);

        // 模拟公有方法
        // 此处用方法名的形式是因为 spy出来的对象具有真实方法，以方法名的方式调用不会执行真实方法。
        PowerMockito.doReturn(materialsA).when(materialService, "list");

        // 模拟私有方法
        PowerMockito.doReturn(materialsB).when(materialService, "pri");

        // 运行
        List<Food> list = foodMenuService.listForPubAndPri();

        // 打印
        // [Food(name=红烧鱼), Food(name=水煮鱼), Food(name=麻辣小龙虾), Food(name=清蒸小龙虾)]
        System.out.println(list);

    }
```

### 模拟静态方法

```java
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

@RunWith(PowerMockRunner.class)
@PrepareForTest(StaticMethodAndField.class)
public class StaticMethodAndFieldTest {

    // 期望数据
    public static final String expect = "127.0.0.1";

    @Test
    public void mockStaticMethod() {

        // 模拟方法
        PowerMockito.mockStatic(StaticMethodAndField.class);
        PowerMockito.when(StaticMethodAndField.getIp()).thenReturn(expect);

        // 调用
        String ip = StaticMethodAndField.getIp();

        // 断言
        Assert.assertEquals(expect, ip);

    }
}
```

### 模拟静态私有属性

```java

@RunWith(PowerMockRunner.class)
@PrepareForTest(StaticMethodAndField.class)
public class StaticMethodAndFieldTest {

    // 期望数据
    public static final String expect = "127.0.0.1";
    
    /**
     * 参考 https://qa.1r1g.com/sf/ask/1621376431/
     * @throws NoSuchFieldException
     * @throws IllegalAccessException
     */
    @Test
    public void mockFinalField() throws NoSuchFieldException, IllegalAccessException {

        // 模拟属性
        
        
        // 获取字段
        Field field = Whitebox.getField(StaticMethodAndField.class, "IP");

        // 设置成public
        field.setAccessible(true);

        // 获取 修饰器
        Field modifiersField = Field.class.getDeclaredField("modifiers");
        // 设置为可使用的
        modifiersField.setAccessible(true);

        // 消除 FINAL 修饰器
        modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

        // 设置新值
        field.set(null, expect);

        // 断言
        Assert.assertEquals(expect, StaticMethodAndField.IP);
    }
}
```



### 模拟枚举类

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import org.powermock.reflect.Whitebox;

import static org.junit.Assert.*;

@RunWith(PowerMockRunner.class)
@PrepareForTest(Colors.class)
public class ColorsTest {


    @Test
    public void mockEnums() {
        // 期望的数据
        String expect = "模拟的蓝色";

        // 模拟的实例
        Colors instance = PowerMockito.mock(Colors.class);

        // 替换实例
        Whitebox.setInternalState(Colors.class, "BLUE", instance);

        // 模拟方法
        PowerMockito.when(instance.getName() ).thenReturn(expect);

        // 断言
        assertEquals(expect, Colors.BLUE.getName());
    }
}
```



## 项目源码

```
https://github.com/iaoongin/summary-project/sp-powermock
```

