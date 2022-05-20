## sequelize

---



### sequelize 이해하기

---



> Sequelize is a modern TypeScript and Node.js ORM for Postgres, MySQL, MariaDB, SQLite and SQL Server, and more. Featuring solid transaction support, relations, eager and lazy loading, read replication and more.



Sequelize는 SQL을 위한 Node.js ORM이다.



#### ORM

---

sequelize를 알아보기 전 우선 ORM에 대해 알아보자.

ORM은 데이터 조작과 데이터베이스에서 온 객체를 사용한 질의하는 것(querying)을 도와주는 Object Relational Mapper 이다.

(An ORM is simply an Object Relational Mapper that helps in data manipulation and querying by the use of objects from the database.)

ORM을 optimizes를 사용하여 SQL 쿼리의 재사용과 유지가 쉬워지고 DB와 객체 사이의 데이터 전환을 간단하게 해준다.

이런 ORM을 사용하기 위해, 우리는 첫번째로 DB를 선택하여야 한다.

그리고 ORM을 선택하는데 Node.js에서 SQL을 사용할 때는 주로 Sequelize를 사용한다.

그 후, DB 모델을 만든다. 

DB 모델을 만들며 정의된 구조는 테이블, 컬렉션 그리고 컬럼을 포함한다. 

이는 DB의 구조를 나타내며 확장성과 효율에서 이점을 제공해준다.

이렇게 모델까지 정의하면 우리는 DB와 ORM을 연결할 수 있다.



#### sequelize

---

위와 같은 ORM 중의 하나가 sequelize이다.

Sequelize는 Node.js의 모듈로 DB로 MySQL, Postgres를 사용할 때 사용하는 ORM이다.

조금 더 sequelize에 대해 알아보기 위해 sequelize를 간단히 구현한 아래 코드를 살펴보자.

```javascript
// instantiate sequelize
const Sequelize = require('sequelize');

// connect db
const connection = new Sequelize("db name", "username", "password");

// define article model
const Article = connection.define("article", {
    title: Sequelize.String,
    content: Sequelize.String
});
connection.sync();
```

먼저 sequelize 생성자 객체를 Sequelize라 정의하였다. 

이 생성자를 활용하여 DB를 정의하였다.

그 후 모델을 정의하였다.

위와 같은 작업은 비동기적으로 처리하면 오류가 생길 수 있어 동기적으로 처리하기 위해 `connection.sync()` 와 같이 `sync()` 함수를 사용하였다.

이렇게 준비가 완료되면 아래 코드와 같이 sequelize의 API를 사용하여 DB에 질의 할 수 있다.

```javascript
// query using sequelize API
Article.findById(5).then(function (article) {
    console.log(article);
})
```



우선 위의 코드를 통해 간단히 sequelize를 알아보았다.

이러한 sequelize의 장점은 다음과 같다.

- Sequelize는 우리가 더 적은 코드를 작성할 수 있게 해준다.
- 우리가 더 지속성 있는 코드를 작성하게 해준다.
- SQL 쿼리 작성을 피할 수 있게 해준다.
- Sequelize가 DB 엔진을 추상화 해준다.
- 이주(migration)에 좋은 툴이다.




<details>
<summary>참고</summary>
<a href=https://www.section.io/engineering-education/introduction-to-sequalize-orm-for-nodejs/>https://www.section.io/engineering-education/introduction-to-sequalize-orm-for-nodejs/</a> 
</details>


### sequelize 활용하기

---



#### Connecting to a database

---

DB에 연결하기 위해서, Sequelize 인스턴스를 만들어야만 한다.

이는 Sequelize 생성자에 매개변수를 통해 값을 전달하거나 연결할 DB의 URI를 넣거는 방식으로 생성할 수 있다.

```javascript
const { Sequelize } = require('sequelize');

// Option 1: Passing a connection URI
const sequelize = new Sequelize('sqlite::memory:') // Example for sqlite
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // Example for postgres

// Option 2: Passing parameters separately (other dialects)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* one of 'mysql' | 'mariadb' | 'postgres' | 'mssql' */
});
```



#### Testing the connection

---

위에서 연결한 DB와 잘 연결되었는지 확인하기 위해서는 `.authenticate()` 메서드를 사용하여 확인할 수 있다.

```javascript
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}
```



