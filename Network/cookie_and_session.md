# 쿠키 & 세션

## 쿠키와 세션을 사용하는 이유?

HTTP 프로토콜의 특징이자 약점을 보완하기 위해서 사용한다.

### HTTP 프로토콜의 특징

1. Connectionless 프로토콜 (비연결 지향)

    클라이언트가 서버에 요청(Request)을 했을 때, 그 요청에 맞는 응답(Response)을 보낸 후 연결을 끊는 처리방식이다.
    * HTTP 1.1 버전에서 커넥션을 계속 유지하고, 요청(Request)에 재활용하는 기능이 추가되었다. (HTTP Header)에 keep-alive 옵션을 주어 커넥션을 재활용하게 한다. HTTP 1.1 버전에선 디폴트(default) 옵션이다.
    * HTTP가 TCP위에서 구현되었기 때문에(TCP : 연결 지향, UDP : 비연결 지향) 연결 지향적이라고 할 수 있다는 얘기가 있어 논란이 있지만, 아직까진 네트워크 관점에서 keep-alive는 옵션으로 두고, 서버 측에서 비연결 지향적인 특성으로 커넥션 관리에 대한 비용을 줄이는 것이 명확한 장점으로 보기 때문에 비연결 지향으로 알아두었다.

2. Stateless 프로토콜
  커넥션을 끊는 순간 클라이언트와 서버의 통신이 끝나며 상태 정보는 유지하지 않는 특성이 있다.

    * 클라이언트와 첫 번째 통신에서 데이터를 주고받았다 해도, 두 번째 통신에서 이전 데이터를 유지하지 않는다.
    * 하지만, 실제로는 데이터 유지가 필요한 경우가 많다.

## 쿠키(Cookie)

HTTP의 일종으로 사용자가 어떠한 웹 사이트를 방문할 경우,
그 사이트가 사용하고 있는 서버에서 사용자의 컴퓨터에 저장하는 작은 기록 정보 파일이다.
HTTP에서 클라이언트의 상태 정보를 클라이언트의 PC에 저장하였다가 필요시 정보를 참조하거나 재사용할 수 있다.

### Cookie 유형

| 쿠키 종류| 특징 |
|---|---|
|Session Cookie|일반적으로 만료시간(Expire Date)를 설정하고 메모리에만 저장되며, 브라우저 종료 시 쿠키를 삭제|
|Persistent Cookie|장기간 유지되는 쿠키이다. 파일로 저장되어 브라우저 종료와 관계없이 사용할 수 있다.|
|Secure Cookie| HTTPS 프로토콜에서만 사용하며, 쿠키 정보가 암호화되어 전송된다.|
|Third-Party Cookie|	방문한 도메인과 다른 도메인의 쿠키이다. 일반적으로 광고 배너 등을 관리할 때 유입 경로를 추적하기 위해 사용한다.|

### 쿠키 특징

1. 이름, 값, 만료일(저장기간), 경로 정보로 구성되어 있다.
2. 클라이언트에 총 300개의 쿠키를 저장할 수 있다.
3. 하나의 도메인 당 20개의 쿠키를 가질 수 있다.
4. 하나의 쿠키는 4KB(=4096byte)까지 저장 가능하다.

### 쿠키의 동작 순서

1. 클라이언트가 페이지를 요청한다. (사용자가 웹사이트에 접근)
2. 웹 서버는 쿠키를 생성한다.
3. 생성한 쿠키에 정보를 담아 HTTP 화면을 돌려줄 때, 같이 클라이언트에게 돌려준다.
4. 넘겨받은 쿠키는 클라이언트가 가지고 있다가(로컬 PC에 저장) 다시 서버에 요청할 때 요청과 함께 쿠키를 전송한다.
5. 동일 사이트 재방문 시 클라이언트의 PC에 해당 쿠키가 있는 경우, 요청 페이지와 함께 쿠키를 전송한다.

* 사용 예시
    1. 방문 사이트에서 로그인 시, "아이디와 비밀번호를 저장하시겠습니까?"
    2. 팝업창을 통해 "오늘 이 창을 다시 보지 않기" 체크

## 세션(Session)

일정 시간 동안 같은 사용자(브라우저)로부터 들어오는 일련의 요구를 하나의 상태로 보고, 그 상태를 유지시키는 기술이다.
여기서 일정 시간은 방문자가 웹 브라우저를 통해 웹 서버에 접속한 시점부터 웹 브라우저를 종료하여 연결을 끝내는 시점을 말한다.

