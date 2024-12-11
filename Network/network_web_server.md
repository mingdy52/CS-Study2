# 웹 서버
> 웹 서버 라는 용어는 웹 서버 소프트웨어 와 웹페이지 제공에 특화된 장비 양쪽 모두를 가리킨다.

1.웹 서버 구현

웹 서버는 HTTP 및 그와 관련된 TCP 처리를 구현한 것.
웹 서버는 HTTP 프로토콜을 구현하고 웹 리소스를 관리하고 웹 서버 관리 기능을 제공. 웹 서버는 TCP 커넥션 관리에 대한 책임을 운영체제와 나눠 갖는다.

운영체제는 컴퓨터 시스템의 하드웨어를 관리, TCP/IP 네트워크 지원, 웹 리소스 유지를 위한 파일 시스템 현재 연산 활동을 제어하기 위한 프로세스 관리를 제공

## 웹서버가 하는 일

1. 커넥션을 맺는다.
2. 요청을 받는다.
3. 요청을 처리한다.
4. 리소스에 접근한다.
5. 응답을 만든다.
6. 응답을 보낸다.
7. 트랜잭션을 로그로 남긴다.

![alt text](https://velog.velcdn.com/images/bahar-j/post/6cbdd139-aa0b-4459-951a-af811bfd5686/image.png)

## 1. 클라이언트 커넥션 수락

### 새 커넥션 다루기
웹 서버는 어떤 커넥션이든 마음대로 거절하거나 즉시 닫을 수 있다.

### 클라이언트 호스트 명 식별
대부분의 쉡 서버는 역방향 DNS를 사용해서 클라이언트의 IP 주소 -> 클라이언트의 호스트명 으로 변환하도록 설정되어 있음.
클라이언트 호스트 명을 구체적인 접근제어와 로깅을 위해 사용할 수 있음
호스트명 룩업은 시간이 많이 걸릴 수 있어 웹 트랜잭션을 느리게 할수 있음
대용량 웹 서버는 호스트 명 분석(hostname resolution)을 꺼두거나 특정 콘텐츠에 대해서면 킨다.

### ident를 통해 클라이언트 사용자 알아내기

Ident 프로토콜은 특정 TCP 연결이 어떤 사용자나 프로세스에 의해 시작되었는지를 파악하기 위해 사용됩니다. 이는 보안 감사, 로깅, 또는 액세스 제어를 강화하기 위한 용도로 사용됩니다.

Ident는 과거에 IRC(Internet Relay Chat) 서버와 같은 시스템에서 유용하게 사용되었습니다. IRC 서버는 연결 요청을 보낸 사용자가 누구인지 확인하기 위해 Ident 요청을 보내고, 결과에 따라 접속 허용 여부를 결정했습니다.

현재는 주로 레거시 시스템이나 특정 환경에서 제한적으로 사용됩니다. 많은 네트워크와 애플리케이션은 이를 더 이상 활성화하지 않거나 더 안전한 대안을 사용합니다.

## 2. 요청 메시지 수신

- 웹 서버는 요청줄을 파싱하여 요청 메서드, 지정된 리소스의 식별자(URI), 버전 번호를 찾는다. 각 값은 스페이스 한 개로 분리 되어 있으며, 요청줄은 캐리지 리턴 줄바꿈(CRLF) 문자열로 끝난다.
- 메시지 헤더들을 읽는다. 각 메시지 헤더는 CRLF로 끝난다.
- 헤더의 끝을 의미하는 CRLF로 끝나는 빈 줄을 찾아낸다.
- 요청 본문이 있다면, 읽어들인다.(길이는 Content-Length 헤더로 정의된다)

네트워크 커넥션은 언제라도 무효화될 수 있기 때문에 웹 서버는 파싱해서 이해하는 것이 가능한 수준의 분량읗확보할 때까지 데이터를 네트워크로부터 읽어서 메시지 일부분을 메모리에 임시로 저장해 둘 필요가 있다.

### 메시지의 내부 표현
몇몇 웹 서버는 요청 메시지를 쉽게 다룰 수 있도록 내부의 자료 구조에 저장한다.

### 커넥션 입/출력 처리 아키텍처
고성능 웹 서버는 수천 개의 커넥션을 동시에 열 수 있도록 지원한다.

웹 서버들은 항상 새 요청을 주시한다. 웹 서버 아키텍처의 차이에 따라 요청을 처리하는 방식도 달라진다.

(a) 단일 스레드 웹 서버

(b) 멀티프로세스와 멀티스레드 웹서버

(c) 다중 I/O 서버

대량의 커넥션을 지원하기 위해
모든 커넥션은 동시에 그 활동을 감시당한다.
스레드와 프로세스는 유휴 상태의 커넥션에 매여 기다리느라 리소스를 낭비하지 않는다.

(d) 다중 멀티스레드 웹 서버

몇몇 시스템은 자신의 컴퓨터 플랫폼에 올라와 있는 CPU 여러 개의 이점을 살리기 위해 멀티스레딩 다중화를 결합한다.
여러개의 스레드는 각각 열려있는 커넥션을 감시하고 각 커넥션에 대해 조금씩 작업을 수행한다.

## 3. 요청처리

웹 서버가 요청을 받으면, 서버는 요청으로부터 메서드, 리소스, 헤더, 본문을 얻어 처리한다.

POST를 비롯한 몇몇 메서드는 요청 메시지에 엔티티 본문이 있을 것을 요구한다.
그 외 OPTIONS를 비롯한 다수의 메서드는 요청에 본문이 있는 것을 허용하되 요구하지는 않는다.
많지는 않으나 GET 같이 요청 메시지에 엔티티 본문이 있는 것을 금지하는 메서드도 있다.

## 4. 리소스의 매핑과 접근
웹 서버는 리소스 서버다. HTML, JPEG 이미지 같은 미리 만들어진 콘텐츠를 제공하며, 서버 위에서 동작하는 리소스 생성 애플리케이션을 통해 만들어진 동적 콘텐츠도 제공한다.

웹 서버가 클라이언트에게 알맞은 콘텐츠를 전달하려면 그전에 요청메시지의 URI에 대응하는 콘텐츠 또는 콘텐츠 생성기를 웹 서버에서 찾아서 그 콘텐츠의 원천을 식별해야 한다.

### Docroot
웹 서버는 여러 종류의 리소스 매핑을 지원하지만, 가장 단순한 방법은 요청 URI를 웹 서버의 파일 시스템 안에 있는 파일 이름으로 사용하는 것이다.

일반적으로 웹 서버 파일 시스템의 특별한 폴더를 웹 콘텐츠를 위해 예약해 둔다. 이 폴더는 루트 혹은 docroot로 불린다.
웹 서버는 요청 메시지에서 URI를 가져와 문서 루트 뒤에 붙인다.
httpd.conf 와 같은 내부 설정파일을 이용하여 웹 서버의 문서 루트를 설정할 수 있다.
서버는 docroot 이외의 다른부분이 노출되지 않도록 주의해야 한다.

ex) 아파치 httpd.conf 파일을 보면 DocumentRoot가 정의되있는 부분을 보실 수 있을 겁니다.

