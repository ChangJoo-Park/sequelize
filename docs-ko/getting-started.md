# 시작하기

## 설치

Sequelize는 NPM과 Yarn 통해 설치할 수 있습니다.

```bash
// NPM
$ npm install --save sequelize

# 사용할 데이터베이스에 따라 선택하세요
$ npm install --save pg pg-hstore
$ npm install --save mysql2
$ npm install --save sqlite3
$ npm install --save tedious // MSSQL

// Yarn
$ yarn add sequelize

# And one of the following:
$ yarn add pg pg-hstore
$ yarn add mysql2
$ yarn add sqlite3
$ yarn add tedious // MSSQL
```

## 커넥션 설정하기

Sequelize는 초기화시 커넥션 풀을 설정하므로 하나의 프로세스에서 데이터베이스에 연결하는 경우 데이터베이스 당 하나의 인스턴스만 작성하는 것이 좋습니다. 여러 프로세스에서 데이터베이스에 연결해야하는 경우 프로세스 당 하나의 인스턴스를 만들어야하지만 각 인스턴스의 최대 커넥션 풀 크기는 "최대 커넥션 풀 크기를 인스턴스 수로 나눈 값"이어야 합니다. 따라서 최대 커넥션 풀 크기가 90이고 워커 프로세스가 3개인 경우 각 프로세스의 인스턴스는 최대 커넥션 풀 크기가 30이어야 합니다.

```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },

  // SQLite만 사용
  storage: 'path/to/database.sqlite'
});

// 또는 커넥션 URI를 이용
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

The Sequelize constructor takes a whole slew of options that are available via the [API reference](/class/lib/sequelize.js%7ESequelize.html).

## 커넥션 테스트하기

커넥션 테스트를 위해 `.authenticate()` 함수를 사용할 수 있습니다.

```js
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```

## 첫번째 모델 만들기

모델은 `sequelize.define('name', {attributes}, {options})`을 이용해 정의합니다.

```js
const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING
  },
  lastName: {
    type: Sequelize.STRING
  }
});

// force: true 는 테이블이 이미 있으면 drop합니다.
User.sync({force: true}).then(() => {
  // Table created
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```

모델을 만드는 자세한 내용은 [Model API 레퍼런스](/class/lib/model.js%7EModel.html)를 읽어보세요

## 첫번째 쿼리 만들기

```js
User.findAll().then(users => {
  console.log(users)
})
```

데이터 조회시 `.findAll()`과 같은 모델의 [데이터 조회](/manual/tutorial/models-usage.html#data-retrieval-finders) 또는 쿼리에 사용하는 `WHERE`와 `JSONB`를 사용할 수 있습니다.

### 애플리케이션 단위 모델 설정

Sequelize 생성자는 `define` 옵션으로 모델에 대한 기본 옵션을 지정할 수 있습니다.

```js
const sequelize = new Sequelize('connectionUri', {
  define: {
    timestamps: false // 기본값 true
  }
});

const User = sequelize.define('user', {}); // timestamps는 기본값 false
const Post = sequelize.define('post', {}, {
  timestamps: true // timestamps는 항상 true
});
```

## 비동기를 위한 Promises

Sequelize는 비동기 작업을 위해 [Bluebird](http://bluebirdjs.com) Promise를 사용합니다.

**Note:** *Sequelize는 독립적인 Bluebird 인스턴스를 사용합니다. Bluebird 설정을 하려면 `Sequelize.Promise` 를 이용하세요*

Promise에 아직 익숙하지 않다면 [이 글](http://bluebirdjs.com/docs/why-promises.html)을 읽어보세요

기본적으로 Promise는 특정 시점에 값을 전달하는 것을 말합니다. - 이 말은 "어떤 시점에 결과나 오류를 전달해줄게" 라는 말과 같습니다.

```js
// 이렇게 사용하지 마세요.
user = User.findOne()

console.log(user.get('firstName'));
```

*위 코드는 절대 작동하지 않습니다!* `user`는 데이터베이스의 데이터가 아닌 Promise 객체입니다. 아래 코드가 올바른 방법입니다.

```js
User.findOne().then(user => {
  console.log(user.get('firstName'));
});
```

개발 환경이나 트랜스파일러가 [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 를 지원하면  아래 코드는[async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 함수에서만 작동합니다.

```js
user = await User.findOne()

console.log(user.get('firstName'));
```

Promise가 무엇이고 어떻게 작동하는지 파악되었으면 [bluebird API 레퍼런스](http://bluebirdjs.com/docs/api-reference.html) 를 읽어보세요. [`.all`](http://bluebirdjs.com/docs/api/promise.all.html) 를 아마 많이 사용하게 될 것입니다.