#### Closing the connection

---

만약 DB와 연결을 끊고 싶다면 `seqelize.close()` 를 통해 연결을 끊을 수 있다.



#### Promises and async/await

---

Sequelize에 의해 제공되는 대부분의 메서드들은 비동기적이다. 

그래서 이들은 Promises를 반환한다. 

그렇기에 우리는 Promise API (`then` , `catch` , `finally` )를 사용할 수 있다.

물론 `async` 와 `await` 역시 사용할 수 있다.



#### Model

---

Sequelize에서 model은 필수적이다.

model은 DB안의 테이블을 나타내는 추상이다.

Sequelize에서는 이것은 Model 클래스를 확장하는 클래스이다.



model은 Sequelize에게 DB안의 테이블 이름이 무엇이고 어떤 칼럼을 그것이 가지고 있는지와 같은 그것이 나타내는 엔터티는 무엇인지 말해주는 것들을 가지고 있다. 



> 엔터티란 “업무에 필요하고 유용한 정보를 저장하고 관리하기 위한 집합적인 것(Thing)”으로 설명할 수 있다.



Sequelize에서 model은 이름을 가지고 있는데 이 이름은 그것이 나타내는 DB안에 있는 테이블의 이름과 같을 필요는 없다.

대게 model은 단수형 이름을 가지고 테이블은 복수형 이름을 가진다. 



> model  -  `User`
>
> table -  `Users`



##### Model Definition

---

Sequelize에서 model은 크게 2가지 방법으로 정의된다.

- Calling `seqeulize.define(modelName, attributes, options)`
- Extending Model and calling `init(attributes, options)`



아래는 위의 2가지 방법을 활용하여 model 정의한 예제 코드이다.

```javascript
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  // Model attributes are defined here
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull defaults to true
  }
}, {
  // Other model options go here
});

// `sequelize.define` also returns the model
console.log(User === sequelize.models.User); // true
```



```javascript
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class User extends Model {}

User.init({
  // Model attributes are defined here
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull defaults to true
  }
}, {
  // Other model options go here
  sequelize, // We need to pass the connection instance
  modelName: 'modelName' // We need to choose the model name
});

// the defined model is the class itself
console.log(User === sequelize.models.User); // true
```



```javascript
// 아래 코드는 model을 따로 구현할 경우 예제이다.

const { DataTypes, Sequelize } = require('sequelize');

module.exports = class User extends Sequelize.Model {
  // 클래스 메서드 init은 sequelize를 매개변수로 받는다.
  static init(sequelize) {
    // Sequelize.Model의 init 메서드를 활용하여 그 값을 반환 받는다.
    return super.init({
      email: {
        type: DataTypes.STRING(40),
        allowNull: true,
        unique: true,
      },
      password: {
        type: DataTypes.STRING(100),
        allowNull: true,
      }
    }, {
      sequelize,
      modelName: 'User',
      tableName: 'users'
    });
  }
};
```

`sequelize.define` 그리고 `Model.init` 는 내부적으로 두 접근 방법은 동일하다.

그렇기에 둘중 어느 것을 사용하여도 상관없다.

다만 한가지 주목할 것은 `Model.init` 을 사용할 때의 `option` 이다.

위의 예제의 `Model.init` 의 `option` 값을 살펴 보면 `seqeulize` 그리고 `modelName` 이 있다.

이는 `User` 가 sequelize에 의해 생성된 객체가 아니라 Sequelize 클래스의 Model 클래스 메서드를 활용한 것이기 때문에 `seqeulize` 를 `option` 을 통해 넣어준 것 같다. 

그리고 `modelName` 역시 동일한 이유로 넣어준 것이라 생각한다.

아래 코드는 위의 생각을 뒷받침할 `node_modules/sequelize/lib/model.js` 속의 코드이다.

```javascript
  static init(attributes, options = {}) {
    var _a, _b;
    if (!options.sequelize) {
      throw new Error("No Sequelize instance passed");
    }
    this.sequelize = options.sequelize; // `sequelize`을 option 값에 넣는 이유
    const globalOptions = this.sequelize.options;
    options = Utils.merge(_.cloneDeep(globalOptions.define), options);
    if (!options.modelName) {
      options.modelName = this.name;
    }
    // `modelName`을 option 값에 넣는 이유
    options = Utils.merge({
      name: {
        plural: Utils.pluralize(options.modelName),
        singular: Utils.singularize(options.modelName)
      },
      indexes: [],
      omitNull: globalOptions.omitNull,
      schema: globalOptions.schema
    }, options);
    
   // ....
```