### 가상 호스팅된 docroot
가상 호스팅 웹 서버는 각 사이트에 그들만의 분리된 문서 루트를 주는 방법으로 한 웹 서버에 여러 개의 웹 사이트를 호스팅 한다.
가상 호스팅 웹 서버는 URI나 Host 헤더에서 얻은 IP 주소나 호스트 명을 이용해 올바른 문서 루트를 식별한다.
하나의 웹 서버 웨어서 두 개의 사이트가 완전 분리된 콘텐츠를 갖고 호스팅 되도록 할 수 있다.
가상으로 호스팅 되는 docroot 설정은 아파치 웹 서버 같은 경우 VirtualHost 블록에서 DocumentRoot 지시자를 포함하도록 설정한다.

https://httpd.apache.org/docs/2.4/ko/vhosts/examples.html

### 사용자 홈 디렉터리 docroots
docroot의 또 다른 대표적인 활용은 사용자들이 한 대의 웹 서버에서 각자 개인의 웹 사이트를 만들 수 있도록 해주는 것이다.
빗금과 물결표 다음에 사용자 이름이 오는 것으로 시작하는 URI는 사용자 개인 문서 루트를 가리킨다.

### 디렉터리 목록
웹 서버는 경로가 파일이 아닌 디렉터리를 가리키는, 디렉터리 URL에 대한 요청을 받을 수 있다.

