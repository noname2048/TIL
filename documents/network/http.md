# HTTP

## 모든-개발자를-위한-HTTP-웹-기본-지식-인프런을-듣고

새로 얻거나 중요하다고 생각했던 것들

* 리다이렉트 관련 상태코드 중에는 리다이렉트시 method 사항이 다른 3가지가 있다.
  Location 헤더에 URL를 정보가 담겨있다.
  1. 같거나 get을 권장하기 (may, 주로 get으로 동작)
  2. get을 명시하기
  3. 동일 method를 사용하기
* 프록시 캐시라는 것이 존재할 수 있고, CDN의 기초이다. 
* 캐시는 만료일자나 버전정보를 설정할 수 있다.
* 원서버와 비교하거나, 프록시 캐시랑 비교하거나, 컴퓨터의 캐시를 사용한다.
*  HTTP3는 UDP를 기준으로 나아가고 있다.
* 이미 네이버와 구글은 HTTP2가 기준이고 , HTTP3을 일부 도입.
* URI는 URL과 URN을 포괄하는 개념이지만, 주로 URL 기준으로 사용한다.
* TCP 전송에서 데이터를 자르는 보편적인 크기는 1500 BYTE 이다.
* TCP가 connect 되었다는건 실제로 연결하는게 아니고, 3 way handshake 를 나눈것 뿐이다.
* checksum 에 대한 부분은 TCP/UDP 계층에서 주로 다룬다.
* PORT 정보역시 TCP/UDP 계층에서 나온다.
* HTTP 메시지는 시작줄, 헤더필드, 빈칸, 공백 으로 구성된다.
* RESTful API는 resource와 method를 적절히 다뤄야만 잘 구성할 수 있다.
* 콘텐츠 협상이라는 개념이 있다. 보통 지역언어, 인코딩셋 등을 설정한다.
* REST 에서 R은 representation(표현) 을 말한다.



### 상태코드들

* 200 은 주로 리소스 GET 요청시 OK 반환
* 201 POST와 같은 정보를 받아 리소스 생성시 Created 반환
* 202 정보는 받았으나 처리가 이후에 일어나는 일일때 Accepted 반환
* 204 정보만 받고 더 이상의 처리가 불필요할때 No Content 반환
* 300 잘안쓴다
* 301 검색엔진에서도 정보를 바꿔주는 Moved Permantly
  GET으로 바뀌고 본문이 제거될 수 있다. (옛날 방식 -> 308)
* 302 예전에 쓰던 방식으로, 자연스럽게 화면을 전환한다. Found
  GET으로 바뀌고 본문이 제거될 수 있다. (옛날 방식 -> 303, 307)
* 303 리다이렉트시 GET으로 메소드 변경 See Other
* 307 리다이렉트시 반드시 메소드와 본문 유지 Temporary Redirect
* 308 리다이렉트시 반드시 메소드와 본문을 유지 permantly Redirect
* 304 Not Modified 캐시가 변경되지 않았다
* 400 Bad Request 클라이언트가 잘못된 요청을 보냄
* 401 Unauthorized 인증 (로그인이 안됨)
  authentication : 로그인, Authorization:  권한 이지만 용어가 잘못되었다고 한다.
* 403 Forbidden 허락되지 않음 (주로 권한 문제)
* 404 Not Found 리소스가 서버에 없음
* 500 Server Error 서버에 심각한 오류가 발생함.
* 503 Service Unavailable 서버 점검이나 예상이 가능한 시간 이후에 복구되는 경우 (Retry-After로 시간 기입)