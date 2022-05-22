# 5강 - node.js 도입

## node j.s

웹서버 기능을 내장하고 있어서 웹서버로 활용 가능

아래 코드 사용

```jsx
var http = require('http');
var fs = require('fs');
var app = http.createServer(function(request,response){
    var url = request.url;
    if(request.url == '/'){
      url = '/index.html';
    }
    if(request.url == '/favicon.ico'){
      response.writeHead(404);
      response.end();
      return;
    }
    response.writeHead(200);
    response.end(fs.readFileSync(__dirname + url));
 
});
app.listen(3000);
```

cmd 창으로 node main.js → 실행 → localhost:3000에 접속시 구현한 웹사이트 보임

ctrl+c 누르고 새로고침시 컴파일 불가 →웹서버로서 node.js가 실행되어지고 있음을 확인

```jsx
console.log(__dirname + url); //추가
response.end(fs.readFileSync(__dirname + url)); //변경
```

response.end(fs.readFileSync(__dirname + url))

url → 경로에 해당하는  __dirname → 파일을 만들고 (//현재 실행중인 파일 경로) fs.readFileSync() → 그 파일을 읽음

추가 및 변경 시, 아래와 같이 cmd 창 출력

> C:\Users\jm\Desktop\web/1.html C:\Users\jm\Desktop\web/coding.jpg C:\Users\jm\Desktop\web/index.html

- C:\Users\jm\Desktop\web → 현재 실행중인 파일이 위치한 경로
- /1.html → 사용자가 요청할 때(접근)마다 전송할 정보