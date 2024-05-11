# E生態開放接口文檔Ver3.1.4

## 0x00 修訂歷史

- **2018年6月18日** 黎欣 創建文檔結構
- **2018年6月19日** 黎欣 增加獲取設備列表, 獲取設備詳情, 調用聆耘App服務接口
- **2018年6月21日** 黎欣 授權用戶列表做爲公共屬性提至通用屬性部分
- **2018年12月25日** 黎欣 增加設備採集數據相關API
- **2018年12月27日** 贾磊 優化文檔格式
- **2019年1月21日** 贾磊 優化參數說明
- **2020年7月2日** 黎欣 增加天圻壓電式雨量計數據查詢接口
- **2021年1月4日** 黎欣 增加根系深度字段以及修改设备属性接口
- **2022年5.7日** 黎欣 增加根系深度字段以及修改設備屬性接口
- **2023年8.8日** 黎欣 增加修改設備上報週期接口
- **2023年10.24** 黎欣 增加見釐示例數據

## 0x01 API概覽

| **API**                                  | **描述**                                     |
| ---------------------------------------- | -------------------------------------------- |
| /v3/token                                | 獲取接口訪問授權碼                           |
| /v3/devices                              | 獲取用戶賬戶下設備列表(含已獲得授權設備)     |
| /v3/device/{sn}                          | 獲取設備詳情                                 |
| /v3/device/{sn}/attr                     | 修改設備屬性                                 |
| /v3/device/{sn}/data                     | 獲取設備採集數據                             |
| /v3/device/{sn}/description              | 獲取設備採集參數的描述                       |
| /v3/device/{sn}/data/incremental         | 獲取設備的增量採集數據                       |
| /v3/device/{sn}/latest                   | 獲取設備最新一包採集數據                     |
| /v3/device/{sn}/data/moment/{datetime}   | 查詢設備某個時刻的採集數據                   |
| /v3/device/webhook                       | 查詢或修改數據推送地址                       |
| /v3/device/lingyun/{sn}/service          | 調用聆耘內App服務                            |
| /v3/device/pluviometer/{sn}/data/stats   | 獲取天圻壓電式雨量計降雨歷史統計數據         |
| /v3/device/pluviometer/{sn}/data/history | 獲取天圻壓電式雨量計歷史數據                 |
| /v3/device/pluviometer/{sn}/data/latest  | 獲取天圻壓電式雨量計傳感器數據及實時降雨信息 |
| /v3/device/{sn}/attr                     | 修改設備屬性和上報週期                       |

## 0x02 快速入門

### 概述

文檔中所介紹接口請求均爲HTTP協議，接口請求支持URL傳參以及RequestBody傳參（支持PUT，POST），接口響應暫時僅支持JSON格式，如無特殊說明，則所有接口都需要傳入授權碼請求頭：Authorization，該字段值通過請求下文中：/v3/token 接口獲得。

### 請求結構

以下是獲取設備列表接口的請求地址:

```
http://openapi.ecois.info/v3/devices?page=1&limit=20
```

- http爲通信協議；
- openapi.ecois.info是開放接口服務器域名
- /v3/devices是接口具體路徑
- ?page=1&limit=20是接口所需的URL參數

完整的請求示例(curl)：

```
curl -H 'Content-Type: application/json' \
-H 'Authorization: AccessToken' \
'http://openapi.ecois.info/v3/devices?page=1&limit=100'
```

**備註**：如果是通過PostMan或cURL等工具發起請求，請記得添加請求頭：Content-Type: application/json

### 接口状态码

