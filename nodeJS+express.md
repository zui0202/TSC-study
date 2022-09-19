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


## 3. EC2 배포하기