클라이언트가 디렉터리 URL을 요청했을 때 몇 가지 다른 행동을 취하도록 설정할 수 있다.

에러를 반환
디렉터리 대신 특별한 색인 파일을 반환
디렉터리를 탐색해서 그 내용을 담은 HTML 페이지를 반환

### 동적 콘텐츠 리소스 매핑
웹 서버와 벡엔드 애플리케이션과 연결하여 동적으로 리소스를 식별하여 매핑한다.
EX) 자바 서블릿

### 서버사이드 인클루드(Server-Side Includes, SSI)
서버는 콘텐츠에 변수 이름이나 내장된 스크립트가 될 수 있는 특별한 패턴이 있는지 검사
특별한 패턴은 변수 값이나 실행 가능한 스크립트의 출력 값으로 치환
동적 콘텐츠를 만드는 쉬운 방법

### 접근 제어
웹 서버는 클라이언트 IP에 근거하여 리소스에 접근하기 위한 비밀번호를 물어 볼수도 있음

## 5. 응답 만들기
응답 메시지는 응답 상태 코드, 응답 헤더, 응답 본문을 포함한다.

### 응답 엔티티
본문이 있다면 응답 메시지는 다음을 포함한다.

응답 본문의 MIME 타입을 서술하는 Content-Type 헤더
응답 본문 길이를 서술하는 Content-Length 헤더
실제 응답 본문의 내용

### MIME 타입 결정하기
웹 서버에게는 응답 본문의 MIME 타입을 결정해야 하는 책임이 있다.

다음은 MIME 타입과 리소스를 연결하는 여러가지 방법이다.

- mime.types    
    > 웹 서버는 MIME 타입을 나타내기 위해 파일 이름의 확장자를 사용할 수 있다. 이런 확장자 기반 타입 연계가 가장 흔한 방법이다.

- 매직 타이핑
    > 각 파일의 MIME 타입을 알아내기 위해 파일의 내용을 검사해서 알려진 패턴에 대한 테이블에 해당하는 패턴이 있는지 찾는다.
    
    > 느리긴 하지만 파일이 표준 확장자 없이 이름이 지어진 경우는 특히 편리하다.

- 유형 명시
    >  파일의 확장자 내용에 상관없이 어떤 MIME 타입을 갖도록 웹 서버를 설정할 수 있다.

- 유형 협상
    > 한 리소스가 여러 종류의 문서 형식에 속하도록 설정할 수 있다.
특정 파일이 특정 MIME 타입을 갖게 끔 설정할 수도 있다.

### 리다이렉션
웹 서버는 요청을 수행하기 위해 브라우저가 다른 곳으로 가도록 리다이렉트 할 수 있다.

Location 응답 헤더는 콘텐츠의 새로운 혹은 선호하는 위치에 대한 URI를 포함한다.

리다이렉트는 다음의 경우에 유용하다.

- 영구히 리소스가 옮겨진 경우

- 임시로 리소스가 옮겨진 경우

- URL 증강
  > 서버는 종종 문맥 정보를 포함시키기 위해 재 작성된 URL로 리다이렉트를 한다.
트랜잭션간 상태를 유지하는 유용한 방법이다

- 부하 균형
    > 과부화된 서버가 요청을 받으면, 서버는 클라이언트를 좀 덜 부하가 걸린 서버로 리다이텍트할 수 있다.

- 친밀한 다른 서버가 있을때

- 디렉터리 이름 정규화
    > 클라이언트가 디렉터리 이름에 대한 URI를 요청하는데 빗금을 빠트렸다면, 대부분의 웹 서버는 상대경로가 정상적으로 동작할 수 있도록 클라이언트를 슬래시를 추가한 URI로 리다이렉트한다.
​

## 6. 응답 보내기
서버는 받을때와 마찬가지로 커넥션 너머로 데이터를 보낼 때도 비슷한 이슈에 직면한다.

서버는 커넥션 상태를 추적해야 하며, 지속커넥션인 경우 특별히 주의해야 한다.

비지속 커넥션이면 모든 메시지를 전송후 자신쪽의 커넥션을 닫을 것이다.

지속커넥션 이라면, Content-Length 헤더를 바르게 계산하기 위해 특별한 주의를 필요로 하는 경우, 클라이언트가 응답이 언제 끝나는지 알 수 없는 경우 커넥션은 열린 상태를 유지할 것이다.