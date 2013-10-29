# Debugger
Инструмент для отладки приложения, используя unix-sockets и websockets(используя [sockJs](http://sockjs.org)).
* [Collector](#collector)
* [Unix-socket сервер](#unix-socket-сервер)
* [Socket клиент](#socket-клиент)
* [Websocket сервер](#websocket-сервер)
* [Websocket клиент](#websocket-клиент)

## Зачем
Основная задача. Отслеживать отладочную информацию в режиме real-time.
С сервиса, отладочная информация попадает на debugger-сервер и записывается в класс Collector



##Collector
``` js
  var collector = require('collector');
```
Модуль для храннеия истории сообщения. Имеет функционал для добавления новой записи и получения данных.
По умолчанию хранится не больше 30 записей.
Добавление новой записи.
``` js
    collector.push(data);
```
Получение данных.
``` js
    // {Array}
    var data = collector.getBuffer(); 
```
 
##Unix-socket сервер
``` js
    var server = require('./lib/socketServer');
```
Создает unix-сокет сервер. При отправки данных от клиента серверу, сервер записывает данные в коллектор.

##Socket клиент
При подключении, модуль пытается подключится к серверу и экспортирует функционал добавления новой записи в коллектор.
``` js
    var debug = require('./lib/collectorClient').debug('label');
    
    debug('debug this');
```

На сервер передается объект:
``` js
   {
   	label: 'label', // Label
   	time: + 10ms, // Время от прошлого вызова
   	workerId: wId, // Id воркера 
	clientIp: clientIp, // Ip клинета
	message: common.log({
		level : 'info', // Уровень(по умолчанию уровень всегда 'info')
		message : msg, // Текст сообщения
		meta : meta  // Мета-данные   
	})
   }
```

Если соединение по каким-либо причинам оборвалось, то клиент пытается переподключится к серверу каждые 3 секунды 


##Websocket сервер
``` js
    var webSocketServer = require('./lib/webSocketServer');
```
Модуль создает вебсокет сервер.
При подключении новго клиента к вебсокет серверу, север возваращет все содержимое колектора
При получении параметров от клиента, сервер возвращает содержимое колектора предварительно его отфильтровав.


##Websocket клиент
Это js файл подключаемый на стороне клиента. Обеспечивает соединение с вебсокет сервером и транслирует данные от сервера.
Для его работы необходима внешняя библиотека [sockJs](http://sockjs.org)


``` js
    var sock = new SockJS('http://localhost:3001/debugger');
    sock.onopen = function() {
        console.log('open debugger');
    };
    sock.onmessage = function(e) {
        // действия при получении новый порции данных.
    };
    sock.onclose = function() {
        console.log('close debugger');
    };
```