- 200 OK - [GET]：服務器成功返回用戶請求的數據，該操作是冪等的（Idempotent）。
- 201 CREATED - [POST/PUT/PATCH]：用戶新建或修改數據成功。
- 202 Accepted - [-]：表示一個請求已經進入後臺排隊（異步任務）
- 204 NO CONTENT - [DELETE]：用戶刪除數據成功。
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用戶發出的請求有錯誤，服務器沒有進行新建或修改數據的操作，該操作是冪等的。
- 401 Unauthorized - [-]：表示用戶沒有權限（令牌、用戶名、密碼錯誤）。
- 403 Forbidden - [-] 表示用戶得到授權（與401錯誤相對），但是訪問是被禁止的。
- 404 NOT FOUND - [-]：用戶發出的請求針對的是不存在的記錄，服務器沒有進行操作，該操作是冪等的。
- 406 Not Acceptable - [GET]：用戶請求的格式不可得（比如用戶請求XML格式，但是隻有JSON格式）。
- 410 Gone -[GET]：用戶請求的資源被永久刪除，且不會再得到的。
- 422 Unprocesable entity - [POST/PUT/PATCH] 當創建一個對象時，發生一個驗證錯誤。
- 500 INTERNAL SERVER ERROR - [-]：服務器發生錯誤

### 返回结果

```
// 如果HTTP狀態碼爲2xx，message == ok，則請求成功
// 否則message爲具體的失敗信息
{
    "message": "ok",    
    //接口相關的返回結果
}
```

## 0x03 API.獲取接口授權碼

### 描述

接口地址：/v3/token，使用該接口時建議將返回的token信息在您的應用程序中緩存起來，以提高接口訪問性能，每次使用時檢查是否過期，如果即將過期或已經過期，請及時重新創建token，避免影響業務。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**              |
| -------- | -------- | ------------ | --------------------- |
| appid    | String   | 是           | 在E生態創建的應用ID   |
| secret   | String   | 是           | 在E生態創建的應用密鑰 |

### 返回參數   

| **名稱** | **類型** | **描述**                       |
| -------- | -------- | ------------------------------ |
| token    | String   | 獲取的授權碼                   |
| expires  | Integer  | 授權碼的有效使用時長，單位：秒 |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/token?appid=test&secret=test
// 響應示例   
{
    "message": "ok",
    "token": "wj3eZGGKofgv7GyzuCmuoLukVUvUsuDq",    
    "expires": 7200
}
```

## 0x04 API.獲取設備列表

### 描述

接口地址：/v3/devices，獲得賬戶下的所有設備（包含授權查看的設備）基本信息。可通過返回設備基本信息中的字段authorized來篩選區分您擁有的設備和被授權使用的設備。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**     |
| -------- | -------- | ------------ | ------------ |
| page     | Integer  | 是           | 查詢頁數     |
| limit    | Integer  | 是           | 每頁設備數量 |

### 返回參數   

| **名稱**          | **類型**   | **描述**                                                     |
| ----------------- | ---------- | ------------------------------------------------------------ |
| count             | Integer    | 設備總數                                                     |
| list              | Array      | 設備集合                                                     |
| sn                | String     | 設備序列號                                                   |
| alias             | String     | 設備別名                                                     |
| type_id           | String     | 設備類型                                                     |
| series            | String     | 設備系列                                                     |
| location          | Object     | 位置信息                                                     |
| authorized        | String     | 授權類型, own:直接擁有, shared:被授權                        |
| status            | Object     | 設備狀態信息                                                 |
| code              | String     | 設備狀態                                                     |
| description       | String     | 設備狀態描述                                                 |
| lng               | Bigdecimal | 緯度                                                         |
| lat               | Bigdecimal | 經度                                                         |
| country           | String     | 國家                                                         |
| province          | String     | 省                                                           |
| city              | String     | 城市                                                         |
| district          | String     | 縣區                                                         |
| widget            | Object     | 爲其他外部應用程序提供的小部件支持                           |
| externalAccessUrl | String     | 通過本地址可直接訪問該設備在E生態的詳情頁面，支持嵌入至iframe |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/devices?page=1&limit=50
// 響應示例   
{
    "message": "ok",
    "count": 1,
    "list": [
        {
            "sn": "00000000000000",
            "alias": "测试设备",
            "type": "土壤墒情仪",
            "series": "智墒",
            "authorized": "own", 
            "status": {
                "code": "Fault",
                "description": "停发数据",
            },
            "location": {
                "lng": 105.6846492425,
                "lat": 33.8720641309,
                "country": "中国",
                "province": "陕西省",
                "city": "咸阳市",
                "district": "秦都区"
            },
            "widget": {
                "externalAccessUrl": "https://www.ecois.info/path/to"
            }
        }
    ]
}
```

