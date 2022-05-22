# 19장-(1) not found 구현하기

## url 분석

```jsx
console.log(url.parse(_url, true));
```

main.js에 추가 시 url에 대한 정보들을 분석해서 알려줌.

- http://localhost:3000/?id=JavaScript에 접속 시 결과

> Url { protocol: null, slashes: null, auth: null, host: null, port: null, hostname: null, hash: null, search: '?id=JavaScript', query: [Object: null prototype] { id: 'JavaScript' }, pathname: '/', path: '/?id=JavaScript', href: '/?id=JavaScript' }

이때 pathname이 ‘/’이고 path에는 '/?id=JavaScript’으로 query string이 포함되어 있음.

→ pathname은 쿼리스트링이 실제로 주소에 포함되어 있어도 쿼리스트링을 제외한 path(/)만 출력.

[][]

- [ ]   왜 pathname에 쿼리스트링 없지?

```jsx
**console.log(url.parse(_url, true).pathname);**
```

위와 같이 수정시 pathname만 출력

> / /favicon.ico

## not found 구현

조건문 활용

```jsx
if(pathname === '/'){ //path가 없는 경우(root)로 접속했을 때
//...
}
else {
  response.writeHead(404);
  response.end('not found');
}
```

if(지금 실행중인 접속이 root(path가 없는 경우라면)) {}안의 내용 실행하고, else (파일을 찾을 수 없는 경우라면) 404를 반환

이때 response.writeHead(숫자);를 통해 서버와 브라우저가 통신하는데 웹서버가

- 200라는 숫자를 보낼 때에는 성공적으로 파일을 전송했다라는 의미이고
- 404라는 숫자를 보낼 때에는 파일을 찾을 수 없을 때이다.
- [ ] path가 없는 경우라는 건 뭐지? 이해가 안됨 /가 path아닌가? 왜 정상적으로 실행되는 url인데 path가 없다고 그러지?
- [ ]   http://localhost:3000/?aadfaa으로 실행하면 홈페이지가 뜬다 왜지? 그냥 무조건 /?만 있으면 되는디? 그러면 무슨 의미?