Yandex Money Logger
========
### Логер, надстройка над модулем [winston](https://github.com/flatiron/winston)

Возвращает объект логера(winston) настроеным через параметры.

Новые возможности:

* буферизация и груперовка логов (yandex-money-logger-buffer)
* обертка wrapProfiler позволяет профилировать функции

Usage
=====

~~~js
var logger = require('yandex-money-logger')();
logger.info('this is a info message', { tag: 'tag-value'});
~~~

в конструктор можно передавать различные параметры:

 * reqCount {Number} - id запроса
 * buffering {Boolean} - флаг буферизации логов


### Буферизация

буферизация реализована с помощью модуля yandex-money-buffer

~~~js
var logger = require('yandex-money-logger')({buffering: true});
logger.log('info', 'this is a info message', { tag: 'tag-value'});
logger.info('this is a info message 2', { tag: 'tag-value'});
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
Что бы профилировать метод или функию необходимо ее передать аргументом в метод wrapProfiler

~~~js
var profileMe = function(arguments) {
// что-то делаем...
}

logger.wrapProfiler(profileMe, args);
~~~

Результаты профелирования вернуться доступными транспортами
