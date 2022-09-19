# TSC-study
## 1. 프로젝트 만들기
https://www.youtube.com/watch?v=7CqJlxBYj-M&t=28s 참고

### MongoDB Atlas
이 프로젝트에서 DB는 mongoDB를 사용한다. mongoDB atlas는 DB를 통합관리할 수 있는 클라우드 DB이다.

https://www.mongodb.com/cloud/atlas/register
위 사이트에서 로그인(회원가입) 후 클러스터를 생성하고 connection string을 알아둔다.

```
mongodb+srv://zui0202:<password>@cluster0.xmkh2bm.mongodb.net/?retryWrites=true&w=majority
```
버전마다 조금씩 다르겠지만 이와 비슷한 형태일 것이다.

### Back end

```
$ npm install express cors mongoose dotenv
```
필요한 것들(express, cors, mongoose, dotenv)을 설치해준다.

+) 추가
```
$ npm install -g nodemon
```
nodemon을 설치하면 수정사항이 생겼을 때 서버를 재시작할 필요가 없어서 편리하다.

```

const express = require('express');
const cors = require('cors');
const mongoose = require(‘mongoose’);

require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

const uri = process.env.ATLAS_URI;
mongoose.connect(uri, { useNewUrlParser: true, useCreateIndex: true }
);
const connection = mongoose.connection;
connection.once('open', () => {
  console.log("MongoDB database connection established successfully");
})

app.listen(port, () => {
    console.log(`Server is running on port: ${port}`);
});
```
서버를 실행할 파일을 생성해준다. 같은 위치에 .env파일을 만들어주고 mongoDB atlas에서 기억해둔 connection string을 ATLAS_URI로 설정해준다.

### Database Schema

DB 스키마를 생성한다.
다음은 user 모델의 예시이다.

```

const mongoose = require('mongoose');

const Schema = mongoose.Schema;

const userSchema = new Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    minlength: 3
  },
}, {
  timestamps: true,
});

const User = mongoose.model('User', userSchema);

module.exports = User;
```

## 2. 도커에 올리기

### Dockerfile
```python
# FROM <IMAGE_NAME>
FROM node:10

# WORKDIR <이동할 경로>
# cd <DIR_NAME>
WORKDIR /usr/src/app

# COPY <src>... <dest>
# 호스트 컴퓨터에 있는 디렉터리나 파일을 Docker 이미지의 파일 시스템으로 복사
COPY package*.json ./
# ADD는 강력한 버전의 COPY
# ADD <src>... <dest>

# RUN <COMMAND>
RUN npm install

# 이미지를 빌드한 디렉터리의 모든 파일을 컨테이너의 usr/src/app 디렉터리로 복사
# 아까 usr/src/app 디렉터리로 이동했기 때문
COPY . .

# 3000/TCP 포트로 리스닝
# 지정하지 않으면 TCP가 기본값으로 사용
EXPOSE 3000

# ENTRYPOINT ["<커맨드>", "<파라미터1>", "<파라미터2>"]
# 명령문은 이미지를 컨테이너로 띄울 때 항상 실행되야 하는 커맨드를 지정할 때 사용

# 해당 이미지를 컨테이너로 띄울 때 디폴트로 실행할 커맨드
# ENTRYPOINT 명령문으로 지정된 커맨드에 디폴트로 넘길 파라미터를 지정할 때 
ENTRYPOINT [ "npm" ]
CMD ["start"]

# ENV <키>=<값>
ENV NODE_ENV production

# ARG <이름>
# ARG <이름>=<기본 값>
ARG port=8080
```

### docker-compose.yml

```python
version: "3"
services:
  app:
    # 컨테이너 이름
    container_name: docker-node-mongo
    # 컨테이너가 실행 중 중단됐을 때 컨테이너를 다시 알아서 재시작
    restart: always
    # Dockerfile이 있는 위치
    build: .
    # <내부에서 개방할 포트>:<외부에서 접근할 포트>
    ports:
      - "3000:3000"
    # 컨테이너를 compose 파일 외부에 있는 서비스와 연결
    external_links:
      - mongo
  mongo:
    container_name: mongo
    image: mongo
    ports:
      - "27017:27017"

```

### 도커 명령어
```
// 이미지 목록 보기
$ sudo docker images

// 이미지 검색
$ sudo docker search <이미지 이름>

// 이미지 받기
$ sudo docker pull <이미지 이름>:<버전>

// 이미지 삭제
$ sudo docker rmi <이미지 id>

// 컨테이너까지 강제 삭제
$ sudo docker rmi -f <이미지 id>

// 컨테이너 목록 보기
$ sudo docker ps

// 컨테이너 실행
$ sudo docker run [options] image[:TAG|@DIGEST] [COMMAND] [ARG...]

// 컨테이너 시작
$ sudo docker start [컨테이너 id 또는 name]

// 컨테이너 재시작
$ sudo docker restart [컨테이너 id 또는 name]

// 컨테이너 접속
$ sudo docker attach [컨테이너 id 또는 name]

// 컨테이너 정지
$ sudo docker stop [컨테이너 id 또는 name]

// 컨테이너 삭제
$ sudo docker rm [컨테이너 id 또는 name]
```


## 3. EC2 배포하기
### 인스턴스 생성
https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateWebServer.html 참고

### 인스턴스 연결
키페어를 받고 해당 pem 파일의 권한을 조정한다.
```
chmode 400 키파일_이름.pem
```

ssh 명령으로 EC2 인스턴스에 접속한다.
```
ssh -i 키파일_이름.pem ubuntu@퍼블릭_ip_또는_퍼블릭_DNS
```

### ubuntu 세팅, nodeJs, pm2, nginx 설치
기본 업데이트
```
$ sudo apt-get update
```

다음 명령어를 입력한다.
```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```
완료되면 nvm이 설치되었는지 버전 확인

```
$ nvm --verion
```

nvm 명령어로 node와 npm을 설치한다.
```
$ nvm install 10.16.3
```

pm2를 전역으로 설치
```
$ npm install pm2 -g
```

nginx 설치
```
$ sudo apt-get install nginx
```


### 프로젝트 올리기


### nginx 프록시 서버 설정하기


