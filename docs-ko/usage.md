# 기본 사용 방법

시작하기 위해 먼저 Sequelize 인스턴스를 만들어야합니다. 다음과 같이 사용하세요.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
});
```

위 코드는 데이터베이스 자격증명을 저장하고 모든 추가 메소드를 제공합니다.

추가로, 기본 값을 사용하지 않고 호스트와 포트를 지정할 수 있습니다:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql',
  host: "my.server.tld",
  port: 9821,
})
```

비밀번호가 없는 경우:

```js
const sequelize = new Sequelize({
  database: 'db_name',
  username: 'username',
  password: null,
  dialect: 'mysql'
});
```

문자열로도 할 수 있습니다:

```js
const sequelize = new Sequelize('mysql://user:pass@example.com:9821/db_name', {
  // 다음 장에서 사용할 수 있는 옵션을 다룹니다
})
```

## 옵션

호스트와 포트 외에도 많은 옵션을 제공합니다. 

- See [Sequelize API](/class/lib/sequelize.js%7ESequelize.html)
- See [Model Definition](/manual/tutorial/models-definition.html#configuration)
- See [Transactions](/manual/tutorial/transactions.html)

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // the sql dialect of the database
  // currently supported: 'mysql', 'sqlite', 'postgres', 'mssql'
  dialect: 'mysql',

  // custom host; default: localhost
  host: 'my.server.tld',
  // for postgres, you can also specify an absolute path to a directory
  // containing a UNIX socket to connect over
  // host: '/sockets/psql_sockets'.
 
  // custom port; default: dialect default
  port: 12345,
 
  // custom protocol; default: 'tcp'
  // postgres only, useful for Heroku
  protocol: null,
 
  // disable logging; default: console.log
  logging: false,

  // you can also pass any dialect options to the underlying dialect library
  // - default is empty
  // - currently supported: 'mysql', 'postgres', 'mssql'
  dialectOptions: {
    socketPath: '/Applications/MAMP/tmp/mysql/mysql.sock',
    supportBigNumbers: true,
    bigNumberStrings: true
  },
 
  // the storage engine for sqlite
  // - default ':memory:'
  storage: 'path/to/database.sqlite',
 
  // disable inserting undefined values as NULL
  // - default: false
  omitNull: true,
 
  // a flag for using a native library or not.
  // in the case of 'pg' -- set this to true will allow SSL support
  // - default: false
  native: true,
 
  // Specify options, which are used when sequelize.define is called.
  // The following example:
  //   define: { timestamps: false }
  // is basically the same as:
  //   sequelize.define(name, attributes, { timestamps: false })
  // so defining the timestamps for each model will be not necessary
  define: {
    underscored: false
    freezeTableName: false,
    charset: 'utf8',
    dialectOptions: {
      collate: 'utf8_general_ci'
    },
    timestamps: true
  },
 
  // similar for sync: you can define this to always force sync for models
  sync: { force: true },
 
  // pool configuration used to pool database connections
  pool: {
    max: 5,
    idle: 30000,
    acquire: 60000,
  },

  // isolation level of each transaction
  // defaults to dialect default
  isolationLevel: Transaction.ISOLATION_LEVELS.REPEATABLE_READ
})
```

**Hint:** 로깅을 위한 사용자 정의 함수를 사용할 수도 있습니다. 그냥 함수를 전달하면 됩니다. 첫번째 전달인자는 문자열 로그입니다.

## 읽기 복제 (Read Replication)

Sequelize는 읽기 복제를 지원합니다. 예를들어, SELECT 쿼리를 수행할 때 연결할 수 있는 여러 서버가 있습니다. 읽기 복제를 수행할 때 복제할 하나 이상의 서버를 지정하고 모든 쓰기 및 업데이트를 처리하고 이를 복사본에 전파하는 마스터 쓰기 서버를 지정합니다. (실제 Sequelize에 의해 복제 프로세스는 처리되지 **않고**, 백엔드에서 설정해야합니다.)

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-username', password: 'some-password' },
      { host: '9.9.9.9', username: 'another-username', password: null }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: 'any-password' }
  },
  pool: { // If you want to override the options used for the read/write pool you can do so here
    max: 20,
    idle: 30000
  },
})
```

모든 복사본에 적용되는 기본 설정이 있는 경우 각 인스턴스마다 제공할 필요가 없습니다. 위 코드에서 데이터베이스 이름과 포트는 모든 복사본으로 전파됩니다. 복사본을 남겨두면 사용자 이름과 암호도 마찬가지입니다. 각 복사본에는 `host`,`port`,`username`,`password`,`database` 옵션이 있습니다.

Sequelize는 복사본 커넥션을 관리하기 위해 풀을 사용합니다. 내부적으로 `pool` 구성을 통해 생성된 두개의 풀을 유지, 관리합니다.

풀을 수정하려면 Sequelize를 인스턴스화 할 때 옵션을 전달합니다.

각각의 `write`또는 `useMaster: true` 쿼리는 쓰기 풀을 사용합니다. `SELECT`를 읽기 풀에서 사용합니다. 읽기 복사본은 기본 라운드 스케줄로 변경됩니다.