이렇게 model이 정의되고난 이후, 그것은 `sequelize.models.modelName` 혹은 이를 담은 변수  `modelName` 과 같은 형태로 사용될 수 있다.



##### Table name inference

---

위의 model을 정의할 때, 2가지 메서드 모두 modelName은 정의하였지만 tableName은 정의하지 않았다. 



기본적으로 tableName이 주어지지 않으면 Sequelize는 자동적으로 modelName을 복수화하여 그것을 tableName으로 활용한다.

이 복수화는 `inflection` 이라는 라이브러리를 통해 진행되어 `person -> people` 과 같은 것도 복수화가 진행된다.



tableName을 자동이 아니라 직접 설정하고 싶다면 `option` 값에 `tableName : 'tableName'`  을 넣어주어 직접 설정하면 된다.



만약 tableName을 modelName과 동일하게 하고 싶다면 `option` 값에 `freezeTable : true` 을 넣어주면 된다.

이를 전체 모델에 적용하고 싶으면 아래 코드와 같이 sequelize를 생성하면 된다.

``` javascript
const sequelize = new Sequelize('sqlite::memory:', {
  define: {
    freezeTableName: true
  }
});
```



##### Model synchronization

---

model을 정의할 때 Sequelize에게 DB안의 테이블에 관한 정보를 주었다.

(`.define` 그리고 `init` 메서드를 통해서 model을 정의하였다.)

하지만 테이블이 DB안에 존재하지 않으면?

만약 존재하더라도 칼럼이 다르거나, 적거나 혹은 다른 다른 것이 있다면? 



위와 같은 이유 때문에 model synchronization이 필요하다.

model은 `model.sync(options)` 와 같은 명령을 통해 DB와 synchronize 될 수 있다. 

위의 명령을 통해, Sequelize는 자동으로 DB에 SQL query를 보낸다.

이를 통한 변화는 DB의 테이블에서 일어나지 Javascript의 model에서 일어나지 않는다.



우리가 DB를 사용할 때 model을 하나만 사용하지는 않을 것이다. 

모든 model을 한번에 synchronizing 하기 위해서는 아래와 같은 코드를 사용하면 된다.

```javascript
await sequelize.sync({ force: true });
console.log("All models were synchronized successfully.");
```



즉, synchronizing을 하기 위해서는 사전에 model이 정의되어 있어야 한다.

이를 확인하기 위해서 `console.log()` 를 활용해 보았다.



본인이 공부하고 있는 폴더의 구조를 간단히 하면 다음과 같다.

```
app_practice
	sequelize
		models
			index.js
	app.js
```

```javascript
// app.js

// serverStart  
dotenv.config();
console.log("sequelize is faster!");
sequelize.sync();
passportConfig();
```

```javascript
const db = {};
db.User = User;
db.sequelize = sequelize; 

console.log("init is faster!");
User.init(sequelize);

module.exports = db;
```

![sequelize](C:\Users\User\Desktop\Study\Nodong\Npm\image\sequelize.JPG)

위의 사진은 서버를 시작한 후 `console` 에 나온 결과이다.

model을 정의하는 `.init()` 이 먼저 수행되고 

`.sync()` 가 수행된다.



##### Dropping tables

---

model과 연관된 테이블을 삭제하기 위해서는 아래와 같은 코드를 

```javascript
await User.drop();
console.log("User table dropped!");
```

모든 테이블을 삭제하기 위해서는 아래와 같은 코드를 사용하면 된다.

```javascript
await sequelize.drop();
console.log("All tables dropped!");
```



##### Timestamps

---

기본적으로 Sequelize는 자동적으로 `createdAt` 그리고 `updateAt` 이라는 필드를 모든 model에 `DataTypes.DATE` 를 활용하여 추가한다. 

그 필드들은 자동적으로 관리가 된다.

`createdAt` 필드는 생성된 순간을 나타내는 타임스탬프 값을 가지고 있고 `updateAt` 은 가장 최근 업데이트된 타임스탬프 값을 가지고 있다.



