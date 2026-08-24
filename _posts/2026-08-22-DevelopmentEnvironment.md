---
title: Node.js로 웹 서버 구성하기
date: 2026-08-22 18:47:48 +0900
categories: [Projects, Study]
tags: [nodejs] # TAG names should always be lowercase
img_path: /devSeungBin/devseungbin.github.io/assets/img/posts/2026-08-22-DevelopmentEnvironment/
image:
  path: preview.jpg
  alt: 이미지 미리보기
comments: true
author: seungbin
math: true
toc: true
pin: false
mermaid: true
---

# Node.js로 웹 서버 구성하기

웹 서버는 기본적으로 다음과 같은 기능을 수행한다.

1) 클라이언트의 요청을 대기한다.
2) 요청을 수신하고 처리한다.
3) 적절한 응답을 보낸다.

## 1. Node.js 웹 서버 만들기

Node.js는 `node:http` 모듈로 웹 서버를 구현한다.
```javascript
// CommonJS
const http = require('node:http');

// ES Module
import * as http from 'node:http';
```

`createServer()`로 웹 서버를 생성한다.
```javascript
const server = http.createServer();
```

서버를 생성한 후 `listen()`으로 지정한 포트에서 클라이언트의 요청을 대기한다.
```javascript
server.listen(PORT);
```
<br>

## 2. HTTP 요청을 분석하고 처리하기

HTTP 요청이 도착하면 `request` 이벤트가 발생한다. 웹 서버는 이벤트 리스너를 등록하거나 `createServer()`에 콜백 함수를 전달해 요청을 처리할 수 있다.
```javascript
// 이벤트 리스너 등록
const server = http.createServer();
server.on('request', (request, response) => {
    // ...요청 처리 구현
});

// 콜백 함수 전달 
const server = http.createServer((request, response) => {
    // ...요청 처리 구현
});
```

`request` 객체에는 HTTP 요청으로 들어온 내용이 담겨있다. 

HTTP 메서드, URL, 헤더 등의 요청 정보는 구조 분해 할당으로 추출하며, 요청 본문은 스트림을 통해 데이터를 수신한다.
```javascript
const { headers, method, url } = request;

let body = [];
request
    .on('error', (err) => {
        console.error(err);
    })
    .on('data', (chunk) => {
        body.push(chunk);
    })
    .on('end', () => {
        body = Buffer.concat(body).toString();
    });
```

추출한 내용을 바탕으로 여러 요청에 대한 처리를 각각 구현한다.
```javascript
const { headers, method, url } = request;

if (method === 'GET' && url === '/') {
    // ...특정 요청에 대한 처리 구현

} else {
    // ...그 외 요청 처리 구현
}
```
<br>

## 3. 요청에 따른 HTTP 응답 보내기 

`response` 객체를 사용해 클라이언트에 HTTP 응답을 보낸다.

HTTP 상태 코드는 `statusCode`로 설정하고, 응답 헤더는 `setHeader()`로 추가한다.`writeHead()`로 상태와 헤더를 한꺼번에 설정할 수도 있다.
```javascript
// 상태와 헤더를 각각 설정
response.statusCode = 200;
response.setHeader('Content-Type', 'application/json');

// 상태와 헤더를 한꺼번에 설정
response.writeHead(200, {
    'Content-Type': 'application/json'
});
```

`write()`로 응답 본문을 작성하고 `end()`로 응답을 종료한다. `end()`에 응답 본문을 전달할 수도 있다.
```javascript
// write()로 본문 작성 후 응답 종료
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();

// end()로 본문을 작성하고 응답 종료
response.end('<html><body><h1>Hello, World!</h1></body></html>');
```

<br>

## 4. 간단한 웹 서버 만들기  

앞에서 살펴본 과정을 하나의 웹 서버로 구현해본다.
* `GET /` 요청에는 인사말을 반환한다.
* `POST /` 요청에서는 요청 본문의 `name` 값을 사용해 응답한다.
* 나머지 요청에는 `404 Not Found` 응답을 보낸다.
* 잘못된 JSON 요청에는 `400 Bad Request` 응답을 보낸다.
* `request`와 `response` 스트림 오류는 서버에서 오류를 기록한다.

```javascript
const http = require('node:http');

const server = http.createServer();
server.on('request', (request, response) => {
    const { method, url } = request;
    let body = [];

    request
        .on('error', (err) => {
            console.error(err);
        })
        .on('data', (chunk) => {
            body.push(chunk);
        })
        .on('end', () => {
            response
                .on('error', (err) => {
                    console.error(err);
                });

            if (method === 'GET' && url === '/') {
                response.writeHead(200, { 'Content-Type': 'text/plain' });
                response.end('Hello, world!');

            } else if (method === 'POST' && url === '/') {
                try {
                    body = JSON.parse(Buffer.concat(body).toString());
                
                    response.writeHead(200, { 'Content-Type': 'text/plain' });
                    response.end(`Hello, ${body.name ?? 'unknown'}!`);
                } catch (err) {
                    console.error(err);

                    response.writeHead(400, { 'Content-Type': 'text/plain' });
                    response.end('Bad Request');
                }

            } else {
                response.writeHead(404, { 'Content-Type': 'text/plain' });
                response.end('Not Found');
            }
        });
});

server.listen(8080);
```

![example.jpg](/example.jpg)  