## 방언 (Dialects)

With the release of Sequelize `1.6.0`, the library got independent from specific dialects. This means, that you'll have to add the respective connector library to your project yourself.

### MySQL

Sequelize를 MySQL과 잘 사용하려면 `mysql2@^1.0.0-rc.10` 또는 그 이상의 버전을 사용해야합니다. 설치가 완료되면 다음과 같이 사용할 수 있습니다.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
})
```

**Note:** `dialectOptions` 매개변수를 설정하여 방언 라이브러리에 옵션을 직접 전달할 수 있습니다. [Options](/docs/latest/usage#options)을 읽어보세요. (현재는 MySQL만 지원됩니다.)

### SQLite

SQLite를 사용하려면 `sqlite3@~3.0.0`버전이 필요합니다. Sequelize설정은 아래와 같습니다.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // sqlite! now!
  dialect: 'sqlite',
 
  // the storage engine for sqlite
  // - default ':memory:'
  storage: 'path/to/database.sqlite'
})
```

또는 커넥션 문자열을 이용할 수 있습니다.

```js
const sequelize = new Sequelize('sqlite:/home/abs/path/dbname.db')
const sequelize = new Sequelize('sqlite:relativePath/dbname.db')
```

### PostgreSQL

PostgreSQL 은 `pg@^5.0.0 || ^6.0.0`를 사용해야합니다. dialect 만 지정해주면 됩니다.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // gimme postgres, please!
  dialect: 'postgres'
})
```

Unix 도메인 소켓을 통해 연결하려면 `host` 옵션으로 소켓 디렉터리에 대한 경로를 지정하세요.

소켓 경로는 `/`로 시작해야합니다.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // gimme postgres, please!
  dialect: 'postgres',
  host: '/path/to/socket_directory'
})
```

### MSSQL

MSSQL의 라이브라리는 `tedious@^1.7.0`를 사용합니다. dialect만 설정하면 됩니다.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql'
})
```

## 원시 SQL 쿼리 실행

원시 또는 이미 준비된 SQL 쿼리를 실행하는 것이 더 쉬운 경우에는 `sequelize.query` 함수를 활용할 수 있습니다.

- [Sequelize.query API](/class/lib/sequelize.js%7ESequelize.html#instance-method-query) 를 읽어보세요
- [Query Types](/variable/index.html#static-variable-QueryTypes)를 읽어보세요

사용방법입니다.

```js
// Arguments for raw queries
sequelize.query('your query', [, options])

// Quick example
sequelize.query("SELECT * FROM myTable").then(myTableRows => {
  console.log(myTableRows)
})

// If you want to return sequelize instances use the model options.
// This allows you to easily map a query to a predefined model for sequelize e.g:
sequelize
  .query('SELECT * FROM projects', { model: Projects })
  .then(projects => {
    // Each record will now be mapped to the project's model.
    console.log(projects)
  })


// Options is an object with the following keys:
sequelize
  .query('SELECT 1', {
    // A function (or false) for logging your queries
    // Will get called for every SQL query that gets send
    // to the server.
    logging: console.log,

    // If plain is true, then sequelize will only return the first
    // record of the result set. In case of false it will all records.
    plain: false,

    // Set this to true if you don't have a model definition for your query.
    raw: false,

    // The type of query you are executing. The query type affects how results are formatted before they are passed back.
    type: Sequelize.QueryTypes.SELECT
  })

// Note the second argument being null!
// Even if we declared a callee here, the raw: true would
// supersede and return a raw object.
sequelize
  .query('SELECT * FROM projects', { raw: true })
  .then(projects => {
    console.log(projects)
  })
```

쿼리를 대체하는 방법은 두가지입니다. 이름이 지정된 매개변수 (`:`로 시작) 또는 이름이 지정되지 않은 경우 ? 를 이용합니다

사용하는 문법은은 함수에 전달 된 대체 옵션에 따라 다릅니다.

- 배열이 전달되면 `?`은 배열에 나타나는 순서대로 대체됩니다.
- 객체가 전달되면 `:key`는 해당 객체의 키로 대체됩니다.객체에 쿼리에서 찾을 수 없는 키가 있거나 그 반대의 경우에는 예외가 발생합니다.

```js
sequelize
  .query(
    'SELECT * FROM projects WHERE status = ?',
    { raw: true, replacements: ['active']
  )
  .then(projects => {
    console.log(projects)
  })

sequelize
  .query(
    'SELECT * FROM projects WHERE status = :status ',
    { raw: true, replacements: { status: 'active' } }
  )
  .then(projects => {
    console.log(projects)
  })
```

**One note:** 테이블의 속성 이름에 점이 있으면 결과 객체가 중첩됩니다.

```js
sequelize.query('select 1 as `foo.bar.baz`').then(rows => {
  console.log(JSON.stringify(rows))

  /*
    [{
      "foo": {
        "bar": {
          "baz": 1
        }
      }
    }]
  */
})
```
