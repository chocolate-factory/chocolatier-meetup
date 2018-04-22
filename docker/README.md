

# What is Docker?

도커는 **컨테이너 기반의 오픈소스 가상화 플랫폼** 

Virtual Machine과 차별화 되는 점이 가상화를 하는 과정에서 발생하는 OverHead가 없습니다. 

![what-is-docker](./image/what-is-docker.png)

# Why use Docker?

1. **Reproducibility**

   1. Docker를 사용하는 환경에 관계 없이 언제나 똑같은 컨테이너를 어디서든 실행 가능
   2. Dev, Prod, Test, Staging

2. **Isolation**

   1. Host OS에 깔려있는 다른 응용프로그램등에 영향을 받지 않는다.
   2. 각 컨테이너가 독립적으로 동작

3. **Security**

   1. 최소한의 패키지로 구동하는데 최적화
   2. 하나의 컨테이너가 공격받아도 나머지 컨테이너는 안전

4. **Docker Hub**

   1. 이미 만들어진(심지어 잘) Docker Image들을 자유롭게 활용!

5. **Environment Management**

   1. 개발환경, 테스트 환경, production 환경 모두에서 같은 Docker Image로 사용 가능

6. **CI/CD**

   1. Travis, Jenkins, and Gitlab CI 등 다양한 CI/CD 툴에서 Docker를 지원



# So, How I use Docker?

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



# Write your Dockerfile

시나리오. 특정 게시물의 조회수를 저장하고 조회시마다 조회수를 올리는 API를 개발!

데이터베이스로는 Redis를 사용 App은 node를 사용 패키지 관리는 yarn을 사용해봅시다.

```
yarn init
yarn add redis
```

```javascript
console.log('start server');
var http = require("http");
var redis = require("redis");
var redisUrl = process.env.REDIS_URL;
console.log('start to connect redis: ' + redisUrl);
var client = redis.createClient({
    url: redisUrl
});
console.log('redis was connected');
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

[참고자료](https://github.com/chocolate-factory/docker-compose-example)

