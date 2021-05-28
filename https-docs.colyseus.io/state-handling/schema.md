---
description: State Handling » Schema　状态处理»模式
---

# Schema

从Colyseus 0.10开始引入`SchemaSerializer`，它是默认的序列化方法。

`Schema`结构只能用于房间的状态（可同步数据）。 您不需要将 `Schema` 及其其他结构用于不可同步算法中的数据。

## 服务器端 <a id="server-side"></a>

要使用`SchemaSerializer`，您必须：

* 有一个扩展Schema类的状态类
* 用`@type()`装饰器注释所有可同步化的属性
* 实例化您房间的状态（`this.setState(new MyState())`）

> Are you not using TypeScript?
>
> Decorators[还不是ECMAScript的一部分](https://github.com/tc39/proposal-decorators)，因此`type`普通JavaScript上的语法仍然有点奇怪，您可以在每个片段的“ JavaScript”标签中看到该语法。

```text
import { Schema, type } from "@colyseus/schema";

class MyState extends Schema {
    @type("string")
    currentTurn: string;
}
```

### Primitive types <a id="primitive-types"></a>

这些是您可以为`@type()`装饰器提供的类型及其限制。

> 如果您确切知道`number`属性的范围，则可以通过为其提供正确的原始类型来优化序列化。
>
> 否则，请使用`"number"`，它将在序列化过程中添加一个额外的字节来标识自身。

| 类型 | 描述 | 局限性 |
| :--- | :--- | :--- |
| `"string"` | utf8字符串 | 的最大字节大小 `4294967295` |
| `"number"` | 自动检测要使用的`int`或`float`类型。（在输出上添加一个额外的字节） | `0` 至 `18446744073709551615` |
| `"boolean"` | `true` 或者 `false` | `0` 或者 `1` |
| `"int8"` | 有符号的8位整数 | `-128` 至 `127` |
| `"uint8"` | 无符号8位整数 | `0` 至 `255` |
| `"int16"` | 有符号的16位整数 | `-32768` 至 `32767` |
| `"uint16"` | 无符号16位整数 | `0` 至 `65535` |
| `"int32"` | 有符号的32位整数 | `-2147483648` 至 `2147483647` |
| `"uint32"` | 无符号32位整数 | `0` 至 `4294967295` |
| `"int64"` | 有符号的64位整数 | `-9223372036854775808` 至 `9223372036854775807` |
| `"uint64"` | 无符号64位整数 | `0` 至 `18446744073709551615` |
| `"float32"` | 单精度浮点数 | `-3.40282347e+38` 至 `3.40282347e+38` |
| `"float64"` | 双精度浮点数 | `-1.7976931348623157e+308` 至 `1.7976931348623157e+308` |

### 子架构属性 <a id="child-schema-properties"></a>

您可以在“root”状态定义中定义更多自定义数据类型，作为直接引用，映射或数组。

```text
import { Schema, type } from "@colyseus/schema";

class World extends Schema {
    @type("number")
    width: number;

    @type("number")
    height: number;

    @type("number")
    items: number = 10;
}

class MyState extends Schema {
    @type(World)
    world: World = new World();
}
```

### 数组模式 <a id="arrayschema"></a>

`ArraySchema`是内置的JavaScript synchronizeable版本[数组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)类型。

> 您可以从数组中使用更多方法。[看一下“ MDN数组文档”](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/)。

**示例：自定义`Schema`类型的数组**

```text
import { Schema, ArraySchema, type } from "@colyseus/schema";

class Block extends Schema {
    @type("number")
    x: number;

    @type("number")
    y: number;
}

class MyState extends Schema {
    @type([ Block ])
    blocks = new ArraySchema<Block>();
}
```

**示例：基本类型的数组**

您不能在数组内混合类型。

```text
import { Schema, ArraySchema, type } from "@colyseus/schema";

class MyState extends Schema {
    @type([ "string" ])
    animals = new ArraySchema<string>();
}
```

**array.push\(\)**

在数组的末尾添加一个或多个元素，并返回该数组的新长度。

```text
const animals = new ArraySchema<string>();
animals.push("pigs", "goats");
animals.push("sheeps");
animals.push("cows");
// output: 4
```

**array.pop\(\)**

从数组中删除最后一个元素，然后返回该元素。此方法更改数组的长度。

```text
animals.pop();
// output: "cows"

animals.length
// output: 3
```

**array.shift\(\)**

从数组中删除第一个元素，并返回该删除的元素。此方法更改数组的长度。

```text
animals.shift();
// output: "pigs"

animals.length
// output: 2
```

**array.unshift\(\)¶**

将一个或多个元素添加到数组的开头，并返回数组的新长度。

```text
animals.unshift("pigeon");
// output: 3
```

**array.indexOf\(\)¶**

返回可以在数组中找到给定元素的第一个索引；如果不存在，则返回-1

```text
const itemIndex = animals.indexOf("sheeps");
```

**array.splice\(\)¶**

通过删除或替换现有元素和/或[在适当位置](https://en.wikipedia.org/wiki/In-place_algorithm)添加新元素[来](https://en.wikipedia.org/wiki/In-place_algorithm)更改数组的内容。

```text
// find the index of the item you'd like to remove
const itemIndex = animals.findIndex((animal) => animal === "sheeps");

// remove it!
animals.splice(itemIndex, 1);
```

**array.forEach\(\)¶**

遍历数组的每个元素。

```text
this.state.array1 = new ArraySchema<string>('a', 'b', 'c');

this.state.array1.forEach(element => {
    console.log(element);
});
// output: "a"
// output: "b"
// output: "c"
```

### MapSchema[¶](https://docs.colyseus.io/state/schema/#mapschema) <a id="mapschema"></a>

`MapSchema`是内置的JavaScript synchronizeable版本的[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)类型。

建议使用Map按ID跟踪您的游戏实体，例如玩家，敌人等。

> 目前仅支持字符串键
>
> 当前，`MapSchema`唯一允许您提供值类型。密钥类型始终为`string`。

```text
import { Schema, MapSchema, type } from "@colyseus/schema";

class Player extends Schema {
    @type("number")
    x: number;

    @type("number")
    y: number;
}

class MyState extends Schema {
    @type({ map: Player })
    players = new MapSchema<Player>();
}
```

**map.get\(\)¶**

通过其键获取地图项：

```text
const map = new MapSchema<string>();
const item = map.get("key");
```

或者

```text
//
// NOT RECOMMENDED
//
// This is a compatibility layer with previous versions of @colyseus/schema
// This is going to be deprecated in the future.
//
const item = map["key"];
```

**map.set\(\)¶**

通过按键设置地图项：

```text
const map = new MapSchema<string>();
map.set("key", "value");
```

或者

```text
//
// NOT RECOMMENDED
//
// This is a compatibility layer with previous versions of @colyseus/schema
// This is going to be deprecated in the future.
//
map["key"] = "value";
```

**map.delete\(\)¶**

通过键删除地图项：

```text
map.delete("key");
```

或者

```text
//
// NOT RECOMMENDED
//
// This is a compatibility layer with previous versions of @colyseus/schema
// This is going to be deprecated in the future.
//
delete map["key"];
```

**map.size¶**

返回`MapSchema`对象中元素的数量。

```text
const map = new MapSchema<number>();
map.set("one", 1);
map.set("two", 2);

console.log(map.size);
// output: 2
```

**map.forEach\(\)¶**

按照插入顺序遍历映射的每个键/值对。打字稿

```text
this.state.players.forEach((value, key) => {
    console.log("key =>", key)
    console.log("value =>", value)
});
```

> 所有地图方法
>
> 您可以从“地图”中使用更多方法。[看一下MDN Maps文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/)。

### 集合模式[¶](https://docs.colyseus.io/state/schema/#collectionschema) <a id="collectionschema"></a>

