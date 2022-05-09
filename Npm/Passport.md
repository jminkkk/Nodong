### Passport

---

> Passport is Express-compatible authentication middleware for Node.js
>
> Passport's sole purpose is **to authenticate requests**, which it does **through an extensible set of plugins known as strategies**.
>
> Passport does not mount routes or assume any particular database schema, which maximizes flexibility and allows application-level decisions to be made by the developer. 
>
> The API is simple: 
> **You provide Passport a request to authenticate**.
> **Passport provides hooks for controlling what occurs when authentication succeeds or fails.**

위는 Passport npm 공식문서의 소개이다.

간단히 요약하면 Passport는 요청(requests)을 전략(strategy)를 통해 인증(authenticate)하는 것이다.

그러기 위해 우리는 Passport에게 인증을 위한 요청을 보내고 Passport는 우리에게 인증이 성공 혹은 실패했을 때 일어나는 것을 컨트롤할 방법(hook)을 제공한다.

위에서 주목해야할 키워드는 **요청, 전략, 인증**이다.

앞으로의 글에서 위의 키워드와 메서드를 중심으로 Passport를 알아보자.



#### passport.initialize()

---

```javascript
app.use(passport.initialize());
```

passport.initialize()는 위와 같이 미들웨어로 등록되어 매 요청이 있을 때 마다 실행이된다. 

그렇다면 passport.initialize()는 어떤 메서드이기에 매 요청마다 실행이 되어야 할까?

node_moudles/passport/lib/middleware/initalize.js 속의 코드를 살펴보자.

```javascript
module.exports = function initialize(passport, options) {
  options = options || {};
  
  return function initialize(req, res, next) {
    if (options.userProperty) {
      req._userProperty = options.userProperty;
    }
    
    var compat = (options.compat === undefined) ? true : options.compat;
    if (compat) {
      req._passport = {};
      req._passport.instance = passport;
    }
    next();
  };
};
```

위의 `req._passport = {};` 과 `req._passport.instance = passport;` 를 보면 passport.initialize()는 req에 passport instance를 만드는 것을 알 수 있다.



#### passport.session

---

```javascript
app.use(passport.session());
```

이 역시 미들웨어로 등록되어 사용된다.

passport.session은 최근 session id의 user 값을 deserialize된 user 객체로 바꾸는 역활을 한다.

그렇기에 passport.session은 아래의 코드와 동일하다

```javascript
app.use(passport.authenticate('session'));
```



passport.session에 관하여 조금 더 알아보자.

아래는 passport.session 과 동일한 passport.authenticate('session')에 의해 수행되는 코드이다.

```javascript
SessionStrategy.prototype.authenticate = function(req, options) {
  if (!req.session) { return this.error(new Error('Login sessions require session support. Did you forget to use `express-session` middleware?')); }
  options = options || {};

  var self = this, 
      su;
  if (req.session[this._key]) {
    su = req.session[this._key].user;
  }

  if (su || su === 0) {      
    var paused = options.pauseStream ? pause(req) : null;
    this._deserializeUser(su, req, function(err, user) {
      if (err) { return self.error(err); }
      if (!user) {
        delete req.session[self._key].user;
      } else {
        var property = req._userProperty || 'user';
        req[property] = user;
      }
      self.pass();
      if (paused) {
        paused.resume();
      }
    });
  } else {
    self.pass();
  }
};
```

우선 세션의 여부부터 확인한다.

그 후 `req.session[this._key].user` 을 통하여 session id의 user 값을 확인한다.

그 이후 `this.deserializeUser` 을 통해 deserialize한 user 값을  `req.user` 에  저장한다.



#### passport.serializeUser

---

```javascript
passport.serializeUser(function(user, cb) {
  process.nextTick(function() {
    cb(null, { id: user.id, username: user.username });
  });
});
```

이는 Passport Tutorial에 나오는 코드이다.

passport.serializeUser는 매개변수로 함수를 받고 있다. 

이 함수는 추후에 req.login을 통한 로그인 요청에서 유저의 어떤 정보를 serialize할 것인지 미리 정의하는 역활을 한다.

이를 보다 정확히 이해하기 위해 passport.serializeUser가 어떻게 작동되는지 살펴보자.