아래는 timestamps를 설정하는 예제 코드이다.

```javascript
sequelize.define('User', {
  // ... (attributes)
}, {
  timestamps: false
});
```

```javascript
class Foo extends Model {}
Foo.init({ /* attributes */ }, {
  sequelize,

  // don't forget to enable timestamps!
  timestamps: true,

  // I don't want createdAt
  createdAt: false,

  // I want updatedAt to actually be called updateTimestamp
  updatedAt: 'updateTimestamp'
});
```



##### Coulmn declaration shorthand syntax

---



```javascript
// This:
sequelize.define('User', {
  name: {
    type: DataTypes.STRING
  }
});

// Can be simplified to:
sequelize.define('User', { name: DataTypes.STRING });
```



##### Default Values

---

Sequelize는 칼럼의 기본값을 `Null` 으로 가지고 있다.

이 값은 `defaultValue` 라는 것을 통해 바꿀 수 있다.



이를 활용한 코드는 아래와 같다.

```javascript
sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    defaultValue: "John Doe"
  }
});
```

```javascript
sequelize.define('Foo', {
  bar: {
    type: DataTypes.DATETIME,
    defaultValue: DataTypes.NOW
    // This way, the current date/time will be used to populate this column (at the moment of insertion)
  }
});
```



##### Data Types

---

model에 정의된 모든 칼럼은 data type을 가지고 있어야 한다.

Sequelize는 많은 내장 data type을 제공한다.

내장 data type에 접근하려면 `DataTypes` 를 다음과 같이 import 하면 된다.

```javascript
const { DataTypes } = require("sequelize"); // Import the built-in data types
```



**Strings, Boolean, Numbers, Dates, UUIDs** 등에 관한 data type이 정의되어 있다. 



자세한 것은 다음을 참고 하자.

[Data Types]: https://sequelize.org/docs/v6/core-concepts/model-basics/#data-types	"Data Types"



##### Coulmn Options

---

위에서 컬럼을 정의할 때 `type` 뿐만 아니라 `allowNull` 그리고 `defaultValue` 를 `option` 에 사용하여 보았다.

아직 다른 다양한 `option` 들이 남아 있다.

이는 아래와 같다.

```javascript
const { Model, DataTypes, Deferrable } = require("sequelize");

class Foo extends Model {}
Foo.init({
  // instantiating will automatically set the flag to true if not set
  flag: { type: DataTypes.BOOLEAN, allowNull: false, defaultValue: true },

  // default values for dates => current time
  myDate: { type: DataTypes.DATE, defaultValue: DataTypes.NOW },

  // setting allowNull to false will add NOT NULL to the column, which means an error will be
  // thrown from the DB when the query is executed if the column is null. If you want to check that a value
  // is not null before querying the DB, look at the validations section below.
  title: { type: DataTypes.STRING, allowNull: false },

  // Creating two objects with the same value will throw an error. The unique property can be either a
  // boolean, or a string. If you provide the same string for multiple columns, they will form a
  // composite unique key.
  uniqueOne: { type: DataTypes.STRING,  unique: 'compositeIndex' },
  uniqueTwo: { type: DataTypes.INTEGER, unique: 'compositeIndex' },

  // The unique property is simply a shorthand to create a unique constraint.
  someUnique: { type: DataTypes.STRING, unique: true },

  // Go on reading for further information about primary keys
  identifier: { type: DataTypes.STRING, primaryKey: true },

  // autoIncrement can be used to create auto_incrementing integer columns
  incrementMe: { type: DataTypes.INTEGER, autoIncrement: true },

  // You can specify a custom column name via the 'field' attribute:
  fieldWithUnderscores: { type: DataTypes.STRING, field: 'field_with_underscores' },

  // It is possible to create foreign keys:
  bar_id: {
    type: DataTypes.INTEGER,

    references: {
      // This is a reference to another model
      model: Bar,

      // This is the column name of the referenced model
      key: 'id',

      // With PostgreSQL, it is optionally possible to declare when to check the foreign key constraint, passing the Deferrable type.
      deferrable: Deferrable.INITIALLY_IMMEDIATE
      // Options:
      // - `Deferrable.INITIALLY_IMMEDIATE` - Immediately check the foreign key constraints
      // - `Deferrable.INITIALLY_DEFERRED` - Defer all foreign key constraint check to the end of a transaction
      // - `Deferrable.NOT` - Don't defer the checks at all (default) - This won't allow you to dynamically change the rule in a transaction
    }
  },

  // Comments can only be added to columns in MySQL, MariaDB, PostgreSQL and MSSQL
  commentMe: {
    type: DataTypes.INTEGER,
    comment: 'This is a column name that has a comment'
  }
}, {
  sequelize,
  modelName: 'foo',

  // Using `unique: true` in an attribute above is exactly the same as creating the index in the model's options:
  indexes: [{ unique: true, fields: ['someUnique'] }]
});
```



