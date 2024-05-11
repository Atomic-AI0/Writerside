# E生态墒情高级识别相关开放接口（合作伙伴有限开放）-2023.3.21

## 一、YH图数据接口

**访问地址：**

GET <http://openapi.ecois.info/v3/device/{sn}/external/partner/yh-graph>

参数说明：{sn} 为设备序列号，目前支持设备类型为智墒，该接口返回如下结果：

```
{
    // 分层耗水量，与节点内索引对应，单位毫米
    "wastage": [
        1.76,
        0.49,
        0.23
    ],
    // 节点信息
    "nodes": [
        "10",
        "20",
        "30",
        "40",
        "50",
        "60",
        "70",
        "80",
        "90",
        "100"
    ],
    // 分层耗水量占比，与节点内索引对应
    "proportion": [
        71.0,
        19.9,
        9.1
    ],
    // 根系深度，与节点内索引对应，无根系时为-1
    "root": 2,
    // 当前最新作物kc值，无识别为0
    "kc": 0.0,
    // 有根系时当前的耗水量
    "etc": 0,
    // 有根系时的有效储水量，单位毫米
    "waterStorage": 0,
    // 有根系时的蓄水潜力，单位毫米
    "storageCapacity": 66.97,
    // 未来7天ET0预测
    "et0": [
        {
            "date": "20210816",
            "val": 3.78
        },
        {
            "date": "20210817",
            "val": 4.09
        },
        {
            "date": "20210818",
            "val": 4.1
        },
        {
            "date": "20210819",
            "val": 4.18
        },
        {
            "date": "20210820",
            "val": 3.45
        },
        {
            "date": "20210821",
            "val": 4.29
        },
        {
            "date": "20210822",
            "val": 4.49
        }
    ],
    // YH图历史水分极值，0 最小值， 1最大值
    "extremum": [
        [
            2.77,
            3.84,
            5.68,
            2.95,
            3.98,
            3.9,
            4.2,
            2.56,
            3.04,
            9.34
        ],
        [
            39.09,
            40.94,
            39.58,
            39.96,
            40.49,
            38.61,
            38.01,
            37.77,
            37.78,
            36.84
        ]
    ],
    // 当前水分含量
    "moisture": [
        14.32,
        30.83,
        34.11,
        34.41,
        35.41,
        37.15,
        37.77,
        37.45,
        37.57,
        36.41
    ],
    "message": "ok"
}
```

## 二、ET0及降水量预测

**访问地址：**

GET <http://openapi.ecois.info/v3/device/{sn}/external/partner/weather-forecast>

参数说明：{sn} 为设备序列号，该接口返回如下结果：

```
{
    // ET0预测
    "et0": [
        {
            "date": "20210817",
            "val": 4.09
        },
        {
            "date": "20210818",
            "val": 4.05
        },
        {
            "date": "20210819",
            "val": 4.19
        },
        {
            "date": "20210820",
            "val": 2.42
        },
        {
            "date": "20210821",
            "val": 3.81
        },
        {
            "date": "20210822",
            "val": 3.69
        },
        {
            "date": "20210823",
            "val": 4.01
        }
    ],
    //降水量预测
    "precipitation": [
        {
            "date": "20210817",
            "val": 1.711
        },
        {
            "date": "20210818",
            "val": 0.158
        },
        {
            "date": "20210819",
            "val": 0.000
        },
        {
            "date": "20210820",
            "val": 5.938
        },
        {
            "date": "20210821",
            "val": 0.000
        },
        {
            "date": "20210822",
            "val": 0.000
        },
        {
            "date": "20210823",
            "val": 0.720
        }
    ],
    // 未来7天ET0累计
    "totalET0": 26.260,
    "message": "ok",
    // 未来7天降水量累计
    "totalPrecipitation": 8.527
}
```

## 三、墒情/气象分析接口

**访问地址：**

GET <http://openapi.ecois.info/v3/device/{sn}/external/partner/analysis>

参数说明：{sn} 为设备序列号，该接口返回如下结果：

```
{
    "soilMoistureAnalyses": {
        //土壤冻结深度（cm）
        "frozen": 10,
        //灌溉数据
        "irrigation": {
            //灌溉日期（yy-MM-dd）
            "date": "2021-11-29",
            //灌溉量(mm)
            "amount": 18.3,
            //灌溉深度(cm)
            "depth": 40,
            //入渗速率
            "infiltration": {
                "2021-11-29 12:00:03": 0.3
            }
        },
        //水涝胁迫
        "waterlogging": {
            // 水涝胁迫持续时间(H)
            "duration": 32.0,
            "beginTime": "2021-04-22 17:00:02",
            "endTime": "2021-04-24 01:00:05"
        }
    },
	//气象站或雨量计返回字段
    "weatherAnalyses": {
    	"alert": {
			// 预警详情
      		"description": null,
			// 预警发布来源
      		"source": null,
			// 预警发布时间
      		"timestamp": 0
    	},
    	"disasterIndex": {
			//高温等级（0：非高温 1：黄色高温，2：橙色高温，3：红色高温）
      		"highTemperature": 0,
			//大风等级（0-12 无风 软风 轻风  微风 和风 劲风 强风  疾风 大风 烈风 狂风 暴风 飓风）
      		"gale": 0,
			//大雨等级（0-4 无雨 小雨 中雨 大雨 暴雨）
      		"heavyRain": 0,
			//干旱等级（0-4 无旱 轻旱 中旱 重旱 特旱）
      		"drought": 0,
			//污染等级（0-5 优 良 轻度污染 中度污染 重度污染 严重污染）
      		"pollution": 0,
			//火险等级（0-4 一级火险 二级火险 三级火险 四级火险 五级火险）
			"fwi": 0,
			//霜冻等级（0-1 无 霜冻）
			"frost": 0,
			//赤霉病等级（0-3 不流行 轻度流行 中等流行 大流行）
			"headBlight": 0
    	}
  	 },
    "message": "ok"
}
```

## 四、修改设备属性和上报周期

**访问地址：**

POST <http://openapi.ecois.info/v3/device/{sn}/attr>

参数说明：{sn} 为设备序列号

请求示例：

```
// name为设备名称，reportCycle为上报周期，单位分钟，支持的值：5, 10, 20, 30, 60, 120 。
{name: '新的设备名称', reportCycle: 5}
```

该接口返回如下结果：

```
{
    "message": "ok"
}
```