```javascript
function Authenticator() {
  this._key = 'passport';
  this._strategies = {};
  this._serializers = [];
  this._deserializers = [];
  this._infoTransformers = [];
  this._framework = null;
  
  this.init();
}
```

> 	위 코드는 Passport 객체의 'Authenticator' 생성자이다. 
> 	위의 생성자를 통해 Passport 최초의 객체가 생성되게 된다.

Passport 최초의 객체가 생성될 때 `this.init()` 메서드를 바로 실행한다.

```javascript
Authenticator.prototype.init = function() {
  this.framework(require('./framework/connect')());
  this.use(new SessionStrategy({ key: this._key }, this.deserializeUser.bind(this)));
  this._sm = new SessionManager({ key: this._key }, this.serializeUser.bind(this));
};
```

> `this.init()` 메서드는 session에 관한 설정을 하는 것으로 파악된다.

위 두 코드를 통해 최초의 Passport 객체의 serializers는 단지 빈 배열일 뿐인 것을 알 수 있다. 

이 빈 배열을 채워주는 메서드가 passport.serializeUser인 것이다.

```javascript
Authenticator.prototype.serializeUser = function(fn, req, done) {
  // 매개변수로 function을 받을 경우 수행되는 코드
  if (typeof fn === 'function') {
    return this._serializers.push(fn);
  }
};
```

> passport.serializeUser은 매개변수로 받은 함수를 앞서 생성한 최초의 Passport 객체의 serializers에 푸시한다.

이렇게 serializers에 푸시된 정보는 `req.login` 이 `sessionManager.logIn` 을 호출하고 `sessionManager.logIn`이 `passport.serializeUser` 을 호출함으로써 사용되게 된다.

```javascript
req.login =
req.logIn = function(user, options, done) {
  if (typeof options == 'function') {
    done = options;
    options = {};
  }
  options = options || {};
  
  var property = this._userProperty || 'user';
  var session = (options.session === undefined) ? true : options.session;
  
  this[property] = user;
  if (session && this._sessionManager) {
    if (typeof done != 'function') { throw new Error('req#login requires a callback function'); }
    
    var self = this;
    this._sessionManager.logIn(this, user, function(err) {
      if (err) { self[property] = null; return done(err); }
      done();
    });
  } else {
    done && done();
  }
};
```

> `req.login` 메서드가 `sessionManager.logIn` 메서드를 호출한다.

```javascript
SessionManager.prototype.logIn = function(req, user, cb) {
  var self = this;
  this._serializeUser(user, req, function(err, obj) {
    if (err) {
      return cb(err);
    }
    // TODO: Error if session isn't available here.
    if (!req.session) {
      req.session = {};
    }
    if (!req.session[self._key]) {
      req.session[self._key] = {};
    }
    req.session[self._key].user = obj;
    cb();
  });
}
```

> `sessionManager.login` 메서드가 세션에 `passport.serializeUser` 을 사용하여 유저 정보를 기록한다.

아래는 passport.serializeUser가 매개변수로 function이 아닌 매개변수, 즉 user 정보 그리고 요청(req) 그리고 콜백수행함수(function)를 받는 경우 수행되는 코드이다.

```javascript
// private implementation that traverses the chain of serializers, attempting
// to serialize a user
var user = fn;
// For backwards compatibility
if (typeof req === 'function') {
    done = req;
    req = undefined;
}
var stack = this._serializers;
(function pass(i, err, obj) { // serializers use 'pass' as an error to skip processing
    if ('pass' === err) {
        err = undefined;
    }
    // an error or serialized object was obtained, done
    if (err || obj || obj === 0) {
        return done(err, obj);
    }
    var layer = stack[i];
    if (! layer) {
        return done(new Error('Failed to serialize user into session'));
    }
    function serialized(e, o) {
        pass(i + 1, e, o);
    }
    try {
        var arity = layer.length;
        if (arity == 3) {
            layer(req, user, serialized);
        } else {
            layer(user, serialized);
        }
    } catch (e) {
        return done(e);
    }
})(0);
```

이때 `user`은 `passport.authenticate` 메서드에서 전략을 수행한 후 전달 받은 유저의 정보를 가지고 있는 객체이다. 