즉, 방문자가 웹 서버에 접속해 있는 상태를 하나의 단위로 보고 그것을 세션이라고 한다.

### 세션 특징

웹 서버에 웹 컨테이너의 상태를 유지하기 위한 정보를 저장한다.
웹 서버의 저장되는 쿠키(=세션 쿠키)
브라우저를 닫거나, 서버에서 세션을 삭제했을 때만 삭제가 되므로, 쿠키보다 비교적 보안이 좋다.
저장 데이터에 제한이 없다. (서버 용량이 허용하는 한에서)
각 클라이언트에 고유 Session ID를 부여한다. Session ID로 클라이언트를 구분해 각 요구에 맞는 서비스를 제공

### 세션의 동작 순서

1. 클라이언트가 페이지에 요청한다.
2. 서버는 접근한 클라이언트의 Request-Header 필드인 Cookie를 확인하여, 클라이언트가 해당 session-id를 보냈는지 확인한다.
3. session-id가 존재하지 않는다면 서버는 session-id를 생성해 클라이언트에게 넘겨준다.
4. 클라이언트는 서버로부터 받은 session-id를 쿠키에 저장한다.
5. 클라이언트는 서버에 요청시 이 쿠키의 session-id 값을 같이 서버에 전달한다.
6. 서버는 전달받은 session-id로 session에 있는 클라이언트 정보를 가지고 요청을 처리 후 응답한다.

* 사용 예시
  * 화면을 이동해도 로그인이 풀리지 않고 로그아웃하기 전까지 유지

## 쿠키와 세션의 차이

1. 저장 위치
    * 쿠키: 클라이언트(브라우저)
    * 세션: 서버

2. 보안성
    * 쿠키: 클라이언트에 저장되어 있어 상대적으로 취약
    * 세션: 서버에서 관리되어 보안성이 높음

3. 라이프사이클
    * 쿠키: 설정된 만료 기간까지 유지
    * 세션: 브라우저 종료시 삭제

4. 속도
    * 쿠키: 클라이언트에 저장되어 있어 빠름
    * 세션: 서버의 처리가 필요해 상대적으로 느림

보통 쿠키와 세션의 차이에 대해서 저장 위치와 보안에 대해서는 잘 알고 있지만, 사실 가장 중요한 것은 라이프사이클이다.

## 라이프사이클 비교

| 단계 | 쿠키(Cookie) | 세션(Session) |
|------|--------------|---------------|
| 생성 시점 | - Set-Cookie 헤더로 생성<br>- JavaScript로 직접 생성 | - 최초 요청 시 세션 ID 생성<br>- 주로 로그인 시점 |
| 유지 기간 | - Session Cookie: 브라우저 종료까지<br>- Persistent Cookie: 만료 시간까지 | - 브라우저 종료까지<br>- 서버 설정 타임아웃까지 |
| 소멸 조건 | - 만료 시간 도달<br>- 브라우저 쿠키 삭제<br>- 브라우저 종료 | - 브라우저 종료<br>- 세션 타임아웃<br>- 서버 재시작<br>- 명시적 로그아웃 |

## 주요 특성 비교

| 특성 | 쿠키(Cookie) | 세션(Session) |
|------|--------------|---------------|
| 지속성 | 명시적 만료 시간 설정 가능 | 브라우저 컨텍스트에 종속 |
| 갱신 방식 | 만료 시 자동 삭제 | 활동마다 타임아웃 리셋 가능 |
| 제어권 | 클라이언트 제어 | 서버 제어 |

## 일반적인 사용 사례

| 용도 | 쿠키(Cookie) | 세션(Session) |
|------|--------------|---------------|
| 로그인 관련 | 자동 로그인, 아이디 저장 | 로그인 인증 정보 |
| 쇼핑몰 | 장바구니(비로그인) | 장바구니(로그인) |
| 사용자 설정 | 테마, 언어 설정 | 사용자 권한 정보 |
| 임시 데이터 | 설문 진행 상태 | 결제 진행 정보 |

## 보안 고려사항

| 항목 | 쿠키(Cookie) | 세션(Session) |
|------|--------------|---------------|
| 데이터 노출 | 클라이언트에서 확인 가능 | 서버에서만 확인 가능 |
| 변조 위험 | 있음 | 상대적으로 낮음 |
| 적합한 데이터 | 민감하지 않은 정보 | 중요한 사용자 정보 |