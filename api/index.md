Tigerfly API Documents
===
API Domain: https://api.tigerflyapp.tw/

Tfas
===

## /tfas/send (系統簡訊發送)
https://api.tigerflyapp.tw/tfas/send
### request
- method: POST
- header: 
    - Content-Type
        - application/x-www-form-urlencoded
- data:
    - access_token **[required]**
    - data **[required]** **(in json format)**
        - id [optional]
        - number **[required]**
        - content **[required]**
        - callback_url [optional]
        - e.g.
`[{"id":1,"number":"0900000000","content":"test 1","callback_url":"https://example.com/callback"},{"id":2,"number":"0900000000","content":"test 2","callback_url":"https://example.com/callback"}]`
### response
```jsonid
Status: 200 OK

{
    "data": [
        {
            "id": 1,
            "tfas_id": "07f67d51c8802cc550ae85972798fa5b"
        },
        {
            "id": 2,
            "tfas_id": "c2ae5d0a3c7cc44505572279293dd33e"
        }
    ]
}
```
```jsonid
Status: 200 OK

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
```jsonld
Status: 400 Bad Request

{
    "error": {
        "code": "1101",
        "message": "empty data"
    }
}
```
```jsonld
Status: 401 Unauthorized

{
    "result": false,
    "error": "invalid_token",
    "message": "The access token provided is invalid"
}
```

### callback params
- tfas_id
    - e.g. `07f67d51c8802cc550ae85972798fa5b`
- status_code

| status_code | description |
| -| -|
| 0 | sent to service provider|
| 1 | service provider accepted |
| 2 | delivered to the phone |
| 3 | the number can't be undelivered |
| 4 | expired |
| 5 | error |
| 99 | undefine |

- update_time (YmdHis)
    - e.g. `20190101000000`

example:
```
https://example.com/callback?tfas_id=07f67d51c8802cc550ae85972798fa5b&status_code=2&update_time=20190101000000
```