#### Model Instance

---



##### Creating an instance

---

model은 클래스로 `new` 를 사용하여 직접적으로 인스턴스를 만드는 것이 아니라 `build` 를 사용하여야 한다.

```javascript
const jane = User.build({ name: "Jane" });
console.log(jane instanceof User); // true
console.log(jane.name); // "Jane"
```



하지만 위의 코드는 DB와 전혀 소통하지 않는다.

이는 `build` 메서드가 DB와 매칭할 수 있는 데어터를 나타내는 객체를 만들기 때문이다.

(This is because the [`build`](https://sequelize.org/api/v6/class/src/model.js~Model.html#static-method-build) method only creates an object that *represents* data that *can* be mapped to a database.)

이 인스턴스를 진짜 DB에 저장하기 위해서는 `save` 메서드를 사용하여야 한다.

```javascript
await jane.save();
console.log('Jane was saved to the database!');
```



위의 `build` 와 `save` 메서드를 합한 `create` 메서드를 Sequelize는 제공한다.

```javascript
const jane = await User.create({ name: "Jane" });
// Jane exists in the database now!
console.log(jane instanceof User); // true
console.log(jane.name); // "Jane"
```



##### Updating an instance

---

인스턴스의 필드 값을 바꾼다면, `save` 를 다시한번 사용하여야 한다.

```javascript
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// the name is still "Jane" in the database
await jane.save();
// Now the name was updated to "Ada" in the database!
```

이때 `save` 는 그 인스턴스에서의 값의 변화 전체를 업데이트한다. 

만약 특정한 필드만 업데이트하고 싶다면 `update` 를 사용하면 된다.

```javascript
const jane = await User.create({ name: "Jane" });
jane.favoriteColor = "blue"
await jane.update({ name: "Ada" })
// The database now has "Ada" for name, but still has the default "green" for favorite color
await jane.save()
// Now the database has "Ada" for name and "blue" for favorite color
```

 여러 필드 값을 바꾸고 싶다면, `set` 메서드를 사용하면 된다.

```javascript
const jane = await User.create({ name: "Jane" });

jane.set({
  name: "Ada",
  favoriteColor: "blue"
});
// As above, the database still has "Jane" and "green"
await jane.save();
// The database now has "Ada" and "blue" for name and favorite color
```



##### Deleting an instance

---

`destroy` 메서드를 통해서 인스턴스를 삭제할 수 있다.

```javascript
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
await jane.destroy();
// Now this entry was removed from the database
```



##### Reloading an instance

---

`reload` 메서드를 통해서 데이터베이스로부터 인스턴스를 다시 불러올 수 있다.

```javascript
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// the name is still "Jane" in the database
await jane.reload();
console.log(jane.name); // "Jane"
```



##### Incrementing and decrementing integer values

---

동시성(concurrency)에 관한 이슈없이 인스턴스의 값을 정렬하기 위해, Sequelize는 `increment` 그리고 `decrement` 메서드를 제공한다.

```javascript
const jane = await User.create({ name: "Jane", age: 100 });
const incrementResult = await jane.increment('age', { by: 2 });
// Note: to increment by 1 you can omit the `by` option and just do `user.increment('age')`

// In PostgreSQL, `incrementResult` will be the updated user, unless the option
// `{ returning: false }` was set (and then it will be undefined).

// In other dialects, `incrementResult` will be undefined. If you need the updated instance, you will have to call `user.reload()`.
```

```javascript
const jane = await User.create({ name: "Jane", age: 100, cash: 5000 });
await jane.increment({
  'age': 2,
  'cash': 100
});

// If the values are incremented by the same amount, you can use this other syntax as well:
await jane.increment(['age', 'cash'], { by: 2 });
```



#### Model Querying - Basics

---

Sequelize는 DB의 데이터를 질의(querying)하기 위한 다양한 메서드를 제공한다.

아래는 특히 CRUD 쿼리를 의주로 알아보려 한다.



##### Simple INSERT queries

---

```javascript
// Create a new user
const jane = await User.create({ firstName: "Jane", lastName: "Doe" });
console.log("Jane's auto-generated ID:", jane.id);
```

`Model.create()` 메서드로 앞서 본 메서드이다.



```javascript
const user = await User.create({
  username: 'alice123',
  isAdmin: true
}, { fields: ['username'] });
// let's assume the default of isAdmin is false
console.log(user.username); // 'alice123'
console.log(user.isAdmin); // false
```

이 메서드는 유저에 의해 채워지는 폼에 기반한 DB를 만들 때 특히 유용하다.

예를 들어, `create` 를 사용하면 `User` 모델이 username은 작성할 수 있지만 admin flag는 작성할 수 없다.

위의 코드에서 볼 수 있듯 

`isAdmin` 의 기본값이 false라면 `create` 를 통해 true라고 인스턴스를 만들더라도 false가 유지된다. 



##### Simple SELECT queries

---

`findAll` 메서드를 통해 DB 테이블 전체를 읽을 수 있다.

```javascript
// Find all users
const users = await User.findAll();
console.log(users.every(user => user instanceof User)); // true
console.log("All users:", JSON.stringify(users, null, 2));
```

```
SELECT * FROM ...
```



##### Specifying attributes for SELECT queries

---

위에서는 테이블 전체를 읽어 왔는데 특정 속성만 읽어오려면 `attributes` 옵션을 사용하면 된다.

```javascript
Model.findAll({
  attributes: ['foo', 'bar']
});
```

```
SELECT foo, bar FROM ...
```



이때 불러오는 속성의 이름을 다르게 해서 읽어오려면 배열(nested array)를 활용하면 된다.

```javascript
Model.findAll({
  attributes: ['foo', ['bar', 'baz'], 'qux']
});
```

```
SELECT foo, bar AS baz, qux FROM ...
```



집계함수(aggregation)을 사용하여 속성을 가져오려면 `sequelize.fn` 을 사용하면 된다.

`sequelize.fn` 은 `function fn(fn: string, ...args: unknown[]): Fn;`  와 같이 정의되어 있어 문자열로 sql에서의 함수명을 받고 이후 그것을 적용할 칼럼을 받는다.

`sequelize.fn` 은 아래서 다시 알아보도록 하자.

```javascript
Model.findAll({
  attributes: [
    'foo',
    [sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats'],
    'bar'
  ]
});
```

```
SELECT foo, COUNT(hats) AS n_hats, bar FROM ...
```

집계함수를 사용할 때는 model에서 그것에 접근할 수 있는 별명(alias)를 설정하여야 한다.

이는 결과 값의 속성명으로 사용된다.



속성 값을 불러올 때 불러올 속성 값을 설정하는 것이 아니라 불러오지 않을 속성 값을 설정하는 방법도 있다.

```javascript
Model.findAll({
  attributes: { exclude: ['baz'] }
});
```

```
-- Assuming all columns are 'id', 'foo', 'bar', 'baz' and 'qux'
SELECT id, foo, bar, qux FROM ...
```

이때 `attributes` 의 값으로 다른 것과 같이 배열이 들어가는 것이 아니라 객체가 들어간다는 것에 주목하자. 

객체를 통해 옵션을 전달해서 해당 속성을 제외하는 것이다.



##### Applying WHERE clauses

---

`where` 옵션은 질의를 분류(filter)하기 위해 사용된다.

이는 `Op` 에 의해 이루어 진다.



`Op.eq` 

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    authorId: {
      [Op.eq]: 2
    }
  }
});
// SELECT * FROM post WHERE authorId = 2;
```



`Op.and` 

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [
      { authorId: 12 },
      { status: 'active' }
    ]
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';
```



