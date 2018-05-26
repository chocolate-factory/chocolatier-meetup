# Docker가 뭔가요?

![docker 아주 귀엽다](./image/docker.png)

우리의 친구 위키피디아를 찾아보자.

> **Docker** is a [computer program](https://en.wikipedia.org/wiki/Computer_program) that performs [operating-system-level virtualization](https://en.wikipedia.org/wiki/Operating-system-level_virtualization) also known as [containerization](https://en.wikipedia.org/wiki/Operating-system-level_virtualization).
>
> [- Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))

위키피디아에서는 Docker를 컨테이너라고 알려진 OS 수준 가상화를 제공하는 프로그램이라고 정의하고 있다. 가상화하면 VM인데 컨테이너는 또 뭘까? 우리가 아는 컨테이너는 화물선에 실리는 화물 컨테이너뿐인데 말이다. 컨테이너 가상화도 화물 컨테이너처럼 규격화되고 추상화해줘서 Host 머신과 상관없이 구동할 수 있게 해주는 프로그램이다.

# Docker를 왜 쓰나요?

### 재사용

Docker를 사용하는 환경에 관계없이 같은 이미지를 이용해 컨테이너를 어디서든 실행 가능하다. 내 노트북 위의 개발 환경, CI의 테스트, 스테이징 서버, 실 서비스 환경에서 같은 Docker 이미지를 사용해 같은 컨테이너를 실행한다는 것이다. Docker 없이 실 서버에 바로 배포를 하는 경우 의존성이 있는 소프트웨어의 로컬 환경과 개발 환경 간의 버전 차이, 설정 차이 등이 발생할 수 있는데 이 부분을 최소화할 수 있다. 이는 테스트 환경에선 만나지 못했는데 실 서비스 환경에서 문제를 만날 확률을 줄여준다.

### 격리

Docker 컨테이너는 Host OS의 커널을 공유하지만 실행 영역을 공유하지 않기 때문에 Host OS의 응용프로그램이나 다른 컨테이너의 영향을 받지 않고 각 컨테이너가 독립적으로 동작한다.

### Docker Hub

https://hub.docker.com github처럼 Docker 이미지를 공유할 수 있는 서비스다. 이곳에서 이미 만들어진 Docker 이미지를 pull해서 사용할 수 있다.  굉장히 다양한 소프트웨어의 Docker 이미지를 사용할 수 있다. github에 Dockerfile이 공개되어있는 오픈소스의 경우 대부분 Docker Hub에서 찾을 수 있다.

### CI/CD

Docker 컨테이너는 사실상의 표준이다. 현재 Travis, Jenkins, Gitlab CI, Circle CI 등 다양한 CI/CD 툴에서 Docker를 지원한다. 이런 다양한 Managed CI/CD 시스템중 하나를 사용한다면 복잡한 설정 없이 Docker를 바로 사용할 수 있다. 

# 그래서 그 좋은 Docker는 어떻게 쓰는 거죠?

