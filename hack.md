<!--
    Hackmd 網址
    https://hackmd.io/dOYmkP30TZKjhyVBhzcolg
-->

Tigerfly API Documents
===
API Domain: https://api.tigerflyapp.tw/

User Authentication (身分驗證)
===
使用者身分的驗證，是以 `Authorization` 當作欄位，並且使用 `Bearer {access_token}` 的組合附加在每次的請求的 header 上，以確認使用者的身分。例如 access token 為 `2bf1efe7cc51f6eh5ad1c98f05y9s34463ca85ad`，那麼在 header 中的格式為
- `Authorization:Bearer 2bf1efe7cc51f6eh5ad1c98f05y9s34463ca85ad`。

Tfas (系統簡訊)
===

## POST /tfas/send (系統簡訊發送)

https://api.tigerflyapp.tw/tfas/send

### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type: (choose one of following) 
        - application/x-www-form-urlencoded
        - application/json
- data:
    - access_token: **[required if no Authorization in headers | string]**
    - campaign: custom campaign，can be used to [filter report](#tfas-report-request). 
**[optional | string | max 255 chars]**
        - example: `202004-notification`
    - data **[required | json | max 5000 objects]**
        - id **[optional]**
        - number **[required]**
        - content **[required]**
        - callback_url **[optional]**
 - examples:
    - Content-Type: application/x-www-form-urlencoded
        ```bash
        curl --location --request POST 'https://api.tigerfly.tw/tfas/send' \
        --header 'Content-Type: application/x-www-form-urlencoded' \
        --header 'Authorization: Bearer {token}' \
        --data-urlencode 'data=[
            {
              "id": 1,
              "number": "0900000000",
              "content": "test 1",
              "callback_url": "https://example.com/callback"
            },
            {
              "id": 2,
              "number": "0900000000",
              "content": "test 2",
              "callback_url": "https://example.com/callback"
            }
          ]' \
        --data-urlencode 'campaign=202004-notification'
        ```
    - Content-Type: application/json
        ```bash
        curl --location --request POST 'https://api.tigerfly.tw/tfas/send' \
        --header 'Authorization: Bearer {token}' \
        --header 'Content-Type: application/json' \
        --data-raw '{
          "data": [
            {
              "id": 1,
              "number": "0900000000",
              "content": "test 1",
              "callback_url": "https://example.com/callback"
            },
            {
              "id": 2,
              "number": "0900000000",
              "content": "test 2",
              "callback_url": "https://example.com/callback"
            }
          ],
          "campaign": "202004-notification"
        }'
        ```
      

### Response

- 200 | OK
    ```json
    {
        "data": [
            {
                "id": 1,
                "tfas_id": "07f67d51c8802cc550ae85972798fa5b"
            },
            {
                "id": 2,
                "error": {
                    "code": "1102",
                    "message": "invalid data"
                },
                "tfas_id": "c2ae5d0a3c7cc44505572279293dd33e"
            }
        ]
    }
    ```

- 400 | Bad Request
    ```json
    {
        "error": {
            "code": "1101",
            "message": "data exceeded the maximum size (5000) limit"
        }
    }
    ```

### Callback

You will be notified when the status changed if `callback_url` given.

- parameters
    - tfas_id
        - e.g. `07f67d51c8802cc550ae85972798fa5b`
    - [status_code](#Tfas-Status-Code)
    - update_time (YmdHis)
        - e.g. `20190101000000`

- example:
    ```
    https://example.com/callback?tfas_id=07f67d51c8802cc550ae85972798fa5b&status_code=2&update_time=20190101000000
    ```

## POST /tfas/search (系統簡訊查詢)

https://api.tigerflyapp.tw/tfas/search

### Request

- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type: application/x-www-form-urlencoded
- data:
    - access_token **[required if no Authorization in headers]**
    - tfas_ids **[required | json]**
        - e.g.
            ```
            ["07f67d51c8802cc550ae85972798fa5b","c2ae5d0a3c7cc44505572279293dd33e"]
            ```

### Response

- 200 | OK
    ```json
    {
        "data": [
            {
                "tfas_id": "07f67d51c8802cc550ae85972798fa5b",
                "number": "886900000000",
                "content": "test 1",
                "status_code": "1",
                "sent_time": "2019-04-26T00:00:00+0000"
            },
            {
                "tfas_id": "c2ae5d0a3c7cc44505572279293dd33e",
                "error": {
                    "code": "1201",
                    "message": "record not found"
                }
            }
        ]
    }
    ```

- 400 | Bad Request
    ```json
    {
        "error": {
            "code": "1101",
            "message": "tfas_ids exceeded the maximum size (500) limit"
        }
    }
    ```
    
## GET /tfas/report (系統簡訊報表)

https://api.tigerflyapp.tw/tfas/report

### <a id="tfas-report-request"></a>Request

- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
- data:
    - page: the current page, default is 1.
    - limit: records per page, default is 10, maximum is 500.
    - campaign: campaign filter **[optional]**
    - start_time: start time filter, default is 3 month ago, **can not be earlier than 3 months ago**.
**It will be treated as UTC time if there is no timezone with it.**
        ```
        2020-01-01T00:00:00Z
        2020-01-01T00:00:00+0800
        ```
    - end_time: end time filter, default is current time.
**It will be treated as UTC time if there is no timezone with it.**
        ```
        2020-01-01T23:59:59Z
        2020-01-01T23:59:59+0800
        ```
    - Example
        ```
        https://api.tigerflyapp.tw/tfas/report?page=1&limit=10&campaign=202004news&start_time=2020-04-01T00:00:00%2B0800&end_time=2020-04-01T23:59:59%2B0800
        ```

### Response

- 200 | OK
    ```json
    {
        "current_page": 1,
        "data": [
            {
                "tfas_id": "aa5fed857be479f3e7b43b1e2c9bfe22",
                "campaign": "202004news",
                "number": "886900000000",
                "content": "test content",
                "status_code": "0",
                "sent_time": "2020-04-01T02:00:00+0000"
            },
            {
                "tfas_id": "cfb74c51bf507a8f076e4d1fe5ad1023",
                "campaign": "202004news",
                "number": "886900000000",
                "content": "test content",
                "status_code": "2",
                "sent_time": "2020-04-01T01:00:00+0000"
            }
        ],
        "first_page_url": "https://api.tigerflyapp.tw/tfas/report?page=1",
        "from": 1,
        "last_page": 3,
        "last_page_url": "https://api.tigerflyapp.tw/tfas/report?page=17",
        "next_page_url": "https://api.tigerflyapp.tw/tfas/report?page=2",
        "path": "https://api.tigerflyapp.tw/tfas/report",
        "per_page": 10,
        "prev_page_url": null,
        "to": 10,
        "total": 30
    }
    ```

- 400 | Bad Request
    ```json
    {
        "error": {
            "code": "400",
            "message": "invalid time range parameters"
        }
    }
    ```
    ```json
    {
        "error": {
            "code": "400",
            "message": "limit exceeded the maximum size (500)"
        }
    }
    ```


Delivery (超商未取貨)
===

## POST /delivery/upload (上傳未取貨名單)

https://api.tigerflyapp.tw/delivery/upload

### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type: application/json
- data:
    - schema
        - order_id 訂單編號
        - phone_number 行動電話號碼 **[required]**
        - email 電子郵件地址
        - shipping_id 物流編號 **[required]**
        - cvs_type 超商種類 **[required]**
            - 1: Seven Eleven
            - 2: Family Mart
        - cvs_store_name 門市名稱
        - cvs_store_no 門市店號 **[required]**
        - delivery_time 貨到店時間 **[required]**

    - examples
        ```json
        [
            {
                "order_id": "CVS0001",
                "phone_number": "0912000111",
                "email": "test@examples.com",
                "shipping_id": "100200300400",
                "cvs_type": "1",
                "cvs_store_name": "林安門市",
                "cvs_store_no": "200065",
                "delivery_time": "2019-06-05 19:30:00"
            },
            {
                "order_id": "CVS0001",
                "phone_number": "0912000111",
                "email": "test@examples.com",
                "shipping_id": "100200300400",
                "cvs_type": "4",
                "cvs_store_name": "林安門市",
                "cvs_store_no": "200065",
                "delivery_times": "2019-06-05 19:30:00"
            }
        ]
        ```

### Response
- 200 | OK
    ```json
    {
        "total": 2,
        "success": 1,
        "failed": 1,
        "failed_descriptions": [
            {
                "row": 2,
                "shipping_id": "100200300400",
                "fields": {
                    "delivery_time": "This field is required.",
                    "cvs_type": "This field should be one of the following: [1, 2]."
                }
            }
        ]
    }
    ```

- 400 | Bad Request
    ```json
    {
        "error": {
            "code": 1103,
            "message": "The request header should be application/json format, and body should be a valid json."
        }
    }
    ```


## PATCH /delivery/update/{shipping_id}/picked (更新包裹狀態)

https://api.tigerflyapp.tw/delivery/update/{shipping_id}/picked

### Request
- method: PATCH
- route:
    - shipping_id 物流編號
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type: application/json

### Response
- 200 | OK
    ```json
    {
        "is_picked": true,
        "shipping_id": "2000065"
    }
    ```

- 404 | 找不到商品
    ```json
    {
        "error": {
            "code": 1104,
            "message": "The shipping_id doesn't exist."
        }
    }
    ```

## GET /delivery/status/{shipping_id} (查詢包裹狀態)

https://api.tigerflyapp.tw/delivery/status/{shipping_id}

### Request
- method: GET
- route:
    - shipping_id 物流編號
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type: application/json

### Response
- 200 | OK
    ```json
    {
        "is_picked": true,
        "is_clicked": false,
        "payload": {
            "order_id": "CVS0001",
            "phone_number": "0912000111",
            "email": "test@examples.com",
            "shipping_id": "20000",
            "cvs_type": "1",
            "cvs_store_name": "林安門市",
            "cvs_store_no": "200065",
            "delivery_time": "2019-06-05 19:30:00"
        },
        "create_time": "2019-06-06 12:00:00",
        "update_time": "2019-06-06 12:00:00"
    }
    ```

- 404 | 找不到商品
    ```json
    {
        "error": {
            "code": 1104,
            "message": "The shipping_id doesn't exist."
        }
    }
    ```



Report (報表)
===

## GET /report/tfam/failed (系統信失敗名單)

https://api.tigerflyapp.tw/report/tfam/failed
> 預設提供前一天的失敗名單

### Request
- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
- param:
    - start 開始日期 (yyyy-mm-dd 格式) 例: 2019-07-30 (最多能查詢三個月內的資料)
    - end 結束日期 (yyyy-mm-dd 格式) 例: 2019-06-01 **[required if start given]**

### Response
- 200 | OK
    ```json
    [
        {
            "subject": "some_subject",
            "time": "2019-06-10T15:06:34+0800",
            "to_email": "example@example.com",
            "from_email": "info@em.tigerflyapp.tw",
            "status": "Failed"
        },
        {
            "subject": "some_subject",
            "time": "2019-06-19T15:06:34+0800",
            "to_email": "example@example.com",
            "from_email": "info@em.tigerflyapp.tw",
            "status": "Failed"
        },
        {
            "subject": "some_subject",
            "time": "2019-06-19T15:06:34+0800",
            "to_email": "example@example.com",
            "from_email": "info@em.tigerflyapp.tw",
            "status": "Bounced"
        }
    ]
    ```


Audience (名單上傳)
===

## POST /audience/tfm (行銷信名單上傳)

### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/x-www-form-urlencoded
- data:
    - name **[required]** **(the upload list name)**
    - lists **[required]** **(in json format)**
        - example
            ```json
            [
                {
                    "email" : "abcdefg@awoo.com.tw",
                    "vars" : [
                        "va1",
                        "va2",
                        "va3",
                        "va4",
                        "va5"
                    ]
                },
                {
                    "email" : "higklmn@awoo.com.tw",
                    "vars" : [
                        "vb1",
                        "vb2",
                        "vb3",
                        "vb4",
                        "vb5"
                    ]
                }
            ]
            ```
### Response
  - 200 | OK
    ```json
    {
        "result": true,
        "list_name": "upload123",
        "list_count": 2
    }
    ```

## POST /audience/tfs (行銷簡訊名單上傳)

### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/x-www-form-urlencoded
- data:
    - name **[required]** **(the upload list name)**
    - lists **[required]** **(json format)**
        - example
            ```json
            [
                {
                    "phone" : "0900000000",
                    "vars" : [
                        "va1",
                        "va2",
                        "va3",
                        "va4",
                        "va5"
                    ]
                },
                {
                    "phone" : "0900000001",
                    "vars" : [
                        "vb1",
                        "vb2",
                        "vb3",
                        "vb4",
                        "vb5"
                    ]
                }
            ]`
            ```
### Response
- 200 | OK
    ```json
    {
        "result": true,
        "list_name": "upload123",
        "list_count": 2
    }
    ```

Tfam (系統信)
===

## POST /tfam/send (系統信發送)

### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - multipart/form-data
- data:
    - data **[required]** **(json format)**
        - example
            ```json
            {
                "from": "example@tigerfly.com",
                "from_displayname": "example",
                "to": [
                    {
                        "name": "to",
                        "email": "to@tigerfly.com"
                    }
                ],
                "cc": [
                    {
                      "name": "cc",
                      "email": "cc@tigerfly.com"
                    }
                ],
                "bcc": [
                ],
                "subject": "主旨",
                "body": "",
                "htmlbody": "<!doctype html><html><head><meta charset='utf-8'><title>標題</title></head><body>內容</body></html>",
                "campaign": "campaign"
            }
            ```
        - `from` 寄件者不可使用免費信箱
        - `from_displayname` 寄件者名稱限制 100 字
        - `to` 收件者最多 5 個
        - `cc` 副本最多 5 個
        - `bcc` 密件副本最多 5 個
        - `subject` 主旨限制 100 字
        - `campaign` 限制 255 字元
    - file[] **(file)**
        - 檔名上限 50 個字
        - 附加檔案數量上限為 10 個
        - 附加檔案大小總上限為 50 MB
### Response
- 200 | OK
    ```json
    {
      "result": true
    }
    ```
    
Client-Users (會員)
===
## POST /client-users (批次新增/更新會員)
### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - schema
        - `user_id` 自定義的會員ID，每一位會員的ID應唯一 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `email` 會員的信箱 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `name` 會員名稱
        - `phone_number` 會員手機號碼

    - examples
        ```json
        [
            {
                "user_id": "U1",
                "email": "test1@example.com",
                "name": "test1",
                "phone_number": "0900000000"
            },
            {
                "user_id": "U2",
                "email": "test2@example.com",
                "name": "test2",
                "phone_number": "0900000000"
            }
        ]
        ```
### Response
- 200 | OK
    ```json
    {
        "insert_count": 2,
        "update_count": 0,
        "errors": []
    }
    ```
- 200 | If there are some invalid data
    ```json
    {
        "insert_count": 0,
        "update_count": 0,
        "errors": [
            {
                "index": "0",
                "data": "{\"email\":\"test1@example.com\",\"name\":\"test1\",\"phone_number\":\"0900000000\"}",
                "message": {
                    "user_id": [
                        "user_id can not be empty"
                    ]
                }
            },
            {
                "index": "1",
                "data": "{\"user_id\":\"U2\",\"name\":\"test2\",\"phone_number\":\"0900000000\"}",
                "message": {
                    "email": [
                        "email can not be empty"
                    ]
                }
            }
        ]
    }
    ```
    
## GET /client-users/{user_id} (單筆查詢會員)
### Request
- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)

### Response
- 200 | OK
    ```json
    {
        "users": [
            {
                "user_id": "U1",
                "email": "test1@example.com",
                "name": "test1",
                "phone_number": "886900000000"
            }
        ]
    }
    ```
- 200 | If the user not exists
    ```json
    {
        "users": [
            {
                "user_id": "U1",
                "error": {
                    "code": "1104",
                    "message": "User not found"
                }
            }
        ]
    }
    ```
    
## POST /client-users/search (批次查詢會員)
### Request
- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - examples
        ```json
        [
            "U1",
            "U2"
        ]
        ```

### Response
- 200 | OK
    ```json
    {
        "users": [
            {
                "user_id": "U1",
                "email": "test1@example.com",
                "name": "test1",
                "phone_number": "886900000000"
            },
            {
                "user_id": "U2",
                "email": "test2@example.com",
                "name": "test2",
                "phone_number": "886900000000"
            }
        ]
    }
    ```
- 200 | If there are some users not exist
    ```json
    {
        "users": [
            {
                "user_id": "U1",
                "error": {
                    "code": "1104",
                    "message": "User not found"
                }
            },
            {
                "user_id": "U2",
                "email": "test2@example.com",
                "name": "test2",
                "phone_number": "886900000000"
            }
        ]
    }
    ```
    
Client-Products (產品)
===
## POST /client-products (批次新增/更新產品)
### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - schema
        - `product_id` 自定義的產品ID，每一位產品的ID應唯一 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `title` 產品名稱 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `product_url` 產品連結 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `image_url` 產品圖片連結 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>

    - examples
        ```json
        [
            {
                "product_id": "P1",
                "title": "test1",
                "product_url": "https://www.example.com/P1",
                "image_url": "https://img.example.com/P1.jpg"
            },
            {
                "product_id": "P2",
                "title": "test2",
                "product_url": "https://www.example.com/P2",
                "image_url": "https://img.example.com/P2.jpg"
            }
        ]
        ```
### Response
- 200 | OK
    ```json
    {
        "insert_count": 2,
        "update_count": 0,
        "errors": []
    }
    ```
- 200 | If there are some invalid data
    ```json
    {
        "insert_count": 0,
        "update_count": 0,
        "errors": [
            {
                "index": 0,
                "data": "{\"title\":\"test1\",\"product_url\":\"https:\\/\\/www.example.com\\/P1\",\"image_url\":\"https:\\/\\/img.example.com\\/P1.jpg\"}",
                "message": {
                    "product_id": [
                        "A product ID can't be empty"
                    ]
                }
            },
            {
                "index": 1,
                "data": "{\"product_id\":\"P2\",\"product_url\":\"https:\\/\\/www.example.com\\/P2\",\"image_url\":\"https:\\/\\/img.example.com\\/P2.jpg\"}",
                "message": {
                    "title": [
                        "Product title can't be empty"
                    ]
                }
            }
        ]
    }
    ```

## GET /client-product/{product_id} (單筆查詢產品)
### Request
- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)

### Response
- 200 | OK
    ```json
    {
        "products": [
            {
                "product_id": "P1",
                "title": "test1",
                "image_url": "https://img.example.com/P1.jpg",
                "product_url": "https://www.example.com/P1"
            }
        ]
    }
    ```
- 200 | If the product not exists
    ```json
    {
        "products": [
            {
                "product_id": "P1",
                "error": {
                    "code": "1104",
                    "message": "Product not found"
                }
            }
        ]
    }
    ```

## POST /client-product/search (批次查詢產品)
### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - examples
        ```json
        [
            "P1",
            "P2"
        ]
        ```

### Response
- 200 | OK
    ```json
    {
        "products": [
            {
                "product_id": "P1",
                "title": "test1",
                "image_url": "https://img.example.com/P1.jpg",
                "product_url": "https://www.example.com/P1"
            },
            {
                "product_id": "P2",
                "title": "test2",
                "image_url": "https://img.example.com/P2.jpg",
                "product_url": "https://www.example.com/P2"
            }
        ]
    }
    ```
- 200 | If there are some users not exist
    ```json
    {
        "users": [
            {
                "product_id": "P1",
                "error": {
                    "code": "1104",
                    "message": "Product not found"
                }
            },
            {
                "product_id": "P2",
                "title": "test2",
                "image_url": "https://img.example.com/P2.jpg",
                "product_url": "https://www.example.com/P2"
            }
        ]
    }
    ```
## DELETE /client-product/{product_id} (單筆刪除產品)
### Request
- method: DELETE
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)

### Response
- 200 | OK

## POST /client-product/delete (批次刪除產品)
### Request
- method: GET
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - examples
        ```json
        [
            "P1",
            "P2"
        ]
        ```

### Response
- 200 | OK

Client-Carts (未結購物車)
===
## POST /client-carts (批次新增/更新購物車)
### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - schema
        - `cart_id` 自定義的購物車ID，每一位購物車的ID應唯一 <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `customer_id` [會員ID](#POST-client-users-%E6%89%B9%E6%AC%A1%E6%96%B0%E5%A2%9E%E6%9B%B4%E6%96%B0%E6%9C%83%E5%93%A1) <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `product_id` [產品ID](#POST-client-products-%E6%89%B9%E6%AC%A1%E6%96%B0%E5%A2%9E%E6%9B%B4%E6%96%B0%E7%94%A2%E5%93%81) <span style="color:#FFF; background:#D95C5C; border-radius:8px; padding:3px 9px;">必填</span>
        - `quantity` 商品在購物車的數量
        - `price` 商品在購物車的價格
        - `currency` 參照 ISO 4217 標準，預設為 `TWD`

    - examples
        ```json
        {
            "cart_id": "C1",
            "customer_id": "U1",
            "products": [
                {
                    "product_id": "P1",
                    "quantity": "1",
                    "price": "999",
                    "currency": "TWD"
                },
                {
                    "product_id": "P2",
                    "quantity": "2",
                    "price": "999",
                    "currency": "TWD"
                }
            ]
        }
        ```
### Response
- 200 | OK
- 400 | If there are some invalid data
    ```json
    {
        "error": {
            "code": "1103",
            "message": "The cart_id is required"
        }
    }
    ```

## POST /client-carts/search (批次查詢購物車)
### Request
- method: POST
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)
    - Content-Type
        - application/json
- data:
    - examples
        ```json
        [
            "C1",
            "C2"
        ]
        ```

### Response
- 200 | OK
    ```json
    {
        "carts": [
            {
                "cart_id": "C1",
                "user_id": "U1",
                "products": [
                    {
                        "product_id": "P1",
                        "quantity": 1,
                        "price": 999,
                        "currency": "TWD"
                    },
                    {
                        "product_id": "P2",
                        "quantity": 2,
                        "price": 999,
                        "currency": "TWD"
                    }
                ]
            },
            {
                "cart_id": "C2",
                "user_id": "U2",
                "products": [
                    {
                        "product_id": "P1",
                        "quantity": 1,
                        "price": 999,
                        "currency": "TWD"
                    },
                    {
                        "product_id": "P2",
                        "quantity": 2,
                        "price": 999,
                        "currency": "TWD"
                    }
                ]
            }
        ]
    }
    ```
- 200 | If there are some carts not exist
    ```json
    {
            "carts": [
                {
                    "cart_id": "C1",
                    "user_id": "U1",
                    "products": [
                        {
                            "product_id": "P1",
                            "quantity": 1,
                            "price": 999,
                            "currency": "TWD"
                        },
                        {
                            "product_id": "P2",
                            "quantity": 2,
                            "price": 999,
                            "currency": "TWD"
                        }
                    ]
                },
                {
                    "cart_id": "C2",
                    "error": {
                        "code": "1104",
                        "message": "Cart not found"
                    }
                }
            ]
        }
    ```
## DELETE /client-carts/{cart_id} (單筆刪除購物車)
### Request
- method: DELETE
- header:
    - [Authorization](#User-Authentication-%E8%BA%AB%E5%88%86%E9%A9%97%E8%AD%89)

### Response
- 204 | OK


Code reference
===

### Tfas Status Code

| Code | Description |
| -| -|
| 0 | Sent to service provider|
| 1 | Service provider accepted |
| 2 | Delivered to the phone |
| 3 | The number can't be undelivered |
| 4 | Expired |
| 5 | Error |
| 99 | Undefined |

### Error Code

#### HTTP Code
| Code | Description |
| -| -|
| 400 | Invalid parameters or body content|
| 401 | Authentication failed |
| 404 | Resource not found |
| 429 | Too many requests |
| 500 | Server error |

#### Detail Error Code
| Code | Description |
| -| -|
| 400 | Invalid parameters or body content for general cases |
| 1101 | Empty field |
| 1102 | Field exceeded the maximum size limit |
| 1103 | Invalid request format |
| 1104 | Not found ||
| 1106 | Too many requests |
| 1201 | User upload quota runs out |

### Global error response

- Invalid access_token
    - 401 | Unauthorized
        ```json
        {
            "result": false,
            "error": "invalid_token",
            "message": "The access token provided is invalid"
        }
        ```
