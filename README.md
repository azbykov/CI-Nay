# Debugger
Инструмент для отладки приложения, использующий unix-sockets и websockets([sockJs](http://sockjs.org)).
##Компоненты
* [Storage](#storage)
* [Unix-socket сервер](#unix-socket-сервер)
* [Socket клиент](#socket-клиент)
* [Websocket сервер](#websocket-сервер)
* [Websocket клиент](#websocket-клиент)

##Зачем
Основная задача: отслеживать отладочную информацию в режиме real-time.
С сервиса отладочная информация попадает на debugger-сервер и записывается в класс Storage.


##Запуск
``` bash
  node index.js
```


##Storage
``` js
  var storage = require('./lib/storage');
```
Модуль для хранения записей. Имеет интерфейс для добавления и вывода записей.

Может работать в двух режимах: 
1. Normal mode. В режиме сохранения истории записей; Для реализации сохранения записей используется модуль yandex-money-cache-api.
По умолчанию записи храняться в течении 30 минут.
2. Strict mode. Без хранения истории записей.



Модуль экспортирует объект Storage, наследуемый от объекта eventEmitter, который имеет методы:
1. getAllrecords - метод получения всех записей. Возвращает массив записей. В режиме strict mode, возвращает пустой массив.

``` js
    // Получение данных.
    // {Array}
    var data = storage.getAllRecords(); 
```

Метод getAllRecords для получения всех записей вызывает метод getAllValues объекта yandex-money-cache-api.

``` js
Storage.prototype.getAllRecords = function() {
	return this.cache.getAllValues();
};
```


2. push - метод добавления новой записи. В режиме strict, генерируем событие 'newRecord' без сохранения записи.

``` js
    // Добавление новой записи.
    storage.push(data);
```

##Collector


``` js
    var server = require('./lib/collector');
```
Создает unix-сокет сервер, используя модуль net.createServer. При получении данных от collector-клиента, сервер записывает данные в хранилище, вызывая метод storage.push

``` js

/**
 * Слушаем клиент и добавляем новую запись в Storage.
 */
var data = '';
client.on('data', function(msg) {
	data += msg.toString();
	// Если сообщение пришло не полностью, то ждем оставшуюся часть.
	if (data.indexOf(DELIMITER) < 0) {
		return;
	}
	// Если сообщение пришло полностью, то разбиваем data по разделителю и добавляем в storage.
	var chunks = data.split(DELIMITER);
	for (var i = 0; i < chunks.length - 1; ++i) {
		// Добавляем новую запись в storage
	}
	// Удаляем уже отправленные в storage записи.
	data = chunks[chunks.length - 1];
});

```
При получении ошибки при работе сервера, сервер пытается заново переподключится. Количесвто попыток переподключения по умолчанию 10.
Переподключение происходит с интервалом увеличиещимуся в геометрической прогрессии.



##Collector-клиент
Реализован ввиде модуля yandex-money-debugger-client
При подключении модуль пытается подключится к модулю collector и экспортирует метод добавления новой записи в хранилище.
``` js
    var debug = require('yandex-money-debugger-client').debug('label');
    
    debug('debug this');
```
В режиме strict mode collector-клиент вешает обработчик на изменения в конфиге поля streaming.
Если streaming = enable, то клиент делает попытку соедениться с коллектором.
В остальных случаях генерируем событие end для отключения от коллектора.

Если соединение по каким-либо причинам оборвалось или соединение вернуло ошибку, то клиент пытается переподключиться к серверу.

Collector-клиент передает модулю collector объект вида:
``` js
{
	msg, 		// Текс сообщения 
	params, 	// Мета-данные сообщения
	systemParams 	// сисетмные параметры
	
	systemParams = {
		label: String,		// Метка отладчика
		timestamp: Number,	// Время когда было получено сообщение
		workerId: String,	// Id воркера
		clientIp: String,	// Ip клиента
		stringTime: String,	//Время чч:мм:сс.мс
		reqId: Number		//Id запроса
	}
}
```


##Transmitter
``` js
    var transmitter = require('./lib/transmitter');
```

Модуль создает вебсокет сервер.
При подключении нового клиента к вебсокет серверу, сервер возвращает все содержимое хранилища.

В режиме strict mode, при подключении нового клиента, модуль меняет значение поля streaming в конфиге с disable на enable.
Если все клиенты отключились от модуля transmitter, значение поля streaming на disable.

При получении параметров от клиента, сервер возвращает содержимое хранилища, предварительно его отфильтровав.
В режиме strict mode, возвращаются только записи с параметром clientIp равным ip подключившегося клиента.


##debugger.js
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
