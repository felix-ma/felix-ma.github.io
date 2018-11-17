---
layout:     post
title:      "ThingsBoard规则引擎解析多层数据"
subtitle:   "目前TB的遥测数据只能存储key:value格式的数据，如果采集的数据是多层级的json对象，那么可以使用规则引擎进行解析成key:value格式，代码逻辑在上个博客上写过了"
date:       2018-11-17 19:05:41
author:     "felix"
header-img: "img/in-post/thingsboard/head.png"
catalog:    true
multilingual: false
tags:
    - ThingsBoard
    - RuleEngine
---

# 规则引擎说明



推荐先去官网学习一下基础的规则引擎。

https://thingsboard.io/docs/user-guide/rule-engine-2-0/re-getting-started/

了解怎么过滤信息，怎么做数据转换就行。

# 思路说明

1. 定义一个key值用于传递json对象，比如说attributesJsonData。
2. 规则引擎中判断遥测数据中是否存在一个约定好的key。
3. 然后将这个key下的所有属性转换成要存储的属性。

整体解决思路是这样的，需要解决一下这几个问题

1. 如果有多个json对象怎么区分。
2. 如果json对象不支持直接存储怎么转换。

## 区分不同的json对象

再次约定一个值，比如说是attributesJsonType。

然后取到这个值之后遍历attributesJsonData里面的json对象，把key值全部改为（type+原有key）

## 多个json转换

在ThingsBoard中的json解析器中判断了传入的值是否是key:value格式。

![1542453888775](img\in-post\thingsboard\ruleengine\1542453888775.png)

如果不是则数据无法存储，所以需要进行转换。

转换格式见上个博客[Js将Json对象解析成key:value格式][1]

[1]: http://felix-ma-blog.cf/2018/11/16/JsconvertKV/	"Js将Json对象解析成key:value格式"

# 规则链

总览图

![1542454061831](img\in-post\thingsboard\ruleengine\1542454061831.png)

规则链js

```
var data, type;
var keyName = "@Name"
if (msg.timeseriesJsonData) {
    var result = {};
    if (isJson(msg.timeseriesJsonData)) {
        convertKV(msg.timeseriesJsonData, "", keyName,
            result);
    } else {
        convertKV(JSON.parse(msg.timeseriesJsonData), "",
            keyName,
            result);
    }
    data = result;
}
if (msg.timeseriesJsonType) {
    type = msg.timeseriesJsonType;
    for (var k in data) {
        data[type + '_' + k] = data[k];
        delete data[k];
    }
}
if (msg.ErrorCode) {
    data.ErrorCode = msg.ErrorCode;
}
if (msg.ErrorMessage) {
    data.ErrorMessage = msg.ErrorMessage;
}
return {
    msg: data,
    metadata: metadata,
    msgType: msgType
};

function convertKV(jsonObj, parentKey, keyName, result) {
    if (isJson(jsonObj) == 0) {
        // 字符串，取消分隔符
        result[parentKey.substring(0, parentKey.length - 1)] = jsonObj;
        return;
    }
    for (var key in jsonObj) {
        var itemkey = key;
        var val = jsonObj[itemkey];
        // 对象
        if (isJson(val) == 1) {
            convertKV(val, parentKey + itemkey + "_",
                keyName, result);
        }
        // 数组对象
        else if (isJson(val) == 2) {
            for (var i = 0; i < val.length; i++) {
                var valElement = val[i];
                var item = i;
                if (valElement[keyName]) {
                    item = valElement[keyName];
                }
                convertKV(valElement, parentKey + itemkey +
                    "_" + item + "_", keyName, result);
            }
        }
        // 值
        else {
            result[parentKey + itemkey] = val;
        }
    }
}

function isJson(obj) {
    // json对象
    if (Object.prototype.toString.call(obj).toLowerCase() ==
        "[object object]" && !obj.length) {
        return 1;
    } else
        // json数组
        if (typeof(obj) == "object" && obj.length) {
            return 2;
        }
    // 值
    else {
        return 0;
    }
}
```

![1542454723554](img\in-post\thingsboard\ruleengine\1542454723554.png)

导出的Root Rule Chain，[点击查看][2]

[2]: img\in-post\thingsboard\ruleengine\root_rule_chain.json	"完整的Rule Chain"

