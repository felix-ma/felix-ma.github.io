---
layout:     post
title:      "Js将Json对象解析成key:value格式"
subtitle:   "ThingsBoard中通过网关遥测存入的数据必须是key:value格式的，不能包含json对象。所以编写了一个规则引擎处理逻辑。用于转换"
date:       2018-11-16 18:16:37
author:     "felix"
header-img: "img/in-post/post-bg-js-version.jpg"
catalog:    true
multilingual: false
tags:
    - JavaScript
    - ThingsBoard
---


# Js将Json对象解析成key:value格式

## json对象：

```
{
    "FareGateFaultInfo": {
        "StatusInfo": {
            "StatusCode": "0",
            "MaintainsDateTime": "2017/01/01 01:01:01"
        },
        "DoorContinuous": {
            "-Name": "属性1",
            "FlapTotal": "0",
            "FlapUpdTime": "2017/02/02 02:02:02",
            "FaultCode": "001001001001000",
            "MaintainsDateTime": "2017/03/03 03:03:03"
        },
        "MagneticModule": [
            {
                "-Name": "属性2",
                "FaultCode": "001001001002000",
                "MaintainsDateTime": "2017/04/04 04:04:04"
            },
            {
                "-Name": "属性3",
                "FaultCode": "001001001002000",
                "MaintainsDateTime": "2017/05/05 05:05:05"
            }
        ],
        "SAMCard": {
            "-Name": "属性4",
            "FaultCode": "001001001003000",
            "MaintainsDateTime": "2017/06/06 06:06:06"
        },
        "CardReader": [
            {
                "-Name": "属性5",
                "FaultCode": "001001001004000",
                "MaintainsDateTime": "2017/07/07 07:07:07"
            },
            {
                "-Name": "属性6",
                "FaultCode": "001001001004000",
                "MaintainsDateTime": "2017/08/08 08:08:08"
            }
        ],
        "WriteDateTime": "2017/09/09  09:09:09"
    }
}
```

## 比对代码：