> **즉 `passport.serializeUser` 은 다음과 같이 정리할 수 있다.**
>
> 1) **최초로 Passport를 선언할 때 그리고 `passport.initialize` 메서드를 통해 요청(req)이 발생시 생성되는 passport 객체가 추후 `serializeUser` 메서드를 어떻게 수행 할 것인가를 `passport.serializeUser` 에 정의한다. 그렇기에 `passport.serizlizeUser`은 `passport.initialize` 메서드가 수행되기 전에 정의되어야 하는 것이다.**
> 2) **`passport.authenticate` 를 수행하는 중 `req.login` 에 의해 `sessionManager.logIn` 이 호출 되고 `sessionManager.logIn` 이 passport 객체에 등록된 `passport.serializeUser` 를 사용하여 유저 정보를 세션에 serialize 한다.** 



#### passport.deserializeUser

---

passport.deserializeUser 은 모든 요청 이후 혹은 `passport.session`에의해 사용된다고 한다. 

passport.deserializeUser도 passport.serializeUser와 동일한 기능을 수행한다.

이를 확인하기 위해 passport.desializeUser과 이를 호출하는 passport.session의 코드를 살펴보자.

```javascript
Authenticator.prototype.deserializeUser = function(fn, req, done) {
  
  // 매개변수로 function을 받을 경우 수행되는 코드
    
  if (typeof fn === 'function') {
    return this._deserializers.push(fn);
  }
  
  // 매개변수로 function이 아닌 매개변수, 즉 id 그리고 콜백수행함수(function)를 받는 경우 수행되는 코드이다.
    
  // private implementation that traverses the chain of deserializers,
  // attempting to deserialize a user
  var obj = fn;

  // For backwards compatibility
  if (typeof req === 'function') {
    done = req;
    req = undefined;
  }
  
  var stack = this._deserializers;
  (function pass(i, err, user) {
    // deserializers use 'pass' as an error to skip processing
    if ('pass' === err) {
      err = undefined;
    }
    // an error or deserialized user was obtained, done
    if (err || user) { return done(err, user); }
    // a valid user existed when establishing the session, but that user has
    // since been removed
    if (user === null || user === false) { return done(null, false); }
    
    var layer = stack[i];
    if (!layer) {
      return done(new Error('Failed to deserialize user out of session'));
    }
    
    
    function deserialized(e, u) {
      pass(i + 1, e, u);
    }
    
    try {
      var arity = layer.length;
      if (arity == 3) {
        layer(req, obj, deserialized);
      } else {
        layer(obj, deserialized);
      }
    } catch(e) {
      return done(e);
    }
  })(0);
};
```

```javascript
SessionStrategy.prototype.authenticate = function(req, options) {
  if (!req.session) { return this.error(new Error('Login sessions require session support. Did you forget to use `express-session` middleware?')); }
  options = options || {};

  var self = this, 
      su;
  if (req.session[this._key]) {
    su = req.session[this._key].user;
  }

  if (su || su === 0) {      
    var paused = options.pauseStream ? pause(req) : null;
    this._deserializeUser(su, req, function(err, user) {
      if (err) { return self.error(err); }
      if (!user) {
        delete req.session[self._key].user;
      } else {
        var property = req._userProperty || 'user';
        req[property] = user;
      }
      self.pass();
      if (paused) {
        paused.resume();
      }
    });
  } else {
    self.pass();
  }
};
```

`passport.session` 이 passport.deserializeUser을 호출하면서 obj 매개변수의 값으로  `req.session[this._key].user`  을 받는다.

이 `req.session.[this._key].user`은 passport.serializeUser을 통해 `req.session.passport.user`에 의해 serialize된 정보를 받는 것이다.

passport.deserializeUser은 이 `req.session[this._key].user` 를 기반으로 
passport.deserializeUser에 푸시해둔 함수를 통해 정보를 정보를 받는다.

받은 정보를 콜백수행함수를 통해 `req.user` 에 저장한다.



이를 활용한 예제 코드도 확인해 보자.

```javascript
passport.deserializeUser((id, done) => {
      User.findOne({
        where: { id }
      })
      .then(user => done(null, user))
      .catch(err => done(err));
 });
```

위 코드에서 id는 `passport.session`이  passport.deserializeUser을 호출하면서  `req.session[this._key].user` 를 id로 받는다.

이 id를 기반으로 User 모델에서 찾은 정보를 `req.user` 에 저장한다.





