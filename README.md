# Express Yandex Money Logger


express-yandex-money-logger предоставляет middlewares для логирования запросов приложения express.js. Он использует "white list", чтобы выбрать свойства из запроса и объектов ответа.
Для передачи логов используется модуль yandex-money-logger


## Usage


### Логирование запросов

Используем `expressLogger.logger(options)` создаем middleware для логирования HTTP запросов.

``` js

    app.use(expressLogger.logger());
    app.use(app.router);
```


### Опции

``` js
    requestFilter // Пользовательский фильтр полей запроса
    responseFilter// Пользовательский фильтр полей ответа
```


## Пример

``` js
    var express = require('express');
    var expressLogger = require('express-yandex-money-logger');
    var winston = require('winston'); // for transports.Console
    var app = module.exports = express();

    app.use(express.bodyParser());
    app.use(express.methodOverride());

    // express-logger нужно установить раньше чем app.route
    app.use(expressLogger.logger());

    app.use(app.router);

    app.get('/error', function(req, res, next) {
      return next(new Error("This is an error and it should be logged to the console"));
    });

    app.get('/', function(req, res, next) {
      res.write('This is a normal request, it should be logged to the console too');
      res.end();
    });

    app.listen(3000, function(){
      console.log("express-winston demo listening on port %d in %s mode", app.address().port, app.settings.env);
    });
```

Открыв в браузере приложение мы увидим:

```
info: HTTP GET / url=/, httpVersion=1.1, originalUrl=/, , , statusCode=200, responseTime=294
info: HTTP GET /css/style.css url=/css/style.css, httpVersion=1.1, originalUrl=/css/style.css, , , statusCode=304, responseTime=3
info: HTTP GET /css/bootstrap.css url=/css/bootstrap.css, httpVersion=1.1, originalUrl=/css/bootstrap.css, , , statusCode=304, responseTime=1
```














Yandex Money Logger
========
### Логер, надстройка над модулем [winston](https://github.com/flatiron/winston)

Возвращает объект логера(winston).

Новые возможности:

* буферизация и группировка логов (yandex-money-logger-buffer)
* обертка wrapProfiler позволяет профилировать функции

Usage
=====

~~~js
var logger = require('yandex-money-logger')();
logger.log('info', 'this is a info message', { tag: 'tag-value'});
logger.info('this is a info message 2', { tag: 'tag-value'});
~~~

В конструктор можно передавать различные параметры:

 * reqCount {Number} - id запроса
 * buffering {Boolean} - флаг буферизации логов


### Буферизация

Буферизация реализована с помощью модуля yandex-money-logger-buffer

~~~js
var logger = require('yandex-money-logger')({buffering: true});
logger.info('this is a info message', { tag: 'tag-value'});
~~~

Что бы вывести логи необходимо:

~~~js
// Выведет логи в доступными транспортами
logger.flush();
// или
// Вернет объект buffer
console.log('%j', logger.getBuffer());
~~~


### Профилирование
Чтобы профилировать метод или функию необходимо ее передать аргументом в метод wrapProfiler

wrapProfiler можно передать параметры:

 * @param fn Ссылка на функцию, которую надо декорировать
 * @param params Параметры с которыми нужно вызвать функцию
 * @param namePattern Имя для идентификации в логе или шаблон имени
 * @param ctx Контекст this функции


~~~js
var profileMe = function(arguments) {
// что-то делаем...
}

logger.wrapProfiler(profileMe, args);
~~~

Результаты профилирования вернуться доступными транспортами



Yandex Money Logger Buffer
========
### Логер, совместим с модулем [winston](https://github.com/flatiron/winston)

Пишет все логи в глобальный объект buffer.
По умолчанию buffer имеет лимит в 8кб

Выводит в логи, когда объект buffer достиг лимита или вызвана команда flush

Usage
=====

``` js
var logger = require('yandex-money-logger-buffer').logBuffer();
logger.log('info', 'this is a info message', { tag: 'tag-value'});
logger.info('this is a info message 2', { tag: 'tag-value'});
```

В конструктор можно передавать различные параметры:

 * reqCount {Number} - id запроса

Чтобы вывести логи необходимо:

``` js
// Выведет логи в доступным транспортом
logger.flush();
// или
// Вернет объект buffer
console.log('%j', logger.getBuffer());
```


Чтобы увеличить или уменьшить лимит, необходимо:

``` js
require('yandex-money-logger-buffer').setBufferLimit(100000);
```

Логи сначала группируются по id воркеру (если воркеров нет, вместо id воркера будет 'master').
Потом группируются по id запроса (по умолчанию 0).


## Пример логов
``` js
Thu Oct 10 2013 19:37:54 GMT+0400 (MSK) - workerId: master - requestCount: 0 - info: log: Express server listening on port 3000 25376_pid
Thu Oct 10 2013 19:38:02 GMT+0400 (MSK) - workerId: master - requestCount: 0 - info: HTTP GET / req: {url: /, httpVersion: 1.1, originalUrl: /, , }, res: {statusCode: 200}, responseTime: 332
Thu Oct 10 2013 19:38:02 GMT+0400 (MSK) - workerId: master - requestCount: 1 - info: HTTP GET /css/style.css req: {url: /css/style.css, httpVersion: 1.1, originalUrl: /css/style.css, , }, res: {statusCode: 304}, responseTime: 4
Thu Oct 10 2013 19:38:02 GMT+0400 (MSK) - workerId: master - requestCount: 1 - info: HTTP GET /css/bootstrap.css req: {url: /css/bootstrap.css, httpVersion: 1.1, originalUrl: /css/bootstrap.css, , }, res: {statusCode: 304}, responseTime: 1
Thu Oct 10 2013 19:38:02 GMT+0400 (MSK) - workerId: master - requestCount: 2 - info: HTTP GET /users req: {url: /users, httpVersion: 1.1, originalUrl: /users, , }, res: {statusCode: 304}, responseTime: 51
Thu Oct 10 2013 19:38:02 GMT+0400 (MSK) - workerId: master - requestCount: 2 - info: list of users 
```