## 0x05 API.獲取設備詳情

### 描述

接口地址：**/v3/device/{sn}**，獲得設備詳細資訊。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**   |
| -------- | -------- | ------------ | ---------- |
| sn       | String   | 是           | 設備序列號 |

### 返回參數   

| **名稱**          | **類型**   | **描述**                                                     |
| ----------------- | ---------- | ------------------------------------------------------------ |
| sn                | String     | 設備序列號                                                   |
| alias             | String     | 設備別名                                                     |
| type              | String     | 設備類型                                                     |
| series            | String     | 設備系列                                                     |
| status            | Object     | 設備狀態資訊                                                 |
| code              | String     | 設備狀態                                                     |
| description       | String     | 設備狀態描述                                                 |
| lng               | Bigdecimal | 緯度                                                         |
| lat               | Bigdecimal | 經度                                                         |
| country           | String     | 國家                                                         |
| province          | String     | 省                                                           |
| city              | String     | 城市                                                         |
| district          | String     | 縣區                                                         |
| authorizedUsers   | Array      | 擁有授權的用戶列表,如果authorized是shared則返回內容為空      |
| authorized        | String     | 授權類型, own:直接擁有, shared:被授權                        |
| name              | String     | 名字                                                         |
| headimg           | String     | 頭像URL地址                                                  |
| feature           | Object     | 設備特性資訊，獲取雲衍/聆耘設備類型時會有內容，否則為空      |
| online            | Boolean    | 是否線上                                                     |
| rssi              | Integer    | 信號強度                                                     |
| network           | String     | 網路模式，返回值包括 { No service                            |
| widget            | Object     | 為其他外部應用程式提供的小部件支持                           |
| externalAccessUrl | String     | 通過本地址可直接訪問該設備在E生態的詳情頁面，支持嵌入至iframe |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000
// 響應示例   
{
    "message": "ok",
    "sn": "00000000000000",
    "alias": "示例设备",    
    "type": "土壤墒情仪",          
    "series": "智墒", 
    "status": {
        "code": "Fault",
        "description": "停发数据"
    },
    "location": {          
        "lng": 105.6846492425,
        "lat": 33.8720641309,
        "country": "中国",
        "province": "甘肃省",
        "city": "陇南市",
        "district": "成县"
    },
    "authorized": "own",   
    "authorizedUsers": [   
        {
            "openid": "s9n3n9s1zns475a",
            "name": "测试用户",
            "headimg": "头像URL地址"
        }
    ],
    "feature": {           
        "online": false,
        "rssi": -56,
        "network":"LTE"
    }，
    "widget": {
        "externalAccessUrl": "https://www.ecois.info/path/to"
    }
}
```

## 0x06 API.調用聆耘內App提供的服務

### 描述

接口地址：/v3/device/lingyun/{sn}/service，調用聆耘內APP提供的不同服務。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**                                               |
| -------- | -------- | ------------ | ------------------------------------------------------ |
| package  | String   | 是           | 敏捷灌溉App包名:com.insentek.microapp.agile-irrigation |
| name     | String   | 是           | 需調用的服務名稱                                       |
| params   | Object   | 否           | 服務參數, 需對參數進行UrlEnocde.                       |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/service?package=&name=&params=
// 響應示例   
{
    "message": "ok",
    "response": {
        //App返回結果
    }
}
```

## 0x07 API.獲取設備採集數據

### 描述

接口地址：/v3/device/{sn}/data，根據不同參數獲取該設備對應時間內的採集數據。其中如需查詢某個參數或者多個參數數據，可通過/v3/device/{sn}/description設備採集參數的描述介面獲取到每個參數的名稱（key值），如：moisture和temperature等。

### 請求參數   

| **名稱**          | **類型** | **是否必需** | **描述**                                             |
| ----------------- | -------- | ------------ | ---------------------------------------------------- |
| sn                | String   | 是           | 設備序列號                                           |
| range             | String   | 否           | 查詢的起始時間與結束時間，不填時默認返回今日採集數據 |
| includeParameters | String   | 否           | 返回結果包含的參數，默認返回全部參數採集數據         |
| includeNodes      | String   | 否           | 過濾設備節點，默認返回全部節點數據                   |

### 返回參數   

| **名稱**  | **類型**  | **描述**                          |
| --------- | --------- | --------------------------------- |
| total     | Integer   | 數據總數                          |
| list      | Array     | 採集數據                          |
| timestamp | Timestamp | 時間戳（單位：秒）                |
| datetime  | DateTime  | 時間（格式：2018-11-02 22:33:01） |
| values    | Object    | 多個節點的採集數據                |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/data?range=20180102,20181112&includeParameters=moisture,temperature&includeNodes=0,1,2,3,4
// 響應示例   
{
    "message": "ok",
    "total": 1,
    "list": [
        {
            "timestamp": 1541169181,
            "datetime": "2018-11-02 22:33:01",
            "values": {
                "10cm": {
                    "moisture": 12.3,
                    "temperature": 10.7
                },
                "20cm": {
                    "moisture": 12.3,
                    "temperature": 9.9
                },
                "30cm": {
                    "moisture": 12.3,
                    "temperature": 9.8
                },
                "40cm": {
                    "moisture": 12.3,
                    "temperature": 8.5
                }
            }
        }
    ]
}
```

## 0x08 API.獲取設備採集參數的描述

### 描述

接口地址：/v3/device/{sn}/description，獲取設備採集的參數說明。其中返回參數nodes節點列表和parameters參數列表通過下標來一一對應。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**   |
| -------- | -------- | ------------ | ---------- |
| sn       | String   | 是           | 設備序列號 |

### 返回參數   

| **名稱**   | **類型** | **描述**                        |
| ---------- | -------- | ------------------------------- |
| nodes      | Array    | 節點                            |
| parameters | Array    | 參數                            |
| name       | String   | 參數名稱                        |
| unit       | String   | 單位符號                        |
| type       | Enum     | 數據類型（text/numeric/binary） |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/description
// 響應示例   
{
    "message": "ok",
    "nodes": [
        "地表",
        "10cm",
        "20cm",
        "30cm",
        "40cm"
    ],
    "parameters": [
        {
            "moisture": {
                "name": "含水量",
                "unit": "%",
                "type": "text|numeric|binary"
            },
            "temperature": {
                "name": "温度",
                "unit": "℃"
            }
        }
    ]
    ...
}
```

## 0x09 API.增量獲取設備採集數據

### 描述

接口地址：/v3/device/{sn}/data/incremental，獲取設備增量的採集數據。該介面返回**上次請求時間**到**本次請求時間**所產生的所有採集數據。如果是第一次請求該介面，則返回近3個月的採集數據，更早期的數據可通過/v3/device/{sn}/data介面指定時間範圍來獲取。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**   |
| -------- | -------- | ------------ | ---------- |
| sn       | String   | 是           | 設備序列號 |

### 返回參數   

| **名稱**   | **類型**  | **描述**                                    |
| ---------- | --------- | ------------------------------------------- |
| lastSync   | DateTime  | 上一次同步時間（格式：2018-11-02 22:33:01） |
| increments | Array     | 增量數據                                    |
| timestamp  | Timestamp | 時間戳（單位：秒）                          |
| datetime   | DateTime  | 時間（格式：2018-11-02 22:33:01）           |
| values     | Object    | 採集數據                                    |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/data/incremental
// 響應示例   
{
    "message": "ok",
    "lastSync": "2018-11-12 12:33:12",
    "increments": [
        {
            "timestamp": 1541169181,
            "datetime": "2018-11-02 22:33:01",
            "values": {
                "10cm": {
                    "moisture": 12.3,
                    "temperature": 8.2
                },
                "20cm": {
                    "moisture": 12.3,
                    "temperature": 7.6
                },
                "30cm": {
                    "moisture": 12.3,
                    "temperature": 7.6
                },
                "40cm": {
                    "moisture": 12.3,
                    "temperature": 7.5
                }
            }
        }
    ]
}
```

## 0x10 API.獲取設備最新一包採集數據

### 描述

接口地址：/v3/device/{sn}/latest，獲取設備最新的採集數據。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**   |
| -------- | -------- | ------------ | ---------- |
| sn       | String   | 是           | 設備序列號 |

### 返回參數   

| **名稱**  | **類型**   | **描述**                          |
| --------- | ---------- | --------------------------------- |
| timestamp | Timestamp  | 時間戳（單位：秒）                |
| datetime  | DateTime   | 時間（格式：2018-11-02 22:33:01） |
| lng       | Bigdecimal | 經度                              |
| lat       | Bigdecimal | 緯度                              |
| values    | Object     | 採集數據                          |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/latest
// 響應示例   
{
    "message": "ok",
    "timestamp": 1541169181,
    "datetime": "2018-11-02 22:33:01",
    "lng": 105.6846492425,
    "lat": 33.8720641309,
    "values": {
        "10cm": {
            "moisture": 12.3,
            "temperature": 8.2
        },
        "20cm": {
            "moisture": 12.3,
            "temperature": 7.6
        },
        "30cm": {
            "moisture": 12.3,
            "temperature": 7.6
        },
        "40cm": {
            "moisture": 12.3,
            "temperature": 7.5
        }
    }
}
```

## 0x11 API.獲取設備指定時刻的採集數據

### 描述

接口地址：/v3/device/{sn}/data/moment/{datetime}，獲取指定時刻的設備採集數據。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**                      |
| -------- | -------- | ------------ | ----------------------------- |
| sn       | String   | 是           | 設備序列號                    |
| datetime | DateTime | 是           | 時間日期（需要encodeURI轉碼） |

### 返回參數   

| **名稱**  | **類型**  | **描述**                          |
| --------- | --------- | --------------------------------- |
| timestamp | Timestamp | 時間戳（單位：秒）                |
| datetime  | DateTime  | 時間（格式：2018-11-02 22:33:01） |
| values    | Object    | 採集數據                          |

### 示例

```
// 請求示例   
GET  http://openapi.ecois.info/v3/device/00000000000000/moment/2018-12-12%2009:32:22
// 響應示例   
{
    "message": "ok",
    "timestamp": 1541169181,
    "datetime": "2018-11-02 22:33:01",
    "values": {
        "10cm": {
            "moisture": 12.3,
            "temperature": 8.2
        },
        "20cm": {
            "moisture": 12.3,
            "temperature": 7.6
        },
        "30cm": {
            "moisture": 12.3,
            "temperature": 7.6
        },
        "40cm": {
            "moisture": 12.3,
            "temperature": 7.5
        }
    }
}
```

## 0x12 API.設置數據推送地址

### 描述

接口地址：/v3/device/webhook，設置設備採集數據推送地址，系統會將設備即時採集數據以POST請求推送至設置的URL，數據格式參考：/v3/device/{sn}/data/moment/{datetime}接口的數據返回格式。

推送時，如果發起連接或等待回應5秒未回應則推送失敗，失敗後系統將在10分鐘、30分鐘、60分鐘後發起重試（首次推送失敗後等待10分鐘重試，第二次失敗後等待30分鐘重試……）。 如果數據接收方處理完畢，則直接回應字串success即可。

### 請求參數   

| **名稱**  | **類型** | **是否必需** | **描述**          |
| --------- | -------- | ------------ | ----------------- |
| notifyUrl | URL      | 是           | 接收數據推送的URL |

### 示例

```
// 請求示例   
POST  http://openapi.ecois.info/v3/device/webhook
postBody:{
	"notifyUrl":"http://domain.com/path/to"
}
```

## 0x13 API.查詢數據推送地址

### 描述

接口地址：/v3/device/webhook，獲取設置的設備採集數據推送地址。

### 返回參數   

| **名稱**     | **類型** | **描述**                                    |
| ------------ | -------- | ------------------------------------------- |
| notifyUrl    | URL      | 接收數據推送的URL                           |
| lastModified | DateTime | 上一次修改時間（格式：2018-11-02 22:33:01） |

### 示例

```
// 請求示例   
GET http://openapi.ecois.info/v3/device/webhook
// 響應示例   
{
    "notifyUrl": "http://domain.com/path/to",
    "lastModified": "2018-11-12 22:22:44"
}
```

## 0x14 API.查詢天圻壓電式雨量計降雨歷史統計[3.1.2新增]

### 描述

接口地址：/v3/device/pluviometer/{sn}/data/stats，獲取天圻壓電式雨量計降雨歷史統計數據。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**                                               |
| -------- | -------- | ------------ | ------------------------------------------------------ |
| sn       | String   | 是           | 設備序列號                                             |
| range    | String   | 否           | 查詢的起始時間與結束時間，不填時默認返回今日統計數據， |

### 返回參數   

| **名稱**         | **類型**  | **描述**                                        |
| ---------------- | --------- | ----------------------------------------------- |
| rainfallTimes    | Integer   | 總降雨次數                                      |
| totalRainfall    | Float     | 總降雨量                                        |
| list             | Array     | 所選時間段內歷次降雨記錄                        |
| beginTime        | Timestamp | 降雨開始時間戳（秒）                            |
| endTime          | Timestamp | 降雨結束時間戳（秒）, 如果尚未結束，則該字段為0 |
| duration         | Float     | 本次降雨持續時長，單位：秒/s                    |
| rainfall         | Float     | 本次降雨量，單位：毫米/mm                       |
| raindropDiameter | Float     | 雨滴直徑估算，單位：毫米/mm                     |
| maxIntensity     | Float     | 最大降雨強度，單位：毫米每分鐘（mm/min）        |
| maxIntensityTime | Timestamp | 最大降雨強度發生時間戳（秒）                    |
| state            | Integer   | 降雨狀態，0 正在降雨 1 降雨結束                 |

### 示例

```
// 請求示例   
GET http://openapi.ecois.info/v3/device/pluviometer/00000000000000/data/stats
// 響應示例   
{
    "message": "ok",
    "rainfallTimes": 24,
    "totalRainfall": 134.32,
    "list": [
        {
            "beginTime": 1577030400,
            "endTime": 1577050400,
            "duration": 452,
            "rainfall": 14.24,
            "raindropDiameter": 1.13,
            "maxIntensity": 3.15,
            "maxIntensityTime": 1577037400,
            "state": 1
        }
        ... 略
    ]
}
```

## 0x15 API.查詢天圻壓電式雨量計歷史數據[3.1.2新增]

### 描述

接口地址：/v3/device/pluviometer/{sn}/data/history，獲取天圻壓電式雨量計歷史數據。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**                                               |
| -------- | -------- | ------------ | ------------------------------------------------------ |
| sn       | String   | 是           | 設備序列號                                             |
| range    | String   | 否           | 查詢的起始時間與結束時間，不填時默認返回今日採集數據， |

### 返回參數   

| **名稱**            | **類型**  | **描述**                         |
| ------------------- | --------- | -------------------------------- |
| total               | Integer   | 數據總數                         |
| datapoints          | Array     | 採集數據資訊                     |
| timeline            | Timestamp | 時間戳（單位：秒）               |
| sensors             | Object    | 與時間戳對應的多個感測器上報數據 |
| Rainfall            | Float     | 本週期累計雨量，單位：毫米/mm    |
| AirTemperature      | Float     | 空氣溫度，單位：度/℃             |
| AtmosphericPressure | Float     | 大氣壓力，單位：百帕/hPa         |
| Humidity            | Float     | 空氣濕度，單位：百分比/%         |
| RainfallDuration    | Integer   | 本週期降雨持續時間，單位：秒/s   |
| Altitude            | Float     | 海拔，單位：米/m                 |

**备注：参数Rainfall和RainfallDuration描述中的“本周期”是指上一次上报到本次上报之间的累计。**

### 示例

```
// 請求示例   
GET http://openapi.ecois.info/v3/device/pluviometer/00000000000000/data/history
// 響應示例   
{
    "message": "ok",
    "total": 24,
    "datapoints": {
        "timeline": [
            1577030400,
            1577037600,
            1577044800,
            1577052000
        ],
        "sensors": [
            {
                "name": "Rainfall",
                "values": [0, 0, 0, 0]
            },
            {
                "name": "AirTemperature",
                "values": [16.3,16.3,16.1,16]
            },
            {
                "name": "AtmosphericPressure",
                "values": [922.1,922.1,922.1,922.1]
            },
            {
                "name": "Humidity",
                "values": [53.2,63.1,63.5,73.9]
            },
            {
                "name": "RainfallDuration",
                "values": [0,0,0,0]
            },
            {
                "name": "Altitude",
                "values": [1376.8,1376.9,1376.9,1375.7]
            }
        ]
    }
}
```

## 0x16 API.查詢天圻壓電式雨量計最新數據和降雨狀態[3.1.2新增]

### 描述

接口地址：/v3/device/pluviometer/{sn}/data/latest，獲取天圻壓電式雨量計最近一次的感測器數據及降雨資訊。

### 請求參數   

| **名稱** | **類型** | **是否必需** | **描述**   |
| -------- | -------- | ------------ | ---------- |
| sn       | String   | 是           | 設備序列號 |

### 返回參數   

| **名稱**               | **類型**  | **描述**                                            |
| ---------------------- | --------- | --------------------------------------------------- |
| latestCollectTimestamp | Timestamp | 最近一次採集時間時間戳（秒）                        |
| rainfallStatus         | Object    | 最近一次降雨資訊，如果無降雨歷史，則該字段為null    |
| beginTime              | Timestamp | 降雨開始時間戳（秒）                                |
| endTime                | Timestamp | 降雨結束時間戳（秒），如果尚未結束，則該字段為0     |
| duration               | Float     | 本次降雨持續時長，單位：秒/s                        |
| rainfall               | Float     | 本次降雨量，單位：毫米/mm                           |
| raindropDiameter       | Float     | 雨滴直徑估算，單位：毫米/mm                         |
| maxIntensity           | Float     | 最大降雨強度，單位：毫米每分鐘（mm/min）            |
| maxIntensityTime       | Timestamp | 最大降雨強度發生時間戳（秒）                        |
| state                  | Integer   | 降雨狀態，0 正在降雨 1 降雨結束                     |
| datapoints             | Object    | 參考0x15 API.查詢天圻壓電式雨量計歷史數據介面內描述 |

### 示例

```
// 請求示例   
GET http://openapi.ecois.info/v3/device/pluviometer/00000000000000/data/latest
// 響應示例   
{
    "message": "ok",
    "latestCollectTimestamp": 1577052000,
    "rainfallStatus": {
        "beginTime": 1577030400,
        "endTime": 1577050400,
        "duration": 452,
        "rainfall": 14.24,
        "raindropDiameter": 1.13,
        "maxIntensity": 3.15,
        "maxIntensityTime": 1577037400,
        "state": 1
    }
    "datapoints": {
        "timeline": [
            1577052000
        ],
        "sensors": [
            {
                "name": "Rainfall",
                "values": [0]
            },
            {
                "name": "AirTemperature",
                "values": [16.3]
            },
            {
                "name": "AtmosphericPressure",
                "values": [922.1]
            },
            {
                "name": "Humidity",
                "values": [53.2]
            },
            {
                "name": "RainfallDuration",
                "values": [0]
            },
            {
                "name": "Altitude",
                "values": [1376.8]
            }
        ]
    }
}
```

## 0x17 API.修改設備屬性和上報週期[3.1.4新增]

**訪問地址：**

POST http://openapi.ecois.info/v3/device/{sn}/attr

參數說明：{sn} 為設備序列號

請求示例   ：

```
// name為設備名稱，reportCycle為上報週期，單位分鐘，支持的值：5, 10, 20, 30, 60, 120 。
{"name": "新的設備名稱", "reportCycle": 5}
```

该接口返回如下结果：

```
{
    "message": "ok"
}
```



## 补充说明：

### 设备状态说明

對0x04/0x05段落中的返回字段status.code進行說明。

| **名稱**      | **描述**                       |
| ------------- | ------------------------------ |
| Used          | 土壤墒情儀、氣象站的工作中狀態 |
| Fault         | 土壤墒情儀、氣象站的故障中狀態 |
| Online        | 雲衍、聆耘的線上狀態           |
| Offline       | 雲衍、聆耘的離線狀態           |
| Produce       | 生產中                         |
| ToBeDelivered | 待發貨                         |
| Repair        | 維修中                         |
| Idle          | 未部署                         |
| Scrap         | 已報廢                         |

### **設備參數說明**

 對0x07中的請求參數includeParameters進行說明。

 設備類型縮寫：氣象監測設備-T，土壤監測設備-Z，見厘液位計-J

| **名稱**                | **描述**           | **所属设备** |
| ----------------------- | ------------------ | ------------ |
| temperature             | 土壤溫度           | Z            |
| moisture                | 水分含量           | Z            |
| ec                      | 鹽分含量           | Z            |
| battery                 | 電池電量           | Z,T          |
| outsideVoltage          | 外部輸入電壓       | Z,T          |
| airTemperature          | 空氣溫度           | T            |
| relativeHumidity        | 相對濕度           | T            |
| dewPoint                | 露點               | T            |
| atmosphericPressure     | 大氣壓力           | T            |
| maxWindSpeed            | 最大風速           | T            |
| averageWindSpeed        | 風速               | T            |
| windDirection           | 風向               | T            |
| rainfall                | 雨量               | T            |
| rainfallDuration        | 雨量時長           | T            |
| solarRadiationIntensity | 當前太陽輻射強度   | T            |
| solarRadiationAmount    | 累計太陽輻射量     | T            |
| sunshineDuration        | 日照時數           | T            |
| pm1.0                   | PM1.0濃度          | T            |
| pm2.5                   | PM2.5濃度          | T            |
| pm10                    | PM10濃度           | T            |
| tvoc                    | 總揮發性有機物含量 | T            |
| laserliquidLevel        | 激光液位           | J            |

### **設備示例數據**

見厘

```
{
  "total": 132,
  "list": [
    {
      "timestamp": 1651939200,
      "datetime": "2022-05-08 00:00:00",
      "values": {
        "地表": {
          // 激光液位
          "laserliquidLevel": 117.0,
          // 電池電壓
          "battery": 3.288,
          // 充電電壓
          "outsideVoltage": 4.845
        }
      }
    },
    {
      "timestamp": 1651939500,
      "datetime": "2022-05-08 00:05:00",
      "values": {
        "地表": {
          "laserliquidLevel": 116.7,
          "battery": 3.288,
          "outsideVoltage": 4.843
        }
      }
    }
  ]
}
```