> **즉 `passport.deserializeUser` 은 다음과 같이 정리할 수 있다.**
>
> 1) **최초로 Passport를 선언할 때 그리고 `passport.initialize` 메서드를 통해 요청(req)이 발생시 생성되는 passport 객체가 추후 `deserializeUser` 메서드를 어떻게 수행 할 것인가를 `passport.deserializeUser` 에 정의한다. 그렇기에 `passport.deserizlizeUser`은 `passport.initialize` 메서드가 수행되기 전에 정의되어야 하는 것이다.**
> 2) **`passport.session` 이 미들웨어로 등록하여 모든 요청(req)에서 `passport.deserializeUser` 을 사용하여 `req.user` 을 사용할 수 있도록 한다.**



####  passport.use

---

passport.use는 전략(strategy)를 어떠한 이름으로 사용할 것인지 passport에 등록하는 메소드이다.

우선 코드를 통해 passport.use를 알아보자.

```javascript
Authenticator.prototype.use = function(name, strategy) {
  if (!strategy) {
    strategy = name;
    name = strategy.name;
  }
  if (!name) { throw new Error('Authentication strategies must have a name'); }
  
  this._strategies[name] = strategy;
  return this;
};
```

위의 코드를 보면 이름을 설정하지 않으면 `strategy.name` 과 같이 기본으로 설정되는 값이 있음을 알 수 있다.

하지만 우리가 모든 전략의 기본 설정값을 알 수는 없기 때문에 이름을 설정하는 것을 추천한다.



#### passport-local

---

local 전략은 HTML 로그인 폼을 통해 전송된 정보들을 기반으로 한다.

어플리케이션은 `username`과 `password`를 받아 그것과 일치하는 정보를 데이터 베이스에서 찾는 `verify 콜백`(콜백수행함수)을 필요로 한다.

그리고 그 정보를  `user ` 객체로 전달하는 `done 콜백`(콜백함수)을 필요로 한다. 

옵션을 통해서는 정보들이 찾아지는 필드를 변경할 수 있다.

아래는 옵션 요소들이다. 

```javascript
Options:
  - `usernameField`  field name where the username is found, defaults to _username_
  - `passwordField`  field name where the password is found, defaults to _password_
  - `passReqToCallback`  when `true`, `req` is the first argument to the verify callback (default: `false`)
```



다음 코드는 passport-local을 사용하는 예제 코드이다.

```javascript
passport.use(new LocalStrategy(
{
  usernameField: 'email',
  passwordField: 'password',
}, 
// verify & done callback
async (email, password, done) => {
    try {
      const exUser = await User.findOne({ where: { email } });
      if (exUser) {
        const result = await bcrypt.compare(password, exUser.password);
        if (result) {
          done(null, exUser);
        } else {
          done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
        }
      } else {
        done(null, false, { message: '가입되지 않은 회원입니다.' });
      }
    } catch (error) {
      console.error(error);
      done(error);
    }
  }
));
```



#### passport.authenticate

---

passport.authenticate는 `strategy` 이름, 옵션 그리고 `strategy` 를 수행한 결과를 받을 콜백이 필요하다.



다음은 간단한 예시 코드이다.

```javascript
passport.authenticate('local', { successRedirect: '/', failureRedirect: '/login' })(req, res);

passport.authenticate('local', function(err, user) {
  if (!user) { return res.redirect('/login'); }
  res.end('Authenticated!');
})(req, res);
```



passport.authenticate 는 매개변수로 받은 `strategy` 를 들어오는 요청을 검증하기 위해 적용한다. 

검증이 성공적이라면 `strategy` 로 부터 콜백수행함수를 통해 유저 정보를 받아 처리한다.

이를 콜백에서 활용할 수 있는데 예시 코드는 다음과 같다.

```javascript
passport.authenticate('local', (authError, user, info) => {
  if (authError) {
    console.error(authError);
    return next(authError);
  }
  if (!user) {
    return res.send('NO EXISTING USER');
  }
  return req.login(user, (loginError) => {
    if (loginError) {
      console.error(loginError);
      return next(loginError);
    }
    return res.send("login success");
  });
})(req, res, next);
```

위의 코드는 유저정보를 `user` 매개변수를 `req.login` 에 전달하여 로그인을 수행하였다.