시작하기 전에 몇가지 개념을 짚고 넘어가자. 먼저 도커 이미지, 컨테이너가 있다. 이미지는 docker build 커맨드를 통해서 만들어진 스냅샷이다. 그리고 컨테이너는 이미지를 통해 만들어진 인스턴스이다. OOP에서 Class와 객체와 같은 개념이라고 생각하면 편하게 이해할 수 있다.

 `Hello World`를 찍어보자.([Docker 설치](https://www.docker.com/community-edition#/download)는 되어있다고 가정한다.) `docker run docker:latest echo "Hello World"`를 입력하면 아래와 같은 화면이 나온다. 아래 Hello world 명령어를 보면서 Docker 명령어를 뜯어보자.

```sh
$ docker run docker:latest echo "Hello World"
Unable to find image 'docker:latest' locally
latest: Pulling from library/docker
ff3a5c916c92: Pull complete
1a649ea86bca: Pull complete
ce35f4d5f86a: Pull complete
e6fece0477c1: Pull complete
550c91598bd2: Pull complete
7ff0c6a709f7: Pull complete
Digest: sha256:8879659d45d4b2115f9b67ec18ca29712ad79dff889763e7b8d3377d4373968b
Status: Downloaded newer image for docker:latest
Hello World
```

이 커맨드는 Docker라는 이미지의 latest 버전 이미지를 사용하여 `echo "Hello world"`라는 명령어를 실행하라는 커맨드다. 한 부분씩 뜯어보면 순서대로 Docker 컨테이너를 실행하라는 `run` 커맨드 어떤 이미지를 사용할지 명시하는 부분  `docker:latest` 이미지 이름:태그명 어떤 명령어를 사용할지 명시하는 `echo "Hello World"`실행할 명령어 이렇게 구성되어 있다.

# 제 어플리케이션을 어떻게 Docker 이미지로 만들죠?

먼저 어떤 어플리케이션을 만들지 시나리오를 정해보자. 아래 요구사항의 간단한 조회수 어플리케이션을 만들고 Docker 이미지로 만들어보자.

1. 특정 URL로 들어오는 조회 수를 저장
2. 해당 URL로 접속이 들어올 때마다 조회수를 1씩 증가
3. 페이지에 대한 응답으로는 조회수를 응답

조회 수를 저장할 데이터베이스로는 Redis를 사용하고 패키지 관리는 yarn을 사용했다.

```sh
# redis-server install
# mac에서 redis server install하기
$ brew install redis 

# 데비안에서 redis server install
$ sudo apt-get install redis-server -y

# yarn 패키지 초기화 및 redis javascript 클라이언트 설치
$ yarn init
$ yarn add redis
```

`index.js` 파일을 작성해보자. 간단히 내용을 설명하면 `REDIS_URL`이라는 환경 변수에서 Redis 접속 URL을 받아 URL 기반의 redis key로 조회 수를 count하고 반환하는 어플리케이션이다. 개선해야 할 문제가 몇 가지 있지만 간단한 예제로써 바라보자.

```javascript
console.log('start server');
var http = require("http");
var redis = require("redis");
var redisUrl = process.env.REDIS_URL;
console.log('start to connect redis: ' + redisUrl);
var client = redis.createClient({
    url: redisUrl
});
http.createServer(function (req, res) {
    res.writeHead(200, { "Content-Type": "application/json" });
    client.incr(req.url, function (error, value) {
        var viewCount = value;
        var response = { 'viewCount': viewCount };
        res.write(JSON.stringify(response));
        res.end();
    });
}).listen(8080);
```

이제 조회 수 Node js 어플리케이션을 돌려볼 시간이다. 

1. `redis-server` 명령어로 foreground에서 Redis 서버를 띄운다.

2. `REDIS_URL='redis://localhost' node index.js` 명령어로 어플리케이션을 실행시킨다.

3. `http://localhost:8080/1`로 접속해본다. 혹은 `curl http://localhost:8080/1` 명령어를 실행시킨다. 아래와 같은 값이 반환될 것이다. 여러번 실행하면 실행할 때마다 viewCount 값이 올라가는 것을 볼 수 있다.

   ```json
   {"viewCount":1}
   ```

이제 만들었던 어플리케이션을 돌아보자. 먼저 로컬에 Redis를 설치했다. 만약에 구동할 서버와 Redis 버전 등이 다르다면 다르게 동작할 가능성을 배제할 수 없다. 만약에 어떤 레거시 프로젝트에서는 Redis 1.2버전을 요구하고 이번에 새로 시작하는 프로젝트에서는 최신 버전인 4.0.9 버전을 요구한다면? 프로젝트마다 Redis 버전을 맞춰주는 수고를 해야 할 것이다. 이런 문제를 수월하게 해결하기 위해서 Docker를 활용해보자.

Dockerfile을 작성해보자. 

```dockerfile
FROM node:9.11

ENV REDIS_URL="redis://redis"
WORKDIR app
COPY ./package.json package.json
COPY ./yarn.lock yarn.lock
RUN yarn install
COPY ./index.js index.js
ENTRYPOINT ["node"]
CMD ["index.js"]
```

`FROM`은 어떤 base Image를 사용할지 정의하는 부분이다. 이 예제에서는 node 9.11버전의 [Docker Image](https://hub.docker.com/_/node/)를 사용한다. 이는 node 사용을 위해서 최소한의 패키지만 설치된 리눅스와 node 9.11 버전이 설치되어 있는 이미지이다. 위에서 언급한 대로 기본 설정은 Docker hub에서 가져온다. Docker hub private이나 ECR 등의 Private Registry에서 가져올 수도 있다.

`ENV`는 환경변수를 정의한다. REDIS_URL이라는 환경변수를 정의하고 기본값으로 `redis://redis`를 넣는다.

`WORKDIR`은 앞으로 실행할 커맨드들의 디렉터리를 명시한다. 아래 커맨드들은 /app 디렉터리에서 실행된다.

`COPY`는 호스트에 있는 파일을 복사한다. 

`RUN` 은 커맨드를 실행한다. 복사한 `package.json, yarn.lock` 을 이용하여 node 패키지를 설치한다.

`ENTRYPOINT`는 컨테이너가 시작했을 때 실행될 명령을 정의한다. 

`CMD`는 `docker run` 실행 시 기본 명령을 설정하거나, `ENTRYPOINT`의 기본 인자를 설정할 때 사용한다. 이 경우에는 `ENTRYPOINT`가 있기 때문에 기본 인자를 설정하는데 사용됐다.

이제 작성한 Dockerfile로 Docker Image를 빌드하고 컨테이너를 만들어보자!

```shell
$ docker build . -t mydocker:latest
Sending build context to Docker daemon  393.7kB
Step 1/9 : FROM node:9.11
 ---> aa3e171e4e95
Step 2/9 : ENV REDIS_URL="redis://heechan.local"
 ---> Running in e0f1070d2b1d
 ---> 4a4b78af15d7
Step 3/9 : WORKDIR app
 ---> 4df475788243
Step 4/9 : COPY ./package.json package.json
 ---> a6c91ba46484
Step 5/9 : COPY ./yarn.lock yarn.lock
 ---> 57d5e7322edd
Step 6/9 : RUN yarn install
 ---> Running in 9394ffb17054
yarn install v1.5.1
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 0.58s.
 ---> 57f3bc6b13a2
Step 7/9 : COPY ./index.js index.js
 ---> 861e3f8f5cd8
Step 8/9 : ENTRYPOINT ["node"]
 ---> Running in 5db0339737d2
 ---> c94f66c0a4a9
Step 9/9 : CMD ["index.js"]
 ---> Running in dd01fe57901b
 ---> d2edde90a736
Successfully built d2edde90a736
Successfully tagged mydocker:latest
```

`docker image ls`명령어를 사용해보면 latest 태그가 붙은 mydocker라는 이미지가 생성되어 있을 것이다.

이제 이 docker image로 컨테이너를 생성해보자. 

```sh
$ docker run -p 8080:8080 --env REDIS_URL=redis://heechan-macbook-13.local mydocker:latest
```

-p 옵션으로 localhost의 8080포트와 컨테이너의 8080포트를 바인딩 했고 REDIS_URL 환경 변수로 호스트 머신의 Redis 주소를 넘겼다.(redis-server 실행 시에 `--protected-mode no` 옵션을 줘야 합니다.) 

이제 `docker container ls` 명령을 사용하면 실행 중인 컨테이너를 확인할 수 있다. `docker container stop 컨테이너 ID` 명령어를 사용하면 컨테이너를 종료할 수도 있다. 