# ORM必看章节

此章节非常对于ORM的学习和使用非常重要，遇到使用问题请先确保有认真看完和排查后再进行提问和反馈。

## 设计思想

ORM全称：object relational mapping，目的是想像操作对象一样操作数据库，是符合面向对象开发思想的。

如将一条数据的插入，映射成一个对象的实例化。

```php
$user = UserModel::create();
$user->data([
    'attr' => 'value'
]);
$user->save();
```

## 常见问题汇总

### 重复使用Model对象

::: tip
一个对象映射一条数据，此时会有很多用习惯了db封装组件的小伙伴把Model当成了db封装使用，重复调用一个Model对象，如下
:::

```php
// 错误使用

// 假设id自增
$user = UserModel::create();

// 插入一条新用户
$user->data([
    'attr' => 'value'
]);
$user->save();

// 插入第二条新用户，此时由于重复调用同一个对象，产生报错，自增id主键重复
$user->data([
    'attr' => 'value2'
]);
$user->save();
```

### ORM生成复杂sql

1.ORM是基于 `mysqli 2.x`组件完成，内部引用 mysqli中的 `QueryBuilder类`完成sql构造，并且在`查询`章节注明了闭包函数使用方式（可直接使用mysqli中的绝大部分连贯操作，如 having 等特殊条件）

2.如果mysqli的连贯操作也无法满足，你有以下几种方式解决该问题
 - 使用自定义sql执行
 - 尝试给组件贡献代码，新增功能特性
 - 提出反馈，在精力允许和大众所需时，我们会维护组件升级


### 优雅删除数据

在ORM设计思想中，对数据的操作映射为对对象的操作，如果按照此原则，那么需要我们先查询出对象，然后再调用对象的`destroy()方法`进行删除。

但是对于执行效率的消耗来说，此次查询在部分业务场景下是无用的。

那么我们到底是否需要遵守设计原则？一般情况下是在`操作前需要校验数据是否存在`时遵守，无需校验则直接传参删除条件即可。

设计原则代表思想，在某些场景下遵守它需要付出一定代价，自己根据喜好去决定它。

```php
$user = User::create()->get($param['id']);

if (!$user){
    return '操作数据不存在，请检查再试';
}
$res = $user->destroy();
```