`Op.or`

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.or]: [
      { authorId: 12 },
      { authorId: 13 }
    ]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;
```

```javascript
const { Op } = require("sequelize");
Post.destroy({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// DELETE FROM post WHERE authorId = 12 OR authorId = 13;
```



##### Operators

---

```javascript
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [{ a: 5 }, { b: 6 }],            // (a = 5) AND (b = 6)
    [Op.or]: [{ a: 5 }, { b: 6 }],             // (a = 5) OR (b = 6)
    someAttribute: {
      // Basics
      [Op.eq]: 3,                              // = 3
      [Op.ne]: 20,                             // != 20
      [Op.is]: null,                           // IS NULL
      [Op.not]: true,                          // IS NOT TRUE
      [Op.or]: [5, 6],                         // (someAttribute = 5) OR (someAttribute = 6)

      // Using dialect specific column identifiers (PG in the following example):
      [Op.col]: 'user.organization_id',        // = "user"."organization_id"

      // Number comparisons
      [Op.gt]: 6,                              // > 6
      [Op.gte]: 6,                             // >= 6
      [Op.lt]: 10,                             // < 10
      [Op.lte]: 10,                            // <= 10
      [Op.between]: [6, 10],                   // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15],               // NOT BETWEEN 11 AND 15

      // Other operators

      [Op.all]: sequelize.literal('SELECT 1'), // > ALL (SELECT 1)

      [Op.in]: [1, 2],                         // IN [1, 2]
      [Op.notIn]: [1, 2],                      // NOT IN [1, 2]

      [Op.like]: '%hat',                       // LIKE '%hat'
      [Op.notLike]: '%hat',                    // NOT LIKE '%hat'
      [Op.startsWith]: 'hat',                  // LIKE 'hat%'
      [Op.endsWith]: 'hat',                    // LIKE '%hat'
      [Op.substring]: 'hat',                   // LIKE '%hat%'
      [Op.iLike]: '%hat',                      // ILIKE '%hat' (case insensitive) (PG only)
      [Op.notILike]: '%hat',                   // NOT ILIKE '%hat'  (PG only)
      [Op.regexp]: '^[h|a|t]',                 // REGEXP/~ '^[h|a|t]' (MySQL/PG only)
      [Op.notRegexp]: '^[h|a|t]',              // NOT REGEXP/!~ '^[h|a|t]' (MySQL/PG only)
      [Op.iRegexp]: '^[h|a|t]',                // ~* '^[h|a|t]' (PG only)
      [Op.notIRegexp]: '^[h|a|t]',             // !~* '^[h|a|t]' (PG only)

      [Op.any]: [2, 3],                        // ANY (ARRAY[2, 3]::INTEGER[]) (PG only)
      [Op.match]: Sequelize.fn('to_tsquery', 'fat & rat') // match text search for strings 'fat' and 'rat' (PG only)

      // In Postgres, Op.like/Op.iLike/Op.notLike can be combined to Op.any:
      [Op.like]: { [Op.any]: ['cat', 'hat'] }  // LIKE ANY (ARRAY['cat', 'hat'])

      // There are more postgres-only range operators, see below
    }
  }
});
```

sql에서 사용하는 많은 조건들을 `Operator` 을 통해 사용할 수 있다.

더 자세한 사용법은 다음 링크를 참고하자.

[Operators]: https://sequelize.org/docs/v6/core-concepts/model-querying-basics/#operators



##### Simple UPDATE queries

---

```javascript
// Change everyone without a last name to "Doe"
await User.update({ lastName: "Doe" }, {
  where: {
    lastName: null
  }
});
```



##### Simple DELETE queries

---

```javascript
// Delete everyone named "Jane"
await User.destroy({
  where: {
    firstName: "Jane"
  }
});
```



##### Creating in bulk

---

앞서 새로운 데이터를 `create` 할 때 하나씩 하였다. 

하지만 `Model.bulkCreate` 를 사용하면 여러 데이터를 한번에 `create` 할 수 있다.

```javascript
const captains = await Captain.bulkCreate([
  { name: 'Jack Sparrow' },
  { name: 'Davy Jones' }
]);
console.log(captains.length); // 2
console.log(captains[0] instanceof Captain); // true
console.log(captains[0].name); // 'Jack Sparrow'
console.log(captains[0].id); // 1 // (or another auto-generated value)
```



`bulkCreate` 를 사용할 때 주의점은 `bulkCreate` 의 기본 값이 만들 각 객체의 유효성(validate) 검사를 하지 않는다는 것이다.

`bulkCreate` 가 유효성 검사를 하게 하려면 `validate : true` 라는 옵션을 제공하여야 한다.

```javascript
const Foo = sequelize.define('foo', {
  bar: {
    type: DataTypes.TEXT,
    validate: {
      len: [4, 6]
    }
  }
});

