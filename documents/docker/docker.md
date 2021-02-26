최고의 Dockerfile 작성을 위한 몇가지 습관들
https://velog.io/@seheon99/%EC%B5%9C%EA%B3%A0%EC%9D%98-Dockerfile-%EC%9E%91%EC%84%B1%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%AA%87-%EA%B0%80%EC%A7%80-%EC%8A%B5%EA%B4%80%EB%93%A4

context: 현재 디렉토리 (빌드 컨텍스트)
context에 Dockerfile이 있다고 가정
-f 를 통해 다른 곳에 있는 Dockerfile 지정가능
-f를 써도 현재 컨텍스트는 Docker에 전달

--no-cache를 이용하면 마지막 빌드 캐시에 의존하지 않는다

.dockerignore 파일을 이용해 제외할 파일 설정
제외할 파일 설정이 필요한 이유: 필요없는거 빌드하면 속도와 크기에서 손해

RUN COPY ADD 는 세 가지 명령어는 레이어를 만든다.

한줄말고 여러줄로 정리하는게 보기가 편하다

Alpine이미지를 애용하라
LABEL을 저장해라

apt-get 관련하여 update와 install 은 같이 사용해야한다.


ADD는 COPY와 다르게 tar.gz 파일을 압축해제하는등에 쓰이기 때문에 명확하게 COPY 사용

WORKDIR는 항상 절대경로로.
