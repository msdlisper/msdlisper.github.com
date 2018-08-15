---
layout: post
title:  "immutable的一些总结"
date:   2018-08-15 21:47:58
categories: learn 
tags:  react redux
---

* content
{:toc}

为什么要用Immutable? 如何搭配redux使用Immutable? 



### 是否值得使用

__值得__

1. 安全: 是指使用上的安全，所有删改操作都是增量的, 避免出现该render时没render, 不该render时, 又render了
``` js
function updateNestedState(state, action) {
    let nestedState = state.nestedState;
    // ERROR: this directly modifies the existing object reference - don't do this!
    nestedState.nestedField = action.data;
​    
    // you can do this:
    return {
        ...state,
        nestedState
    };
}

// 不该render时, 却render了? 在redux里看mapStateToProps里的Props的引用是否改变了, 有可能你内容一模一样, 但引用变了, 还是会render
// 研究rendux的shouldComponentUpdate, connect执行时机
```
2. rich api: 
Maps, Lists, Sets, Records, etc. sort, filter, and group the data, reverse it, flatten it
3. 性能提升: [cleverly sharing data structures](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2#.z1g1ofrsi)避免不必要的re-render, 数据越大越好, 小体量的数据会吃亏
4. 美观: 避免过多的使用析构, 造成阅读困难
``` javascript
// 对比 nested Object:
function updateVeryNestedField(state, action) {
    return {
        ...state,
        first : {
            ...state.first,
            second : {
                ...state.first.second,
                [action.someId] : {
                    ...state.first.second[action.someId],
                    fourth : action.someValue
                }
            }
        }
    }
}

function updateVeryNestedField(state, action) {
    return state.setIn(['first', 'second', action.someId, 'fourth'], actoin.someValue);
}

// 对比 更新数组
function insertItem(array, action) {
    let newArray = array.slice();
    newArray.splice(action.index, 0, action.item);
    return newArray;
}

function insertItem(List, action) {
    return List.update(0, action.item);
}

```

__不值得__

1. 破坏了Js的语法, 可能会感觉难用, 比如不能析构
2. Once used, Immutable.JS will spread throughout your codebase(如果爱, 请深爱)
3. 新api的学习成本, 比如读一个数据: myImmutableMap.getIn([‘prop1’, ‘prop2’, ‘prop3’])
4. 如果toJS()使用不当, 性能会反降


### 使用技巧
[完整的doc](http://facebook.github.io/immutable-js/docs/)

__常用的数据结构__

- List, 对应到[]
- Map, 对应到{}
- Set, hashCode不能重复

__常用api:__ 

- fromJS(): 将一个js数据转换成immutable, Object=>Map, Array=>List
    + 用法: fromJS({a: '1', b: '3'})
- toJS(): 将Immutable数据转成JS类型
    + 用法: value.toJS()
- is(): 比较两个immutable数据的hashCode
    + is(map1, map2)
- get(), getIn()
- set(key, value), setIn(keyPath, value)
- update() 更新List的某个值, 比如:[{name:'one'}, {name: 'two'}, {name: 'three'}], 想更新name='three'的这个对象
    + update(index, updater)
    ``` javascript
    list = list.update(
      list.findIndex(function(item) { 
        return item.get("name") === "third"; 
      }), function(item) {
        return item.set("count", 4);
      }
    ); 
    ```


### 结合redux, 比较好的实践

结合[redux的官方指南](https://redux.js.org/faq/immutabledata)

-  Never use toJS() in mapStateToProps
    ``` javascript
    // 千万别这样....
    function mapStateToProps(state) {
      return {
        todos: state.get('todos').toJS() // Always a new object
      }
    }
    ``` 
    + toJS()本身非常耗性能, 这是次要的, 主要是toJS()返回的是new object, 那只要state tree有变化, 这个组件不管state是否改变了, 都会更新(unnecessary re-render)
-  Never mix plain JavaScript objects with Immutable.JS
    +  避免你 myImmutableData.get('first').attr = 'another attr', 然后会引起: 不必要的render; 不可变的数据莫名奇妙的改变了...;
-  Make your entire Redux state tree an Immutable.JS object
    +  use redux-immutable, 提供的一个combineReducers
-  Use Immutable.JS everywhere except your dumb components
    +  纯函数组件不用Immutable, 保证纯洁
-  不管是否使用Immutable, 都区分下domain data, app data, ui data,别把ui data放在store里, 而且尽可能的缩小store的体量, 只放需要共享的数据

### 调试插件

immutable.js 和普通js长的不太像, 所以在调试是会比较麻烦, 可以结合这个插件用, 调试就轻松很多

[官方插件](https://github.com/mattzeunert/immutable-object-formatter-extension)


