---
title: 使用JSON.Stringify
tags: [javascript,json]
date: 2015/02/01
---

加入有一个对象具有参数"prop1", "prop2", "prop3"。 我们可以通过传递 附加参数 给 JSON.stringify 来选择性将参数生成字符串，像这样：

```
var obj = {
    'prop1': 'value1',
    'prop2': 'value2',
    'prop3': 'value3'
};

var selectedProperties = ['prop1', 'prop2'];

var str = JSON.stringify(obj, selectedProperties);

// str
// {"prop1":"value1","prop2":"value2"}
```

"str" 降至包含选择的参数。
除了传递数组，我们也可以传递函数。

```
function selectedProperties(key, val) {
    // the first val will be the entire object, key is empty string
    if (!key) {
        return val;
    }

    if (key === 'prop1' || key === 'prop2') {
        return val;
    }

    return;
}
```

最后一个参数，可以修改生成字符串的方式。

```
var str = JSON.stringify(obj, selectedProperties, '\t\t');

/* str output with double tabs in every line.
{
        "prop1": "value1",
        "prop2": "value2"
}
*/
```

[转载自github:loverajoel/jstips](https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-02-09-using-json-stringify.md);