```
function convertKV(jsonObj, parentKey, result) {
    for (key in jsonObj) {
        var itemkey = key;
        var val = jsonObj[itemkey];
        // 对象
        if (isJson(val) == 1) {
            convertKV(val, parentKey + itemkey + ".", result);
        }
        // 数组
        else if (isJson(val) == 2) {
            for (var i = 0; i < val.length; i++) {
                var valElement = val[i];
                convertKV(valElement, parentKey + itemkey + i + ".", result);
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
    if (Object.prototype.toString.call(obj).toLowerCase() == "[object object]" && !obj.length) {
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

## 调用

```
convertKV(data, "", result);
console.log(JSON.stringify(result));
```

返回值

```
{
    "FareGateFaultInfo.StatusInfo.StatusCode":"0",
    "FareGateFaultInfo.StatusInfo.MaintainsDateTime":"2017/01/01 01:01:01",
    "FareGateFaultInfo.DoorContinuous.-Name":"属性1",
    "FareGateFaultInfo.DoorContinuous.FlapTotal":"0",
    "FareGateFaultInfo.DoorContinuous.FlapUpdTime":"2017/02/02 02:02:02",
    "FareGateFaultInfo.DoorContinuous.FaultCode":"001001001001000",
    "FareGateFaultInfo.DoorContinuous.MaintainsDateTime":"2017/03/03 03:03:03",
    "FareGateFaultInfo.MagneticModule0.-Name":"属性2",
    "FareGateFaultInfo.MagneticModule0.FaultCode":"001001001002000",
    "FareGateFaultInfo.MagneticModule0.MaintainsDateTime":"2017/04/04 04:04:04",
    "FareGateFaultInfo.MagneticModule1.-Name":"属性3",
    "FareGateFaultInfo.MagneticModule1.FaultCode":"001001001002000",
    "FareGateFaultInfo.MagneticModule1.MaintainsDateTime":"2017/05/05 05:05:05",
    "FareGateFaultInfo.SAMCard.-Name":"属性4",
    "FareGateFaultInfo.SAMCard.FaultCode":"001001001003000",
    "FareGateFaultInfo.SAMCard.MaintainsDateTime":"2017/06/06 06:06:06",
    "FareGateFaultInfo.CardReader0.-Name":"属性5",
    "FareGateFaultInfo.CardReader0.FaultCode":"001001001004000",
    "FareGateFaultInfo.CardReader0.MaintainsDateTime":"2017/07/07 07:07:07",
    "FareGateFaultInfo.CardReader1.-Name":"属性6",
    "FareGateFaultInfo.CardReader1.FaultCode":"001001001004000",
    "FareGateFaultInfo.CardReader1.MaintainsDateTime":"2017/08/08 08:08:08",
    "FareGateFaultInfo.WriteDateTime":"2017/09/09 09:09:09"
}
```



# 升级版本：加入指定key值取name

## 比对代码

```
function convertKV(jsonObj, parentKey, keyName, result) {
    for (key in jsonObj) {
        var itemkey = key;
        var val = jsonObj[itemkey];
        // 对象
        if (isJson(val) == 1) {
            convertKV(val, parentKey + itemkey + ".", keyName, result);
        }
        // 数组
        else if (isJson(val) == 2) {
            for (var i = 0; i < val.length; i++) {
                var valElement = val[i];
                var item = i;
                if (valElement[keyName]) {
                    item = valElement[keyName];
                }
                convertKV(valElement, parentKey + itemkey + "." + item + ".", keyName, result);
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
    if (Object.prototype.toString.call(obj).toLowerCase() == "[object object]" && !obj.length) {
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

## 返回值

```
{
    "FareGateFaultInfo.StatusInfo.StatusCode":"0",
    "FareGateFaultInfo.StatusInfo.MaintainsDateTime":"2017/01/01 01:01:01",
    "FareGateFaultInfo.DoorContinuous.-Name":"属性1",
    "FareGateFaultInfo.DoorContinuous.FlapTotal":"0",
    "FareGateFaultInfo.DoorContinuous.FlapUpdTime":"2017/02/02 02:02:02",
    "FareGateFaultInfo.DoorContinuous.FaultCode":"001001001001000",
    "FareGateFaultInfo.DoorContinuous.MaintainsDateTime":"2017/03/03 03:03:03",
    "FareGateFaultInfo.MagneticModule.属性2.-Name":"属性2",
    "FareGateFaultInfo.MagneticModule.属性2.FaultCode":"001001001002000",
    "FareGateFaultInfo.MagneticModule.属性2.MaintainsDateTime":"2017/04/04 04:04:04",
    "FareGateFaultInfo.MagneticModule.属性3.-Name":"属性3",
    "FareGateFaultInfo.MagneticModule.属性3.FaultCode":"001001001002000",
    "FareGateFaultInfo.MagneticModule.属性3.MaintainsDateTime":"2017/05/05 05:05:05",
    "FareGateFaultInfo.SAMCard.-Name":"属性4",
    "FareGateFaultInfo.SAMCard.FaultCode":"001001001003000",
    "FareGateFaultInfo.SAMCard.MaintainsDateTime":"2017/06/06 06:06:06",
    "FareGateFaultInfo.CardReader.属性5.-Name":"属性5",
    "FareGateFaultInfo.CardReader.属性5.FaultCode":"001001001004000",
    "FareGateFaultInfo.CardReader.属性5.MaintainsDateTime":"2017/07/07 07:07:07",
    "FareGateFaultInfo.CardReader.属性6.-Name":"属性6",
    "FareGateFaultInfo.CardReader.属性6.FaultCode":"001001001004000",
    "FareGateFaultInfo.CardReader.属性6.MaintainsDateTime":"2017/08/08 08:08:08",
    "FareGateFaultInfo.WriteDateTime":"2017/09/09 09:09:09"
}
```

