### Multer

---



#### Multer 이해하기

---



> Multer is a node.js middleware for handling `multipart/form-data`, which is primarily used for uploading files. It is written on top of [busboy](https://github.com/mscdex/busboy) for maximum efficiency.
>
> **NOTE**: Multer will not process any form which is not multipart (`multipart/form-data`).



위는 multer의 공식문서이다.

multer는 node.js의 `multipart/form-data`를 다루기 위한 미들웨어이다.

이는 `busboy` 에 기반해서 최대한의 효율을 끌어내기 위해 만들어졌다.

주의할 것은 multer는 `multipart/form-data`외의 `form`에는 사용되지 않는다는 것이다.



multer는 `request` 객체의 `body` 에  `file` 혹은 `files` 를 추가한다. (`req.file` , `req.files`)

`body` 객체는 `form`의 text fields 값을 포함하고 `file` 혹은 `files` 은 `form` 을 통해 업로드 되는 파일들을 포함한다.



```javascript
function multer (options) {
  if (options === undefined) {
    return new Multer({})
  }

  if (typeof options === 'object' && options !== null) {
    return new Multer(options)
  }

  throw new TypeError('Expected object for argument options')
}
```

```javascript
function Multer (options) {
  if (options.storage) {
    this.storage = options.storage
  } else if (options.dest) {
    this.storage = diskStorage({ destination: options.dest })
  } else {
    this.storage = memoryStorage()
  }

  this.limits = options.limits
  this.preservePath = options.preservePath
  this.fileFilter = options.fileFilter || allowAll
}
```

위의 2가지 코드는 multer을 사용할 때 생성되는 multer 객체 생성하는 코드이다.

첫 번째 코드를 통해 multer는 그 객체를 생성할 때부터 `options` 을 설정할 수 있다는 것이다.

그 `options` 으로는 `storage` 와 `dest` 그리고 `limits`  , `preservePath` , `fileFilter` 가 있다.



`options` 를 설정한 multer 예제 코드를 보자.

```javascript
const upload = multer({
    storage: multer.diskStorage({
        destination(req, file, cb) {
            cb(null, 'uploads/');
        },
        filename(req, file, cb) {
            const ext = path.extname(file.originalname);
            cb(null, path.basename(file.originalname, ext) + Date.now() + ext);
        },
    }),
    limits: { fileSize: 5 * 1024 * 1024 },
});
```

위의 코드에서는 `storage` 와 `limits` 가 사용되었다.

위에서 본인이 주목한 것은 `multer.diskStroage` 와 multer 생성자 객체를 `upload` 라는 변수에 담았다는 것이다.



우선 `multer.diskStroage` 부터 알아보자.

`multer.diskStroage` 에서 알 수 있듯 이는 multer의 생성자 메서드이다.

이를 통해 `storage` 옵션에 제공할 알맞은 객체를 생성할 수 있다.

```javascript
function DiskStorage (opts) {
  this.getFilename = (opts.filename || getFilename)

  if (typeof opts.destination === 'string') {
    mkdirp.sync(opts.destination)
    this.getDestination = function ($0, $1, cb) { cb(null, opts.destination) }
  } else {
    this.getDestination = (opts.destination || getDestination)
  }
}
```

`getFilename` 은 옵션으로 제공한 콜백함수를 저장한다.

`getDestination`  역시 제공한 콜백함수를 저장한다.



`getDestination` 의 경우 아래 코드를 통해 조금 더 자세히 알아보자.

```javascript
DiskStorage.prototype._handleFile = function _handleFile (req, file, cb) {
  var that = this

  that.getDestination(req, file, function (err, destination) {
    if (err) return cb(err)

    that.getFilename(req, file, function (err, filename) {
      if (err) return cb(err)

      var finalPath = path.join(destination, filename)
      var outStream = fs.createWriteStream(finalPath)

      file.stream.pipe(outStream)
      outStream.on('error', cb)
      outStream.on('finish', function () {
        cb(null, {
          destination: destination,
          filename: filename,
          path: finalPath,
          size: outStream.bytesWritten
        })
      })
    })
  })
}
```

위의 모든 코드를 이해할 수는 없지만 `getDestination` 는 콜백수행함수를 실행한다. 

그 속에서 `getFilename` 의 콜백수행함수도 실행하는데 이를 통해 `filename` 에 관한 정보를 받고 위의 `getDestination` 의 콜백수행함수를 통해 받은 `destination` 과 함께 경로를 확정한다.



위와 같은 과정을 통해 multer의 `storage` 옵션에 제공할 객체를 생성할 수 있는 것이다.



이제 multer 생성자 객체를 `upload` 라는 변수은 이유에 관해 알아보자.

```javascript
router.post('/image', upload.single("fieldName"), function(req,res){
    console.log(req.file)
    res.send(req.file)
});
```

이는`upload `변수를 활용한 예제 코드이다. 

`upload.single()` multer 생성자를 통해 생성한 객체의 인스턴스 메서드를 사용하였다.

multer 생성자 객체를 `upload` 라는 변수은 이유 multer의 인스턴스 메서드를 활용하기 위해서 이다.



multer의 인스턴스 메서드는 `upload.single()` , `upload.array()` , `upload.fields()` , `upload.none()` 그리고 `upload.any()` 가 있다.

이중 자주 쓰이는 메서드는 `upload.single()` , `upload.array()` 그리고 `upload.fields()` 이다.

본인은 위의 3가지 메서드들이 어떻게 구현되는지 아래 코드를 통해 알아보고자 한다.

```javascript
Multer.prototype._makeMiddleware = function (fields, fileStrategy) {
  function setup () {
    var fileFilter = this.fileFilter
    var filesLeft = Object.create(null)

    fields.forEach(function (field) {
      if (typeof field.maxCount === 'number') {
        filesLeft[field.name] = field.maxCount
      } else {
        filesLeft[field.name] = Infinity
      }
    })

    function wrappedFileFilter (req, file, cb) {
      if ((filesLeft[file.fieldname] || 0) <= 0) {
        return cb(new MulterError('LIMIT_UNEXPECTED_FILE', file.fieldname))
      }

      filesLeft[file.fieldname] -= 1
      fileFilter(req, file, cb)
    }

    return {
      limits: this.limits,
      preservePath: this.preservePath,
      storage: this.storage,
      fileFilter: wrappedFileFilter,
      fileStrategy: fileStrategy
    }
  }

  return makeMiddleware(setup.bind(this))
}
```

```javascript
Multer.prototype.single = function (name) {
  return this._makeMiddleware([{ name: name, maxCount: 1 }], 'VALUE')
}

Multer.prototype.array = function (name, maxCount) {
  return this._makeMiddleware([{ name: name, maxCount: maxCount }], 'ARRAY')
}

Multer.prototype.fields = function (fields) {
  return this._makeMiddleware(fields, 'OBJECT')
}

// 참고
Multer.prototype.none = function () {
  return this._makeMiddleware([], 'NONE')
}

Multer.prototype.any = function () {
  function setup () {
    return {
      limits: this.limits,
      preservePath: this.preservePath,
      storage: this.storage,
      fileFilter: this.fileFilter,
      fileStrategy: 'ARRAY'
    }
  }

  return makeMiddleware(setup.bind(this))
}
```

한 가지 주목할 만한 것은 3가지 메서드 모두 `_makeMiddleware` 를 사용한다는 것이다.

`_makeMiddleware` 에  공통적으로 `name` 과 `maxCount` 값을 배열의 형태로 제공한다.



```javascript
fields.forEach(function (field) {
  if (typeof field.maxCount === 'number') {
    filesLeft[field.name] = field.maxCount
  } else {
    filesLeft[field.name] = Infinity
  }
})
```

이는 위의 `fields.forEach` 를 위한 것이다.

배열을 통해 받은 `name` 과 `maxCount` 를 활용하여 위에서 설정한 `filesLeft` 라는 빈 객체에 기록되고 `wrappedFileFilter` 함수를 통해  `fileFilter` 에 그 값이 저장된다.

이는 `return makeMiddleware(setup.bind(this))` 를 통해 `makeMiddleware` 에 전달되는데 `makeMiddleware` 에서 `busboy` 를 통해 정보를 처리한다.



#### Multer 활용하기

---

multer을 활용하기 앞서 multer는 node.js의 `multipart/form-data`를 다루기 위한 미들웨어이라는 것이라는 이해를 확실히 해야 한다.

즉, `multipart/form-data` 요청이 있는 경우에만 사용되기 때문에 다른 미들웨어처럼 `app.use()` 를 사용하여 등록할 필요가 없다는 것이다.

`multipart/form-data` 를 `post` 하는 경우에만 미들웨어를 사용하면 되는 것이다.

그렇기에 낯선 상황이 펼쳐진다. 

`multipart/form-data` 요청이 존재하는 라우터에서 multer 객체를 생성해 주어야 한다는 것이다. 

```javascript
var express = require('express'),
    router = express.Router(),
    multer  = require('multer'),
    path = require('path'),
    fs = require('fs');

try {
    fs.readdirSync('uploads');
  } catch (error) {
    console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
    fs.mkdirSync('uploads');
  }
  
const upload = multer({
    storage: multer.diskStorage({
        destination(req, file, cb) {
            cb(null, 'uploads/');
        },
        filename(req, file, cb) {
            const ext = path.extname(file.originalname);
            cb(null, path.basename(file.originalname, ext) + Date.now() + ext);
        },
    }),
    limits: { fileSize: 5 * 1024 * 1024 },
});

router.post('/image', upload.single("fieldName"), function(req,res){
    console.log(req.file)
    res.send(req.file)
});

module.exports = router;
```

즉, 위와 같이 multer 객체를 생성해 주어야 하는 것이다.

하지만 위와 같이 multer 객체를 생성해 주는 것은 보기 좋은 코드가 아니라는 생각이 들었다.

이를 해결하기 위해서는 multer class를 만들고 class를 통해 객체를 만들어 그 객체를 필요할 때 제공하는 것이 좋겠다는 생각을 하고 아래와 같이 구현해 보았다.

```javascript
// src/middleware/Multer.js

const multer  = require('multer'),
    path = require('path'),
    fs = require('fs');

class MulterClass {

    constructor() {
        try {
            fs.readdirSync('uploads');
            } catch (error) {
            console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
            fs.mkdirSync('uploads');
            }
            
        this.upload = multer({
            storage: multer.diskStorage({
                destination(req, file, cb) {
                    cb(null, 'uploads/');
                },
                filename(req, file, cb) {
                    const ext = path.extname(file.originalname);
                    cb(null, path.basename(file.originalname, ext) + Date.now() + ext);
                },
            }),
            limits: { fileSize: 5 * 1024 * 1024 },
        });
    }
        
    single(fieldName) {
        return this.upload.single(fieldName)
    }

    array(fieldName, count) {
        return this.upload.array(fieldName, count)
    }
    
    fields (arrayField) {
        return this.upload.fields(arrayField)
    } 
}
    
    
module.exports = MulterClass
```

 우선 Multer Class이다. 

`constructor()` 를 통해 `uploads` 폴더의 유무를 확인하고 변수 `upload`를 정의하였다.

변수 `upload` 는 multer를 통해 생성한 객체를 가지고 있다.

이 변수 `upload` 를 활용하여 Multer Class의 함수들을 정의한다.



이렇게 정의한 Multer Class를 통해 아래와 같이 Multer Object를 생성한다.

```javascript
// src/middleware/multerObj.js

var Multer = require("./Multer");

multerObj = new Multer();

exports.single = (fieldName) => {
    return multerObj.single(fieldName);
}

exports.array = (fieldName, count) => {
    return multerObj.array(fieldName, count);
}

exports.fields = (array) => {
    return multerObj.fields(array);
}
```

 Multer Class를 Multer 변수로 담아와 Multer 객체를 생성한다.

이는 multerobj 변수에 저장한다.

Multer 객체를 생성하여 Multer Class를 정의하며 만들었던 인스턴스 함수들을 사용할 수 있어  그것들을 활용한 함수들을 위와 같이 만든다.



이렇게 생성한 함수들을 아래와 같이 불러와 사용한다.

```javascript
var express = require('express'),
    router = express.Router();

var multer = require("../middlewares/multerObj");

router.post('/image', multer.single("fieldName"), function(req,res){
    console.log(req.file)
    res.send(req.file)
});

module.exports = router;
```

위를 보면 기존에 코드에 비해 깔끔해진 모습을 볼 수 있다.
