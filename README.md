# Debugger
Инструмент для отладки приложения, использующий unix-sockets и websockets([sockJs](http://sockjs.org)).
##Компоненты
* [Collector](#collector)
* [Unix-socket сервер](#unix-socket-сервер)
* [Socket клиент](#socket-клиент)
* [Websocket сервер](#websocket-сервер)
* [Websocket клиент](#websocket-клиент)

##Зачем
Основная задача: отслеживать отладочную информацию в режиме real-time.
С сервиса отладочная информация попадает на debugger-сервер и записывается в класс Collector.


##Запуск
``` bash
  node app.js
```


##Collector
``` js
  var collector = require('./lib/collector');
```
Модуль для хранения истории сообщений. Имеет функционал для добавления новой записи и получения всех записей.
По умолчанию хранится не больше 30 записей.


``` js
    // Добавление новой записи.
    collector.push(data);
```

``` js
    // Получение данных.
    // {Array}
    var data = collector.getBuffer(); 
```
 
##Unix-socket сервер
``` js
    var server = require('./lib/socketServer');
```
Создает unix-сокет сервер. При отправке данных от клиента серверу, сервер записывает данные в коллектор.

##Socket клиент
При подключении модуль пытается подключится к серверу и экспортирует функционал добавления новой записи в коллектор.
``` js
    var debug = require('./lib/collectorClient').debug('label');
    
    debug('debug this');
```

На сервер передается объект:
``` js
   {
   	label: 'label', // Label
   	time: '+ 10ms', // Время от прошлого вызова
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
При подключении нового клиента к вебсокет серверу, сервер возваращет все содержимое коллектора.
При получении параметров от клиента, сервер возвращает содержимое коллектора, предварительно его отфильтровав.


##Websocket клиент
Это js файл подключаемый на стороне клиента. Создает соединение с вебсокет сервером и обеспечивает канал связи между клиентом и сервером.
Для его работы необходима внешняя библиотека [sockJs](http://sockjs.org).


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




