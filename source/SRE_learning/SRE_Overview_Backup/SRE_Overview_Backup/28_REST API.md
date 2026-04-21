# REST API


:::spoiler **目錄**
[TOC]
:::




## 學習目標
- What is an API?
- REST API 設計
- HTTP Status Code
- 部署書目管理系統
- 練習呼叫 REST API 
- 登入 H2 資料庫


## 情境說明

- 在網路書店擔任程式設計師
- 負責書目庫存查詢系統開發
- 由兩個子系統組成
	- 書目管理系統(已完成)  <  ----- 呼叫REST API 練習
	- 庫存查詢系統(開發中)

## What is an API?

![](https://i.imgur.com/KQsBa6Y.png)

使用者不需要知道後台Server的操作過程，透過API就可以完成需要的目的

### Public API

公共開放的API

[公共API 列表](https://github.com/public-apis/public-apis)

[政府公共平台API](https://data.gov.tw/API)

## REST API

:::spoiler 英文定義
<font color=red> REST is a set of architectural constraints, not a protocol or a standard. </font> API developers can implement REST in a variety of ways.

<font color=red>The REST architectural style describes six constraints.</font> These constraints, applied to the architecture, were originally communicated by Roy Fielding in his doctoral dissertation and defines the basis of RESTful-style.
:::

**它不是開發的標準，也不是協定，是程式開發的江湖規矩，並沒有強制的限制，有6種規定，都有達成的話，我們就稱它為REST API**

- REST API Quick Tips
	- Use HTTP Verbs to Make Your Requests Mean Something
		- 遵循HTTP的Methods
	- Provide Sensible Resource Names
		- 提供合適的資源名稱
		- 規範api的名稱：/api/getbooks --> GET /api/books (前面的GET是Http的動詞)
	- Use HTTP Response Codes to Indicate Status
	- Offer Both JSON and XML

### HTTP Methods

HTTP defines methods (sometimes referred to as verbs, but nowhere in the specification does it mention verb, nor is OPTIONS or HEAD a verb) to indicate the desired action to be performed on the identified resource.

- GET
- POST
- PUT
- DELETE

### REST API 設計

![](https://i.imgur.com/464Q2fa.png)


### CRUD

![](https://i.imgur.com/hDRUY1G.png)



---

## HTTP Status Code

HTTP 狀態碼表明一個 HTTP 要求是否已經被完成。回應分為五種：
- 資訊回應 (Informational responses, 100–199)
- 成功回應 (Successful responses, 200–299)
- 重定向 (Redirects, 300–399)
- 用戶端錯誤 (Client errors, 400–499)
- 伺服器端錯誤 (Server errors, 500–599)


HTTP Server的回應一定要跟Clinet端看到的一樣


---

# 練習範例1

## Table Schema

![](https://i.imgur.com/lgmrjpm.png)


## 部署書目管理系統

建立並執行 Container 
```
$ docker run --rm -d -p 8080:8080 --name bookstore-catalog quay.io/grassknot/bookstore-catalog:1.2.0
```


確認 Container 運行狀態
```
$ docker ps -a
CONTAINER ID   IMAGE                                       COMMAND                  CREATED         STATUS                    PORTS                    NAMES
669f476fa8b9   quay.io/grassknot/bookstore-catalog:1.2.0   "java -Dspring.profi…"   9 seconds ago   Up 8 seconds              0.0.0.0:8080->8080/tcp   bookstore-catalog
```

確認 bookstore-catalog image
```
$ docker images
REPOSITORY                            TAG       IMAGE ID       CREATED         SIZE
quay.io/grassknot/bookstore-catalog   1.2.0     64f1607d41a2   3 days ago      125MB
```

## 測試書目管理系統

透過`curl`命令測試連線我們的網站

```
$ curl localhost:8080
Welcome to Bookstore Catalog REST API
```


```
$ curl localhost:8080/ruok
imok
```


```
$ curl localhost:8080/ruready
yes
```

`curl`命令後面加參數`-v`，會進入健談模式，把它工作的資訊都說出來

```
$ curl -v localhost:8080/ruready
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /ruready HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200                                        ## 200的狀態碼代表成功
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 3
< Date: Fri, 27 May 2022 02:23:29 GMT
<
* Connection #0 to host localhost left intact
```


## 取得所有主題

```
$ curl localhost:8080/api/topics
```

## 打開Google Chrome 

在網址搜尋：`http://<$VMIP>:8080/`
![](https://i.imgur.com/7banjE5.png)

按F12，按F5，選Network，選Headers




---

## 搭配 jq 命令
透過jq 翻譯後就會比較好看

```
$ curl localhost:8080/api/topics | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   835    0   835    0     0   118k      0 --:--:-- --:--:-- --:--:--  135k
[
  {
    "id": 1,
    "name": "Cloud Native",
    "books": [
      {
        "id": 1,
        "isbn": "1492090719",
        "title": "Design Patterns for Cloud Native Applications",
        "author": "Kasun Indrasiri, Sriskandarajah Suhothayan",
        "price": 1980,
        "page": 314
      }
    ]
  },
  ...以下省略
```

在`curl` 後面加上參數 `-s`，代表silent mode，就可以只呈現我們要看的內容
```
$ curl -s localhost:8080/api/topics | jq
[
  {
    "id": 1,
    "name": "Cloud Native",
    "books": [
      {
        "id": 1,
        "isbn": "1492090719",
        "title": "Design Patterns for Cloud Native Applications",
        "author": "Kasun Indrasiri, Sriskandarajah Suhothayan",
        "price": 1980,
        "page": 314
      }
    ]
  },
  ...以下省略
```

到[天瓏書局的網站](https://www.tenlong.com.tw/)查詢Cloud Native，真的查得到書籍

:::spoiler K8s的書籍

```
"id": 2,
    "name": "Kubernetes",
    "books": [
      {
        "id": 2,
        "isbn": "9865024918",
        "title": "Kubernetes 最佳實務",
        "author": "Brendan Burns, Eddie Villalba, Dave Strebel and Lachlan Evenson",
        "price": 411,
        "page": 272
      },

```

:::

取得ID為1的主題

命令：`curl -s localhost:8080/api/topics/1 | jq`
```
$ curl -s localhost:8080/api/topics/1 | jq
{
  "id": 1,
  "name": "Cloud Native",
  "books": [
    {
      "id": 1,
      "isbn": "1492090719",
      "title": "Design Patterns for Cloud Native Applications",
      "author": "Kasun Indrasiri, Sriskandarajah Suhothayan",
      "price": 1980,
      "page": 314
    }
  ]
}
```


## 新增一個主題

命令：
```bash=
$ curl -v -s localhost:8080/api/topics \
--header 'Content-Type: application/json' \
--data '{
    "name": "Big Data"
}'
```

命令回傳結果：
```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
## curl 命令看到--data時，會將Http的Method，從GET轉成POST
## 開頭是 >  ，代表傳給Server的訊息
> POST /api/topics HTTP/1.1   
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 26
>
* Mark bundle as not supporting multiuse
## 開頭是 < ，代表Server回傳的訊息
## 201 ，表示請求成功且有一個新的資源已經依據需要而被建立
< HTTP/1.1 201                                     
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Location: http://localhost:8080/api/topics/4     ## 新增主題取得的資訊
< Content-Length: 0
< Date: Fri, 27 May 2022 03:05:37 GMT
<
* Connection #0 to host localhost left intact
```

檢查
```
$ curl -s http://localhost:8080/api/topics/4 | jq
{
  "id": 4,
  "name": "Big Data",
  "books": []
}
```

### **更新 ID 為 4 的主題**

命令：
```
curl -v -s -X PUT localhost:8080/api/topics \
--header 'Content-Type: application/json' \
--data '{
    "id": "4",
    "name": "Data Technology"
}'
```
> `-x`的參數，代表可以指定Http Methods的動作
`PUT`是Http Methods的動作：更新



命令回傳結果
```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> PUT /api/topics HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 48
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Location: http://localhost:8080/api/topics/4
< Content-Length: 0
< Date: Fri, 27 May 2022 03:20:15 GMT
<
* Connection #0 to host localhost left intact
```

檢查
```
$ curl -s http://localhost:8080/api/topics/4 | jq
{
  "id": 4,
  "name": "Data Technology",      ## 更新成功
  "books": []
}
```


### 刪除所有主題

刪除命令：
```
$ curl -s -v -X DELETE localhost:8080/api/topics
```

命令回傳結果：

```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> DELETE /api/topics HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 405
< Allow: GET, PUT, POST   < ---------------- 只允許這3個Http Methods
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Fri, 27 May 2022 03:24:12 GMT
<
* Connection #0 to host localhost left intact
{"timestamp":"2022-05-27T03:24:12.497+00:00","status":405,"error":"Method Not Allowed","path":"/api/topics"}
```


### 刪除 ID 為 5 的主題

命令：

```
$ curl -v -X DELETE localhost:8080/api/topics/5
```

回傳結果:

```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> DELETE /api/topics/5 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 404
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Fri, 27 May 2022 03:28:18 GMT
<
* Connection #0 to host localhost left intact
{"timestamp":"2022-05-27T03:28:18.157+00:00","status":404,"error":"Not Found","path":"/api/topics/5"}
```

最後一行：
{"timestamp":"2022-05-27T03:28:18.157+00:00","status":404,"error":"Not Found","path":"/api/topics/5"}

> 可以看到最後一行它說找不到這個組題，所以刪除失敗

### 刪除 ID 為 1 的主題

命令：

```
$ curl -v -X DELETE localhost:8080/api/topics/1
```

命令回傳結果：

```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> DELETE /api/topics/1 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.83.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Length: 0
< Date: Fri, 27 May 2022 03:31:28 GMT
<
* Connection #0 to host localhost left intact
```

檢查

```
$ curl -s http://localhost:8080/api/topics/1 | jq
{
  "timestamp": "2022-05-27T03:32:17.892+00:00",
  "status": 404,
  "error": "Not Found",
  "path": "/api/topics/1"
}
```

> 刪除成功！

### 還原書目管理系統

重新部署
停止Container

```
$ docker stop bookstore-catalog
bookstore-catalog
```

> 因為剛剛建立Container時，有加`--rm`的參數，所以關閉後，Container就會刪除。


再建立一次Container

```
$ docker run --rm -d -p 8080:8080 --name bookstore-catalog quay.io/grassknot/bookstore-catalog:1.2.0
```


---


# 實作練習1

## 題目


請先還原書目管理系統
進行主題與書的管理，並完成下圖結果
![](https://lh3.googleusercontent.com/rH0JlJizVVrinLi6QsvuzuD6RX78fw1yh0Dk6Mldimsv6WG9PqZmcz-JX1TsDuqP4V0iuEF69s6704uvGbJq8Rv9FC_-sDpRI9GjOnnrEe6xs7Zm9XHDrKUTKvgbqwuLuHqihpUW850LGrmQbg)



---

## 答案

```bash=
#刪除 topics、books
curl -v -X DELETE localhost:8080/api/topics/2
curl -v -X DELETE localhost:8080/api/topics/3
curl -v -X DELETE localhost:8080/api/books/3
curl -v -X DELETE localhost:8080/api/books/4
curl -v -X DELETE localhost:8080/api/books/5

#確認內容
curl -v localhost:8080/api/topics | jq
curl -v localhost:8080/api/books | jq

#修改書籍資訊
curl -v -s -X PUT localhost:8080/api/books \
--header 'Content-Type: application/json' \
--data '{
    "id": 2,
    "isbn": "9865024918",
    "title": "Kubernetes 最佳實務",
    "author": "Brendan Burns, Eddie Villalba, Dave Strebel and Lachlan Evenson",
    "price": 411,
    "page": 272,
    "topic": {
      "id": 1,
      "name": "Kubernetes"  ## 這行可以不打
    }
}'
```

---

# 實作練習2


## 題目


請先還原書目管理系統
參考新書資訊新增主題與書
:::spoiler 新書資訊
新書資訊
書名 ｜Hadoop: The Definitive Guide, 4/e (Paperback)
作者 ｜Tom White
ISBN ｜ 1491901632
頁數 ｜756
價錢 ｜1890
所屬主題 ｜Data Technology
:::

預期結果請參考下圖

![](https://lh5.googleusercontent.com/_UhkqLY0haNtVX8AqPD__Wo1K277jF6cn3afA6aGNRbVDyIoPfPyj18kuVBAd506NqVeryHc7eFjbLulWD2-nbOC3tqK4rYB2t8omGvTnHcDSkWpXok35G8PWPjGWo4dQvdZ4ZkKFuH06uLEpA)


## 答案

```bash=
curl -v -s -X PUT localhost:8080/api/books \
--header 'Content-Type: application/json' \
--data '{
    "id": 6,
    "isbn": "1491901632",
    "title": "Hadoop: The Definitive Guide, 4/e (Paperback)",
    "author": "Tom White",
    "price": 1890,
    "page": 756,
    "topic": {
      "id": 4,
      "name": "Data Technology"
    }
}'
```


```bash=
curl -v -s localhost:8080/api/books \
--header 'Content-Type: application/json' \
--data '{
    "id": "6",
    "isbn": "1491901632",
    "title": "Hadoop: The Definitive Guide, 4/e (Paperback)",
    "author": "Tom White",
    "price": 1890,
    "page": 756,
    "topic": {
      "id": "4"
    }
}'
```


---


# 登入 H2 資料庫

[h2資料庫官網](https://www.h2database.com/html/main.html)

打開 chrome 瀏覽器，網址如下：
`http://<$VMIP>:8080/h2`

![](https://i.imgur.com/7ljnbGP.png)

jdbc:h2:mem:bookstore

bookstore




## 使用 SQL 檢視資料

![](https://i.imgur.com/QDli1Lf.png)






###### tags: `CI/CD`