# HTTPS

- HTTPS
  - SSL
  - SSO




## HTTPS

---

HTTPS는 HTTP와 기본 골격이나 사용 목적 등은 거의 동일하다.

하지만 데이터를 주고 받는 과정에서 '보안' 요소가 추가된 것이 가장 큰 차이이다.

HTTPS를 사용하면 서버와 클라이언트 사이의 모든 통신 내용이 암호화된다.

HTTPS는 SSL이나 TLS 프로토콜을 통해 세션 데이터를 암호화하며, 기본 TCP/IP 포트는 443이고, SSL 프로토콜 위에서 HTTPS 프로토콜이 동작한다.



### 암호화 방식

---



#### 대칭키 방식

---

대칭키는 동일한 키로 암호화와 복호화를 같이 할 수 있는 방식의 암호화 기법이다.

즉 암호화를 할 때 1234라는 값을 사용했다면 복호화를 할 때 1234라는 값을 입력해야 한다. 



대칭키 방식은 단점이 있다. 

암호를 주고 받는 사람들 사이에 대칭키를 전달하는 것이 어렵다는 점이다. 

대칭키가 유출되면 키를 획득한 공격자는 암호의 내용을 복호화 할 수 있기 때문에 암호가 무용지물이 되기 때문이다.



#### 공개키 방식

---

위와 같은 단점 때문에 나온 암호화 방식이 공개키 방식이다.



공개키 방식은 2개의 키를 갖게 된다.

A키를 암호화하면 B키로 복호화할 수 있고, B키로 암호화하면 A키로 복호화할 수 있는 방식이다.

2개의 키 중 하나를 비공개키라 하고 나머지를 공개키로 지정한다.

비공개키는 자신만이 가지고 있고, 공개키를 타인에 제공한다.

공개키를 받은 타인은 공개키를 이용해서 정보를 암호화 한다.

그렇게 되면 비공개키를 가지고 있는 소유자만이 암호화된 정보를 복호화할 수 있는 것이다.

그렇기에 공개키가 유출된다고해도 비공개키를 모르면 복호화할 수 없어 안전하다.



<details>
<summary>공개키 응용 사례 "전자 서명"</summary>
이 방식은 다음과 같이 응용할 수도 있다.<br>
비공개키 소유자가 비공개키를 이용해 정보를 암호화 한 후 공개키와 함께 암호화 된 정보를 전송한다.<br>
정보와 공개키를 받은 사람은 공개키를 이용해서 암호화된 정보를 복호화 한다.<br>
이 과정에서 공개키가 유출된다면 의도치 않은 공격자에 의해서 데이터가 복호화 될 위험이 있다.<br>
이런 위험에도 비공개키를 이용해서 암호화하는 이유는 데이터를 보호하는 것이 목적이 아니기 때문이다.<br>
암호화된 데이터를 공개키를 가지고 복호화할 수 있는 것은 그 데이터가 공개키와 쌍을 이루는 비공개키에 의해서 암호화 되었다는 것을 의미한다.<br>
즉 공개키가 데이터를 제공한 사람의 신원을 보장해 주는 것이다.<br>
이를 전자 서명이라 한다.<br>
</details>




### SSL 인증서

----

SSL 인증서는 클라이언트와 서버간에 통신을 제 3자가 보증해주는 전자화된 문서이다.

클라이언트가 서버에 접속한 직후에는 서버는 클라이언트에게 이 인증서 정보를 전달한다.

클라이언트는 이 인증서 정보가 신뢰할 수 있는 것인지 검증한 후에 다음 절차를 수행한다.

