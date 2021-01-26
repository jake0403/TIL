## Docker

내부 포트 8080로 외부에서 연결을 하고자 하는 것은 port fowarding 이라고 한다.



`docker`

docker run -p 8080 example/echo:latest

docker ps



docker stop (id)



docker run -p 9000:8080 example/echo:latest

(9000번이 8080으로 호출이 된다.)



docker run -it golang:1.9 bash



docker container run -d(demo 백그라운드에서) -p(포트)

docker run -d -p 13306:3306



ocker run -d -p 13306:3306



컨데이너가 삭제 되어도 이미지는 영향 x

기존에 이미지 다운받은게 있기 때문에