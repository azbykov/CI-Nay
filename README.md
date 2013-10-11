# Express Yandex Money Logger


express-yandex-moneylogger предоставляет middlewares для логирования запросов приложения express.js. Он использует "белые списки", чтобы выбрать свойства из запроса и объектов ответа.
Для передачи логов используется модуль yandex-money-logger


## Usage


### Логирование запросов

Используем `expressWinston.logger(options)` to create a middleware to log your HTTP requests.

``` js

    app.use(expressWinston.logger());
    app.use(app.router); // notice how the router goes after the logger.
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
      // here we cause an error in the pipeline so we see express-winston in action.
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

Возвращает объект логера(winston) настроенным через параметры.

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

в конструктор можно передавать различные параметры:

 * reqCount {Number} - id запроса
 * buffering {Boolean} - флаг буферизации логов


### Буферизация

буферизация реализована с помощью модуля yandex-money-logger-buffer

~~~js
var logger = require('yandex-money-logger')({buffering: true});
logger.info('this is a info message', { tag: 'tag-value'});
~~~

Что бы вывести логи необходимо

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