// This will not throw an error, both instances will be created
await Foo.bulkCreate([
  { name: 'abc123' },
  { name: 'name too long' }
]);

// This will throw an error, nothing will be created
await Foo.bulkCreate([
  { name: 'abc123' },
  { name: 'name too long' }
], { validate: true });
```

 

##### Ordering and Grouping

---

Sequelize는 sql의 `ORDER BY` 그리고 `GROUP BY` 에 해당하는 `order` 그리고 `group` 을 생성한다.



###### Ordering

`order` 옵션은 쿼리나 sequelize 메서드로 정렬할 아이템의 배열을 받는다.

이 아이템은 `[column, direction]` 형태의 배열이다. 

```javascript
Subtask.findAll({
  order: [
    // Will escape title and validate DESC against a list of valid direction parameters
    ['title', 'DESC'],

    // Will order by max(age)
    sequelize.fn('max', sequelize.col('age')),

    // Will order by max(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // Will order by  otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // Will order an associated model's createdAt using the model name as the association's name.
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's createdAt using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // Will order by an associated model's createdAt using the name of the association.
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's createdAt using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's createdAt using an association object. (preferred method)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's createdAt using association objects. (preferred method)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's createdAt using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // Will order by a nested associated model's createdAt simple association objects.
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ],

  // Will order by max age descending
  order: sequelize.literal('max(age) DESC'),

  // Will order by max age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.fn('max', sequelize.col('age')),

  // Will order by age ascending assuming ascending is the default order when direction is omitted
  order: sequelize.col('age'),

  // Will order randomly based on the dialect (instead of fn('RAND') or fn('RANDOM'))
  order: sequelize.random()
});

