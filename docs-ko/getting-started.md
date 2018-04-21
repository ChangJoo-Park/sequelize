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

Sequelize will setup a connection pool on initialization so you should ideally only ever create one instance per database if you're connecting to the DB from a single process. If you're connecting to the DB from multiple processes, you'll have to create one instance per process, but each instance should have a maximum connection pool size of "max connection pool size divided by number of instances".  So, if you wanted a max connection pool size of 90 and you had 3 worker processes, each process's instance should have a max connection pool size of 30.

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

  // SQLite only
  storage: 'path/to/database.sqlite'
});

// Or you can simply use a connection uri
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

The Sequelize constructor takes a whole slew of options that are available via the [API reference](/class/lib/sequelize.js%7ESequelize.html).

## 커넥션 테스트하기

You can use the `.authenticate()` function like this to test the connection.

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

Models are defined with `sequelize.define('name', {attributes}, {options})`.

```js
const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING
  },
  lastName: {
    type: Sequelize.STRING
  }
});

// force: true will drop the table if it already exists
User.sync({force: true}).then(() => {
  // Table created
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```

You can read more about creating models at [Model API reference](/class/lib/model.js%7EModel.html)

## 첫번째 쿼리 만들기

```js
User.findAll().then(users => {
  console.log(users)
})
```

You can read more about finder functions on models like `.findAll()` at [Data retrieval](/manual/tutorial/models-usage.html#data-retrieval-finders) or how to do specific queries like `WHERE` and `JSONB` at [Querying](/manual/tutorial/querying.html).

### 애플리케이션 단위 모델 설정

The Sequelize constructor takes a `define` option which will be used as the default options for all defined models.

```js
const sequelize = new Sequelize('connectionUri', {
  define: {
    timestamps: false // true by default
  }
});

const User = sequelize.define('user', {}); // timestamps is false by default
const Post = sequelize.define('post', {}, {
  timestamps: true // timestamps will now be true
});
```

## 비동기를 위한 Promises

Sequelize uses [Bluebird](http://bluebirdjs.com) promises to control async control-flow.

**Note:** *Sequelize use independent copy of Bluebird instance. You can access it using
`Sequelize.Promise` if you want to set any Bluebird specific options*

If you are unfamiliar with how promises work, don't worry, you can read up on them [here](http://bluebirdjs.com/docs/why-promises.html).

Basically, a promise represents a value which will be present at some point - "I promise you I will give you a result or an error at some point". This means that

```js
// DON'T DO THIS
user = User.findOne()

console.log(user.get('firstName'));
```

*will never work!* This is because `user` is a promise object, not a data row from the DB. The right way to do it is:

```js
User.findOne().then(user => {
  console.log(user.get('firstName'));
});
```

When your environment or transpiler supports [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) this will work but only in the body of an [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) function:

```js
user = await User.findOne()

console.log(user.get('firstName'));
```

Once you've got the hang of what promises are and how they work, use the [bluebird API reference](http://bluebirdjs.com/docs/api-reference.html) as your go-to tool. In particular, you'll probably be using [`.all`](http://bluebirdjs.com/docs/api/promise.all.html) a lot.