![img](https://t1.daumcdn.net/cfile/tistory/2171D643590C3F380B)



#### SSL 인증서의 역할

---

- 클라이언트가 접속한 서버가 신뢰할 수 있는 서버임을 보장한다.
- SSL 통신에 사용할 공개키를 클라이언트에 제공한다.



#### CA

---

인증서의 역할은 클라이언트가 접속한 서버가 클라이언트가 의도한 서버가 맞는지를 보장하는 역활이다.

SSL을 통해서 암호화된 통신을 제공하려는 서비스는 CA를 통해서 인증서를 구입해야 한다.



#### SSL 인증서의 내용

---

SSL 인증서에는 다음과 같은 정보가 포함되어 있다.

- 서비스의 정보 (인증서를 발급한 CA, 서비스의 도메인 등등)
- 서버 측 공개키 (공개키의 내용, 공개키의 암호화 방법)



위와 같은 내용은 CA에 의해 함호화 된다. 

이 때 사용하는 암호화 기법이 공개키 방식이다.

CA는 자신의 CA 공개키를 이용해서 서버가 제출한 인증서를 암호화하는 것이다.

그렇기에 CA의 비공개키는 절대로 유출되서는 안된다.



인증서를 이해하는데 꼭 알고 있어야 하는 것이 CA의 리스트이다.

브라우저는 내부적으로 CA의 리스트를 미리 파악하고 있다.

브라우저의 소스코드 안에 CA의 리스트가 들어있는 것이다.

브라우저가 미리 파악하고 있는 CA의 리스트에 포함되어야만 공인된 CA가 될 수 있는 것이다. 

CA의 리스트와 각 CA의 공개키를 브라우저는 이미 알고 있다.



#### SSL 인증서가 서비스를 보증하는 방법

---

웹 브라우저가 서버에 접속할 때 서버는 제일 먼저 인증서를 제공한다.

브라우저는 이 인증서를 발급한 CA가 자신이 내장한 CA의 리스트에 있는지 확인한다.

CA 리스트에 포함되어 있다면 해당 CA의 공개키를 이용해서 인증서를 복호화한다.

CA의 공개키를 이용해서 인증서를 복호화 할 수 있다는 것은 이 인증서가 CA의 비공개키에 의해서 암호화 된 것을 의미한다.

해당 CA의 비공개 키를 가지고 있는 CA는 해당 CA 밖에 없기 때문에 서버가 제공한 인증서가 CA에 의해서 발급된 것이라는 것을 의미한다.



### SSL의 동작방법

---

SSL은 암호화된 데이터를 전송하기 위해서 공개키와 대칭키를 혼합해서 사용한다.

즉 클라이언트와 서버가 주고받는 실제 정보는 대칭키 방시긍로 암호화하고, 대칭키 방식으로 암호화된 실제 정보를 복호화할 때 사용할 대칭키는 공개키 방식으로 암호화해서 클라이언트와 서버가 주고 받게 된다.



컴퓨터와 컴퓨터가 네트워크를 이용해서 통신 할때는 "악수 => 전송 => 세션종료" 3가지 단계가 있다.



#### 악수(handshake)

---

SSL 방식을 이용해서 통신을 하는 브라우저와 서버는 역시 핸드쉐이크를 하는데, 이 때 SSL 인증서를 주고 받게된다. 

인증서에 포함된 서버 측 공개키의 역활은 무엇일까?



일단 공개키는 이상적인 통신 방법이다. 

암호화와 복호화를 할 때 사용하는 키가 서로 다르기 때문에 메시지를 전송하는 쪽이 공개키로 데이터를 암호화하고, 수신 받는 쪽이 비공개키로 데이터를 복호화하면 되기 때문이다.

그런데 SSL에서는 이 방식을 사용하지 않는다.

왜냐하면 공개키 방식의 암호화는 매우 많은 컴퓨터 자원을 사용하기 때문이다.

반면 대칭키 방식은 적은 컴퓨터 자원으로 암호화를 수행할 수 있기 때문에 효율적이지만 수신측과 송신측이 동일한 키를 공유해야 하기 때문에 보안의 문제가 발생한다.



그래서 SSL은 공개키와 대칭키의 장점을 혼합한 방식을 사용한다.

헨드쉐이크 단계에서 클라이언트와 서버가 통신하는 과정을 순서대로 살펴보자.



1. 클라이언트가 서버에 접속한다. (Client Hello)
   **클라이언트 측에서 생성한 랜덤 데이터**
   **클라이언트가 지원하는 암호화 방식들** : 클라이언트와 서버가 지원하는 암호화 방식이 서로 다를 수 있기 때문에 상호간에 어떤 암호화 방식을 사용할 것인지에 대해 협상을 한다. 이 협상을 위해서 클라이언트는 자신이 사용할 수 있는 암호화 방식을 전송한다.
   **세션 아이디** : 이미 SSL 핸드쉐이킹을 했다면 비용과 시간을 절약하기 위해서 기존의 세션을 재활용하게 되는데 이때 사용할 연결에 대한 식별자를 서버 측으로 전송한다.
2. 서버는 Client Hello에 대한 응답으로 Server Hello를 하게 된다. 
   **서버 측에서 생성한 랜덤 데이터**
   **서버가 선택한 클라이언트의 암호화 방식** : 클라이언트가 전달한 암호화 방식 중에서 서버 쪽에서도 사용할 수 있는 암호화 방식을 선택해서 클라이언트로 전달한다. 이로써 암호화 방식에 대한 협상이 종료되고 서버와 클라이언트는 이 암호화 방식을 이용해서 정보를 교환한다.
   **인증서**
3. 클라이언트는 서버의 인증서가 CA에 의해서 발급된 것인지를 확인하기 위해서 클라이언트에 내장된 CA 리스트를 확인한다. CA 리스트에 인증서가 없다면 사용자에게 경고 메시지를 출력한다. 인증서가 CA에 의해서 발급된 것인지 확인하기 위해서 클라이언트에 내장된 CA의 공개키를 이용해서 인증서를 복호화 한다. 복호화에 성공했다면 인증서는 CA의 개인키로 암호화된 문서임이 암시적으로 보증된 것이다. 



이러한 절차로 인증서를 전송한 서버를 신뢰할 수 있게 된다.



클라이언트는 2번 단계를 통해서 받은 서버의 랜덤 데이터와 클라이언트가 생성한 랜덤 데이터를 조합해서 pre master secret 이라는 키를 생성하게 된다. 

이 키는 세션 단계에서 데이터를 주고 받을 때 암호화하기 위해서 사용된다.

이 때 사용할 암호화 기법은 대칭키이기 때문에 pre master secret 값은 제 3자에게 노출 되어서는 안된다.



이 pre master secret 값은 공개키 방식으로 서버에 전달된다.

서버의 공개키로 pre master secret 값을 암호화해서 서버로 전송하면 서버는 자신의 비공개키로 안전하게 복호화 한다.

클라이언트는 서버로부터 받은 인증서 안에 서버의 공개키가 있기 때문에 이 공개키를 이용해서 pre master secret 값을 암호화한 후에 서버로 전송하면 안전하게 전송할 수 있다. 



4.  서버는 클라이언트가 전송한 pre master secret 값을 자신의 비공개키로 복호화한다. 이로써 서버와 클라이언트 모두 pre master secret 값을 공유하게 된다. 그리고 서버와 클라이언트는 모두 일련의 과정을 거쳐서 pre master secret 값을 master secret 값으로 만든다. master secret은 session key를 생성하는데 이 session key 값을 이용해서 서버와 클라이언트는 데이터를 대칭키 방식으로 암호화한 후에 주고 받게 된다. 이렇게해서 세션키를 클라이언트와 서버가 모두 공유하게 된다.
5.  클라이언트와 서버는 핸드쉐이크 단계의 종료를 서로에게 알린다.



#### 세션

---

세션은 실제로 서버와 클라이언트가 데이터를 주고 받는 단계이다. 

이 단계의 핵심은 정보를 상대방에게 전송하기 전에 session key 값을 이요해서 대칭키 방식으로 암호화 한다는 점이다.

암호화된 정보는 상대방에게 전송될 것이고, 상대방도 세션키 값을 알고 있어 암호를 복호화 할 수 있다.



#### 세션 종료

---

데이터의 전송이 끝나면 SSL 통신이 끝났음을 서로에게 알려준다. 이 때 통신에서 사용한 대칭키인 세션키를 폐기하게 된다.



<details>
<summary>참고</summary>
<a href=https://12bme.tistory.com/80>https://12bme.tistory.com/80</a>
</details>



### SSO

---

SSO는 Single Sign On의 약자이다.

이는 여러 서비스를 '로그인 한 번'으로 이용하도록 하는 기술이다.



#### 장단점

---

- 장점
  - ID/Password 개별 관리의 위험성 해소
  - 사용자 편의성 증가
- 단점 
  - 구축 비용이 비싸고 시스템 복잡도 증가
  - Single Point of Failure



#### 구현 방식

---



##### 위임방식

---

SSO 에이전트가 인증을 대행하는 방식

![SSO 위임 방식 구조도.jpg](https://itwiki.kr/images/thumb/7/7f/SSO_%EC%9C%84%EC%9E%84_%EB%B0%A9%EC%8B%9D_%EA%B5%AC%EC%A1%B0%EB%8F%84.jpg/500px-SSO_%EC%9C%84%EC%9E%84_%EB%B0%A9%EC%8B%9D_%EA%B5%AC%EC%A1%B0%EB%8F%84.jpg)

대상 애플리케이션의 인증 방식을 변경하기 어려울 때 많이 사용

사용자의 인증 정보를 SSO 에이전트가 관리하며 로그인 대신 수행



#### 전파방식

---

SSO에서 인증을 수행하고, 토큰 발급하고 전달하여 인증 수행

![SSO 전파 방식 구조도.jpg](https://itwiki.kr/images/thumb/8/80/SSO_%EC%A0%84%ED%8C%8C_%EB%B0%A9%EC%8B%9D_%EA%B5%AC%EC%A1%B0%EB%8F%84.jpg/500px-SSO_%EC%A0%84%ED%8C%8C_%EB%B0%A9%EC%8B%9D_%EA%B5%AC%EC%A1%B0%EB%8F%84.jpg)

SSO에서 인증을 받아 대상 애플리케이션으로 전달할 토큰 생성

애플리케이션에선 SSO의 토큰을 검증하고 인증된 것으로 처리



#### 구현

---



##### 웹 기반 SSO

---

- 전파 방식의 경우 쿠키를 통해 토큰을 저장한다.
  - 토큰 쿠키가 있을 경우 토큰값을 검증해 로그인을 수행
  - 쿠키에 저장된 토큰은 필히 암호화하여야 한다.
- 위임 방식의 경우 저장해둔 ID/PASSWORD를 POST로 전달하여 로그인 한다.
  - 또는 ID/PASSWORD를 대신할 수 있는 토큰 URL(GET) 방식으로 전달하여 로그인 한다.



##### 단말 프로그램

---

- 기업용 스프트웨어의 경우 대부분 SSO를 위한 인증 API를 제공한다.
- 소프트웨어에서 인증 기능 커스트마이징이 전혀 되지 않을 경우 SSO 연동이 불가능하다. 
  - 단말용 소프트웨어 도입시 RFP에 해당 SSO 연동 지원 내용 포함 필요



### Localhost에서 HTTPS 적용하기

---



#### mkert

---



> mkcert is a simple tool for making locally-trusted development certificates. It requires no configuration.



mkcert는 로컬 개발 인증서를 만들어주는 도구이다.

이는 특별한 환경설정을 필요로 하지 않는다.



mkcert를 window 환경에서 사용하려면 Chocolately를 이용하여 설치하여야 한다.



##### Chocolately

---

우선 Chocolately는 Linux에서의 apt, yum이나 macOS에서의 Homebrew처럼 패키지를 설치/업데이트/제거 등 관리하는 데에 사용되는 Window용 프로그램이다.



Chocolately를 설치하기 위해서는 powershell을 관리자 권한을 통해 실행시켜 주어야 한다.

그냥 실행하게 되면 설치경로의 변경 등의 작업이 요구된다.



관리자 권한으로 powershell을 실행하였으면 다음의 명령을 실행한다.

```cmd
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```



이후 조금의 시간이 지나면 Chocolately 설치가 완료된다.



##### 인증서 발급

---

위와 같이 Chocolately를 설치한 후 mkcert를 설치한다.

```cmd
choco install mkcert
```



이제 mkcert를 통해 도메인 인증서를 발급한다.

```
 mkcert localhost
```



```
// 결과
Created a new certificate valid for the following names 📜
 - "localhost"

The certificate is at "./localhost.pem" and the key at "./localhost-key.pem" ✅

It will expire on 20 August 2024 🗓
```



그 결과 위와 같은 키파일 localhost-key.pem 그리고 인증파일 localhost.pem이 생성된다.

이 파일들은 서버가 실행되는 파일과 같은 위치에 위치시킨다.



#### https 서버에 적용

---

우선 http 서버와 https 서버를 적용할 포트를 구분해 준다.

```javascript
var http_port = process.env.HTTP_PORT;
var https_port = process.env.HTTPS_PORT;
```



이후 https 서버를 만들 때 옵션을 옵션을 설정한다.

```javascript
var https_options = { key: fs.readFileSync(path.join(__dirname+'/localhost-key.pem')), cert: fs.readFileSync(path.join(__dirname+'/localhost.pem')) };
```



이제 다음과 같이 https 서버를 만든다.

```javascript
https.createServer(https_options, app).listen(https_port, function() {
    logger.info (`${https_port} 서버 가동`);
});
```



전체적인 코드는 다음과 같다.

```javascript
var app = require( '../app');
var fs =require('fs');
var https = require('https')
var path = require('path')
var logger = require("../src/config/logger")

var http_port = process.env.HTTP_PORT;
var https_port = process.env.HTTPS_PORT;
var https_options = 
{ 
    key: fs.readFileSync(path.join(__dirname+'/localhost-key.pem')), 
    cert: fs.readFileSync(path.join(__dirname+'/localhost.pem')) 
};

app.listen (http_port, function() {
    logger.info (`${http_port} 서버 가동`);
});

https.createServer(https_options, app).listen(https_port, function() {
    logger.info (`${https_port} 서버 가동`);
});
```

