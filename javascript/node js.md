# Node Js 





### **Init**

```bash
npm init
npm install express --save # 스프링의 디스패처 서블릿, HTTP기반 API 프레임워크
npm install body-parser --save # body에 접근하기 위한 미들웨어
```



### **서버 세팅 및 구동**

```javascript
const express = require('express');
const app = express();

app.listen(${port_number}, () => { // app(port 번호, callback함수)
  console.log("Server is running ");
})

app.get(${url}, (req, res) => { // GET방식으로 ${url}에 라우팅
  res.send("Hello World!\n");
})
```

express에서 서버의 기능을 미들웨어 형태로 제공, 이 미들웨어는 express 인스턴스의 `use()` 를 사용하여 추가 할 수 있다.



### **url 맵핑**

url속에 **:변수명** 을 통해 url로 변수를 받을 수 있고, 메소드 내에서 **req.params.변수명** 으로 꺼내서 사용할 수 있다.

```javascript
app.get('/users/:id', (req, res) => {
	console.log(req.params.id);
});
```



### **body-parser**

express에 요청 바디의 데이터에 접근하기 위해서는 body-parser라는 미들웨어가 필요

```javascript
const bodyParser = require('body-parser');

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true}));
```



### 라우팅 모듈 만들기 및 사용하기

**라우팅 모듈 생성**

```javascript
// api/users/controller.js
const express = require('express');
const router = express.Router();

exports.index = (req,res) => {
	//..
};
//...

```

```javascript
// api/users/index.js
const express = require('express');
const router = express.Router();
const controller = require('./user.controller');

router.get('/users', controller.index);
```



**라우팅 모듈 사용**

```javascript
// app.js
const express = require('express');
const app = express();
app.use('/users', require('./api/user')); //'/users'로 들어오는 요청에 대해 위임
```



### ORM - Model

```bash
npm install Sequelize --save
```

```javascript
const Sequelize = require('sequlize');
const sequelize = new Sequlize(${데이터베이스 이름}, ${데이터베이스 계정}, ${비밀번호})
```



### **Mongo DB**

**Connect**

```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/testDB');
```



**Model 생성**

```javascript
const mongoose =require('mongoose');
const WaitingDataSchema = new mongoose.Schema({
    college_key: Number, // 학교 키
    balance_amt : Number, // 통장 잔액
    transaction_amt : Number, // 거래 금액
    transaction_type : String, // 거래 타입 : 입금/출금
    content : String, // 거래내용
    created_date : Date
});
mongoose.model('WaitingData',WaitingDataSchema);

module.exports = mongoose.model('WaitingData');
```



### Promise

**promise란 미래 결과에 대한 약속**이다. 이 결과가 성공인지 실패인지에 따라서 이를 핸들링하기 위한 로직을 정의해놓는다. 순서를 정리하자면 아래와 같다.

1. 비동기 함수에게 처리를 요청한다.
2. 비동기함수는 처리에 대한 결과 대신 promise 객체를 리턴한다.
3. promise를 리턴받은 사용자는 성공/실패 여부에 따라 핸들링 할 수 있는 로직을 promise에 작성한다.
4. 비동기함수가 실행을 마치면, 3번에서 정의해놓은 로직을 수행한다.

위 과정을 코드로 살펴보자 

```javascript
var promise = async_fuction(param);
promise.then((result) => 성공로직, (err) => 실패로직);
```

비동기 함수 내부를 살펴보자

```javascript
function async_function(param) {
		return new Promise(resolvec, rejected){
				if(성공 검증){
					resolved("결과"); // "결과"는 promise.then에서 성공로직을 담당하는 함수의 파라미터가 된다.
				} else{
          rejected(Error(err));
        }
		}
}
```

pomise는 chaining을 지원한다.  chaining을 이용해서 여러개의 비동기 작업들을 순차적으로 수행 할 수 있다.

보다 자세한 정보는 [조대협씨 블로그](https://bcho.tistory.com/1086) 를 참고하자.



### asycn / await

함수 내부에서 비동기 처리를 동기적으로 풀 필요가 있을 때, 가장 바깥 함수에 asycn 키워드를, 동기로 풀고싶은 비동기 함수에 await 키워드를 사용한다.

### Require

- 다른 패키지 경로 잡아주기

  root

  ​	common / constant.js

  ​	wating_data/ WatingDataController.js

   WatingDataController.js에서 contant.js 가져오기

  ```javascript
  require('./../common/constant.js') # ./ - wating_data, ./../ - root
  ```



### Request

- Methods
  - query : GET 방식으로 넘어오는 쿼리 스트링 파라미터를 담고 있다.
  - body : POST 방식으로 넘어오는 파라미터를 담고 있다.  HTTP의 body 부분에 담겨져 있는데, 파싱을 위해 body-parser와 같은 패키지가 필요하다
  - headers : HTTP 헤더 정보를 가지고 있다.



### Repsonse

- Methods 
  - status : 응답코드 설정
  - send(body) : 클라이언트에 응답을 보낸다. 기본 콘텐츠 타입은 text/html 이다
  - josn(json) : 클라이언트로 json값을 보냄
  - render(view, [locals], callback) : locals는 뷰를 렌더링하는 기본 콘텍스트를 포함하는 객체, 뷰를 렌더링한다.