Foo.findOne({
  order: [
    // will return `name`
    ['name'],
    // will return `username` DESC
    ['username', 'DESC'],
    // will return max(`age`)
    sequelize.fn('max', sequelize.col('age')),
    // will return max(`age`) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // will return otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // will return otherfunction(awesomefunction(`col`)) DESC, This nesting is potentially infinite!
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
  ]
});
```



###### Grouping

grouping의 문법은 ordering의 문법과 같다. 

다만 배열 마지막의 방향에 관한 것은 받지 않는다.

```javascript
Project.findAll({ group: 'name' });
// yields 'GROUP BY name'
```



##### Utility methods

---

`count`,  `max`,  `min`,  `sum`, `increment`, `decrement`  와 같은 메서드가 있다.

이는 위에서 알아보았던 것을 간단히 한 메서드이다.



##### Sequelize.fn()

---

`Sequelize.fn()`을 사용하면 sql 함수를 쉽게 사용할 수 있다.

```typescript
function fn(fn: string, ...args: unknown[]): Fn;
```

첫 번째 매개변수로 함수명을, 두 번째 매개변수 부터는 대상을 지정한다.

즉, sql에서 사용하는 함수명을 알고 있다면 `Squelize.fn()` 을 사용할 수 있는 것이다.



##### Sequelize.literal()

---

서술적 sql 함수는 `Sequelize.fn()` 으로 해결할 수 없다고 한다.

IF나 CASE의 경우에는 컬럼이나 상수 뿐만 아니라, >=, <=, >, <, IN 등의 문자가 들어가서 fn()으로 처리하기 까다롭다 한다.

이러한 상황에서 `Sequelize.literal()` 을 사용할 수 있다.

```javascript
const completeMail = Sequelize.literal(`
    if(user.is_local IS NULL, CONCAT(user.email, ${domain}), user.email)
`);

await this.model.findAndCountAll({
    attributes: ["id", [completeMail, "email"]],
    where: {
        id: user.id
    }
});
```






<details>
<summary>참고</summary>
<a href=https://sequelize.org/docs/v6/getting-started/>https://sequelize.org/docs/v6/getting-started/</a></br>
<a href=https://jaehoney.tistory.com/141?category=869692>https://jaehoney.tistory.com/141?category=869692</a>
</details>
