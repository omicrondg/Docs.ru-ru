---
uid: signalr/overview/older-versions/handling-connection-lifetime-events
title: Понимание и обработка событий времени существования подключений в SignalR 1.x | Документация Майкрософт
author: pfletcher
description: В этой статье описывается, как использовать API для концентраторов событий.
ms.author: riande
ms.date: 06/05/2013
ms.assetid: e608e263-264d-448b-b0eb-6eeb77713b22
msc.legacyurl: /signalr/overview/older-versions/handling-connection-lifetime-events
msc.type: authoredcontent
ms.openlocfilehash: f965c38e18c442268f9bb1d7ffb5e98a135efade
ms.sourcegitcommit: 74e3be25ea37b5fc8b4b433b0b872547b4b99186
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53287688"
---
<a name="understanding-and-handling-connection-lifetime-events-in-signalr-1x"></a>Понимание и обработка событий времени существования подключений в SignalR 1.x
====================
по [Флетчера Патрик](https://github.com/pfletcher), [том Дайкстра](https://github.com/tdykstra)

[!INCLUDE [Consider ASP.NET Core SignalR](~/includes/signalr/signalr-version-disambiguation.md)]

> В этой статье Обзор SignalR подключение, повторное подключение и отключение событий, которые могут обрабатывать и параметры времени ожидания и keepalive, которые можно настроить.
> 
> В этой статье предполагается, что вы уже ознакомились со SignalR и подключения события времени жизни. Введение в SignalR, см. в разделе [SignalR — Обзор — Приступая к работе](index.md). Список событий времени существования подключений ознакомьтесь со следующими ресурсами:
> 
> - [Способ обработки событий времени существования подключений в классе концентратора](index.md)
> - [Способ обработки событий времени существования подключений в клиентах JavaScript](index.md)
> - [Способ обработки событий времени существования подключений в клиентах .NET](index.md)


## <a name="overview"></a>Обзор

В этом разделе содержатся следующие подразделы:

- [Терминология время существования подключения и сценарии](#terminology)

    - [Подключения SignalR и транспорта, физических соединений](#signalrvstransport)
    - [Сценарии отключения транспорта](#transportdisconnect)
    - [Сценарии отключения клиента](#clientdisconnect)
    - [Сценарии отключения сервера](#serverdisconnect)
- [Параметры времени ожидания и проверки активности](#timeoutkeepalive)

    - [Значение ConnectionTimeout](#connectiontimeout)
    - [DisconnectTimeout](#disconnecttimeout)
    - [KeepAlive](#keepalive)
    - [Как изменить параметры времени ожидания и проверки активности](#changetimeout)
- [Как для уведомления пользователя об отключениях](#notifydisconnect)
- [Постоянно повторное подключение к](#continuousreconnect)
- [Как отключить клиент в серверном коде](#disconnectclientfromserver)

Приведены ссылки на разделы, справочник по API .NET 4.5 версия API. Если вы используете .NET 4, см. в разделе [версия .NET 4 разделов API](https://msdn.microsoft.com/library/jj891075(v=vs.100).aspx).

<a id="terminology"></a>

## <a name="connection-lifetime-terminology-and-scenarios"></a>Терминология время существования подключения и сценарии

`OnReconnected` Обработчик событий в концентратор SignalR могут выполнять сразу после `OnConnected` , но не после `OnDisconnected` для данного клиента. У вас есть выполняется попытка повторного подключения без отключения причина в том, что существует несколько способов, в которых используется слово «соединения» в SignalR.

<a id="signalrvstransport"></a>

### <a name="signalr-connections-transport-connections-and-physical-connections"></a>Подключения SignalR и транспорта, физических соединений

В этой статье рассматриваются различия между *подключений SignalR*, *транспорта подключений*, и *физических соединений*:

- **Подключения SignalR** ссылается на логическую связь между клиентом и URL-адрес сервера, поддерживаемых SignalR API и однозначно идентифицируется идентификатор подключения. Данные об отношении обслуживается SignalR и используется для установления транспортное соединение. Конечных точках и SignalR удаляет данных, когда клиент вызывает `Stop` метода или предельное время ожидания будет достигнут, хотя SignalR пытается повторно установить потеряны транспортное соединение.
- **Транспортировки подключения,** ссылается на логическую связь между клиентом и сервером, обслуживается одним из четырех транспорта API-интерфейсы: WebSockets, события отправки сервером, неограниченно долго кадра или долгие опросы. SignalR использует транспорт API для создания транспортного соединения, а API транспорта зависит от существования физического сетевого подключения, создать транспортное соединение. Транспортное соединение завершается, когда SignalR завершает его или когда транспорт API обнаруживает, что физическое соединение разрывается.
- **Физическое подключение** относится к физической сети ссылки--провода, беспроводной сигнал, маршрутизаторы, и пр.--упрощают взаимодействие между клиентским компьютером и компьютере сервера. Физическое подключение должно присутствовать для того чтобы установить транспортное соединение, и необходимо установить подключение транспорта для подключения SignalR. Тем не менее критические физическое подключение не ограничивается всегда следуют сразу транспортного подключения или подключения SignalR, как будет описано далее в этом разделе.

На следующей схеме подключении SignalR представленный API концентраторов и слой PersistentConnection API SignalR транспортное соединение представляется на уровне транспорта и физическое соединение представляется линиями между сервером и клиентами.

![Схема архитектуры SignalR](handling-connection-lifetime-events/_static/image1.png)

При вызове `Start` метод в клиенте SignalR, вы предоставляете код клиента SignalR всю информацию, необходимые для того чтобы установить физическое подключение к серверу. Код клиента SignalR использует эти сведения, чтобы выполнить запрос HTTP и установить физическое соединение, в которых используется один из методов четыре транспорта. Если сбой подключения транспорта или на сервере происходит сбой, подключении SignalR не убрать немедленно потому, что клиент по-прежнему содержит информацию, он будет автоматически повторно установить новое подключение транспорта на один и тот же адрес SignalR. В этом сценарии участвует без вмешательства пользователя приложения, и когда клиентский код SignalR устанавливает новое транспортное подключение, он не приводит к запуску нового подключения SignalR. Непрерывность подключения SignalR отражается на самом деле, идентификатор подключения, который создается при вызове `Start` метода, остается неизменным.

`OnReconnected` Обработчик событий в концентраторе выполняется, когда транспортное соединение будет автоматически восстановлено после была потеряны. `OnDisconnected` Выполнения обработчика событий в конце подключения SignalR. Подключения SignalR можно завершить в любом из следующих способов:

- Если клиент вызывает `Stop` метод, сообщение отправляется на сервер и клиентские и серверные подключения SignalR немедленно завершен.
- После теряется связь между клиентом и сервером, клиент пытается повторно подключиться и ожидания сервера для клиента для повторного подключения. Если попытки повторного подключения завершаются неудачей и заканчивается период ожидания отключения, клиентских и серверных завершения подключения SignalR. Клиент прекращает попытки выполнить повторное подключение и сервер удаляет ее представление подключения SignalR.
- Если клиент останавливается без необходимости возможность вызвать `Stop` метод, сервер ожидает от клиента, чтобы повторно подключиться, а затем завершает соединение SignalR после истечения времени ожидания отключения.
- Если сервер останавливает выполнение, клиент пытается повторно подключиться (повторно создать подключение транспорта), а затем завершает соединение SignalR после истечения времени ожидания отключения.

Когда нет никаких проблем подключения, и приложения пользователя подключения SignalR, вызвав `Stop` метод, подключении SignalR и транспортное соединение начинается и заканчивается приблизительно за одинаковое время. В следующих разделах более подробно других сценариев.

<a id="transportdisconnect"></a>

### <a name="transport-disconnection-scenarios"></a>Сценарии отключения транспорта

Физических соединений может быть медленным, или может быть перерывы в работе с подключением. В зависимости от факторов, таких как длина прерывания могут теряться транспортное соединение. SignalR затем пытается повторно установить подключение транспорта. Иногда транспортное соединение API обнаруживает прерывания и разрывает подключение транспорта, а также SignalR о немедленно потеряно соединение. В других сценариях ни транспортного подключения API, ни SignalR стало известно, немедленно, потере соединения. Для всех транспортов за исключением долго опрашивающего клиента SignalR использует функцию с именем *keepalive* проверяемый на возможность потери подключения, используемые транспортом API не удается обнаружить. Сведения о долго опроса подключениях см. в разделе [параметры времени ожидания и keepalive](#timeoutkeepalive) далее в этом разделе.

Если подключение неактивно, периодически сервер отправляет пакета проверки активности для клиента. Начиная с даты, который выполняется запись в этой статье частота по умолчанию — каждые 10 секунд. Путем прослушивания для этих пакетов клиентам можно понять, есть проблемы с подключением. Если пакета проверки активности не получено, когда ожидалось, через некоторое время клиент предполагает, что нет проблем подключения, такие как медленная работа или перерывы в работе. Если keepalive по-прежнему не будет получено после занимает больше времени, клиент предполагает, что соединение было удалено, и она начинает повторного подключения.

На следующей схеме показано, клиентских и серверных событий, которые вызываются в типичном сценарии, если возникают проблемы с физического соединения, которые не распознаются немедленно транспорта API. Схема применяется в следующих случаях:

- Транспортом является WebSockets, неограниченно долго кадра или события отправки сервером.
- Имеются различные точки прерывания в физической сети.
- Транспорт API не по нескольким признакам перерывы в работе, поэтому SignalR полагается на функции проверки активности для их обнаружения.

![Отключения транспорта](handling-connection-lifetime-events/_static/image2.png)

Если клиент переходит в режим подключения, но не может установить транспортное соединение в течение времени ожидания отключения, сервер разрывает подключение SignalR. Когда это происходит, сервер выполняет центра `OnDisconnected` метод и помещает сообщение об отключении отправки клиенту, в случае, если клиент управляет подключиться позже. Если клиент затем подключиться повторно, он получает команды отключения и вызовы `Stop` метод. В этом случае `OnReconnected` не выполняется при повторном подключении клиента, и `OnDisconnected` выполняется только в том случае, когда клиент вызывает `Stop`. Этот сценарий проиллюстрирован на следующей схеме.

![Нарушений в работе транспорта - время ожидания сервера](handling-connection-lifetime-events/_static/image3.png)

Ниже перечислены события времени жизни подключения SignalR, которые могут возникать на стороне клиента.

- `ConnectionSlow` событие клиента.

    Вызывается, когда предустановленного доля истечения времени ожидания проверки активности прошло с момента последнего сообщения или keepalive ping было получено. По умолчанию keepalive предупреждение период ожидания равен 2/3 времени ожидания проверки активности. Время ожидания проверки активности — 20 секунд, поэтому предупреждение возникает в приблизительно 13 секунд.

    По умолчанию сервер отправляет пакеты проверки активности каждые 10 секунд, и клиент проверяет наличие проверок связи keepalive о каждые 2 секунды (треть разница между значением времени ожидания проверки активности и предупреждение значение времени ожидания проверки активности).

    Если транспорт API узнает о отключении, SignalR может знать отключения перед передачей предупреждение истечения времени ожидания проверки активности. В этом случае `ConnectionSlow` не должно порождаться событие, и SignalR бы перейти непосредственно к `Reconnecting` событий.
- `Reconnecting` событие клиента.

    Вызывается, когда (a) транспорта API обнаружит, что соединение потеряно, или (б) период ожидания проверки активности прошло с момента последнего сообщения, или keepalive ping было получено. Код клиента SignalR начинает повторного подключения. Это событие можно обработать, если требуется приложению выполнить определенное действие, при потере соединения транспорта. В настоящее время, истечения времени ожидания проверки активности по умолчанию — 20 секунд.

    Если код клиента пытается выполнить вызов метода концентратора, во время SignalR повторное подключение в режиме, SignalR будет предпринята попытка отправить команду. В большинстве случаев, такие попытки будут завершаться ошибкой, но в некоторых случаях они может быть выполнена успешно. Для события отправки сервером, неограниченно долго кадра и долго опрашивающего транспорта SignalR использует два канала связи, который клиент использует для отправки сообщений и, используемый для получения сообщений. Канал, используемый для получения та же постоянно открытыми, и тот, который закрывается при перебоях в работе физического соединения. Канал, используемый для отправки остается доступной, поэтому если восстанавливается физическое подключение, вызов метода от клиента к серверу может пройти успешно, прежде чем получить канал будет восстановлено. Возвращаемое значение будет не получен, пока не SignalR снова открывает канал, используемый для получения.
- `Reconnected` событие клиента.

    Вызывается, когда восстанавливается подключение транспорта. `OnReconnected` Выполнения обработчика событий в концентраторе.
- `Closed` событие клиента (`disconnected` событий в JavaScript).

    Вызывается, когда истечения периода ожидания отключения, а код клиента SignalR пытается повторно установить подключение после потери подключения транспорта. Значение по умолчанию отключить время ожидания составляет 30 секунд. (Это событие также возникает при завершении подключения, так как `Stop` вызывается метод.)

Нарушения подключения транспорта, которые не определяются транспортом API и не откладывать получение пакеты проверки активности на сервере дольше, чем предупреждение истечения времени ожидания проверки активности не может вызвать события времени существования любого подключения.

Некоторые сетевые среды намеренно закройте неактивных соединений, а другую функцию пакеты проверки активности — помочь избежать этого, поскольку позволяет эти сети знать, что подключения SignalR уже используется. В исключительных случаях частотой по умолчанию keepalive проверок связи может оказаться недостаточно для предотвращения закрытых подключений. В этом случае можно настроить пакеты проверки активности более часто отправляемых. Дополнительные сведения см. в разделе [параметры времени ожидания и keepalive](#timeoutkeepalive) далее в этом разделе.

> [!NOTE] 
> 
> [!IMPORTANT]
> Последовательность событий, описанные здесь не гарантируется. SignalR делает все возможное для вызова события времени жизни подключения предсказуемым образом в соответствии с этой схемой, но существует много разновидностей событий сети и множество способов, в которых их обработки базовой связи платформ, таких как транспорт API-интерфейсы. Например `Reconnected` событие не может быть вызвано при повторном подключении клиента или `OnConnected` обработчик на сервере может выполняться, когда для установления соединения не удалось. В этом разделе описывается только эффекты, которые обычно будут получены, определенных в обычных условиях.


<a id="clientdisconnect"></a>

### <a name="client-disconnection-scenarios"></a>Сценарии отключения клиента

В клиенте браузера код клиента SignalR, который будет поддерживать соединение SignalR выполняется в контексте JavaScript веб-страницы. Имеет почему в подключении SignalR для завершения при переходе от одной страницы в другую и вот почему у вас есть несколько подключений с нескольких идентификаторов подключений при подключении из нескольких окнах браузера или вкладки. Когда пользователь закрывает окно браузера или на отдельной вкладке, или переходит на новую страницу или обновляет страницу, подключении SignalR сразу же завершается потому, что код клиента SignalR обрабатывает это событие браузера и вызовы `Stop` метод. В этих сценариях или в любую клиентскую платформу, когда приложение вызывает `Stop` метод, `OnDisconnected` немедленно выполнения обработчика событий на сервере, и клиент вызывает `Closed` событий (событие с именем `disconnected` в JavaScript).

Если клиентское приложение или компьютер, на котором он выполняется на аварийно завершает работу, или переходит в спящий режим (например, когда пользователь закрывает переносном компьютере), сервер не получает уведомления о что произошло. Как сервер знал о, потери клиента может быть из-за прерывания подключения и клиент пытается получить повторного подключения. Таким образом, в этих сценариях, сервер ожидает, чтобы дать возможность повторного подключения, клиент и `OnDisconnected` не выполнено, пока не истечет время ожидания disconnect (около 30 секунд по умолчанию). Этот сценарий проиллюстрирован на следующей схеме.

![Сбой клиентского компьютера](handling-connection-lifetime-events/_static/image4.png)

<a id="serverdisconnect"></a>

### <a name="server-disconnection-scenarios"></a>Сценарии отключения сервера

Когда сервер переходит в автономный режим — он перезагружается, домена приложения в случае сбоя перезапуски, и т.д. — результат может быть аналогичен потеряно подключение или транспорта API и SignalR может сразу же узнавать, что сервер отсутствует и SignalR, может начинаться повторного подключения без вызов `ConnectionSlow` событий. Если клиент переходит в режим подключения и восстанавливает сервер или перезагрузки или нового сервера будет подключен к сети до истечения периода ожидания отключения, клиент будет подключиться к серверу восстановленным или новым. В этом случае продолжает подключения SignalR на стороне клиента и `Reconnected` события. На первом сервере `OnDisconnected` никогда не выполняется и на новом сервере, `OnReconnected` выполняется несмотря на то что `OnConnected` не была выполнена для этого клиента на этом сервере, прежде чем. (Эффект, что так же, при повторном подключении клиента на том же сервере после очистки домена приложения или перезагрузки, так как при перезапуске сервера, он не пользуется памятью действия предыдущего подключения.) Следующая диаграмма предполагает, что транспорт API узнает о потере подключения немедленно, поэтому `ConnectionSlow` событие не происходит.

![Сбой сервера и повторного подключения](handling-connection-lifetime-events/_static/image5.png)

Если сервер не станет доступен в течение периода ожидания отключения, завершении подключения SignalR. В этом случае `Closed` событий (`disconnected` в клиентах JavaScript) вызывается на стороне клиента, но `OnDisconnected` никогда не вызывается на сервере. Следующая схема предполагается, транспорт API нескольким признакам потеряно подключение, поэтому ее обнаружении по функциональным возможностям keepalive SignalR и `ConnectionSlow` события.

![Сбой сервера и время ожидания](handling-connection-lifetime-events/_static/image6.png)

<a id="timeoutkeepalive"></a>

## <a name="timeout-and-keepalive-settings"></a>Параметры времени ожидания и проверки активности

Значение по умолчанию `ConnectionTimeout`, `DisconnectTimeout`, и `KeepAlive` значения подходят для большинства сценариев, но можно изменить, если в вашей среде есть особые потребности. Например если сетевая среда закрывает подключения, которые простаивают в течение 5 секунд, может потребоваться уменьшить значение проверки активности.

<a id="connectiontimeout"></a>

### <a name="connectiontimeout"></a>Значение ConnectionTimeout

Этот параметр представляет количество времени, чтобы оставить транспортного соединения, откройте и ожидает ответа перед его закрытием и открытия нового соединения. Значение по умолчанию — 110 секунд.

Этот параметр применяется только при keepalive функция отключена, которая обычно применяется только к долго опрашивающий транспорт. На следующей схеме показано влияние этой настройки на long опроса транспортное соединение.

![Время опроса транспортного соединения](handling-connection-lifetime-events/_static/image7.png)

<a id="disconnecttimeout"></a>

### <a name="disconnecttimeout"></a>DisconnectTimeout

Этот параметр представляет время ожидания после потери транспортного соединения перед порождением `Disconnected` событий. Значение по умолчанию - 30 секунды. При задании `DisconnectTimeout`, `KeepAlive` автоматически устанавливается значение 1/3 `DisconnectTimeout` значение.

<a id="keepalive"></a>

### <a name="keepalive"></a>KeepAlive

Этот параметр представляет время ожидания перед отправкой пакета проверки активности через бездействующее подключение. Значение по умолчанию — 10 секунд. Это значение должно быть не более чем 1/3 `DisconnectTimeout` значение.

Если вы хотите установить оба `DisconnectTimeout` и `KeepAlive`, задайте `KeepAlive` после `DisconnectTimeout`. В противном случае ваши `KeepAlive` параметр будут перезаписаны при `DisconnectTimeout` автоматически задает `KeepAlive` 1/3, значения времени ожидания.

Если вы хотите отключить функцию keepalive, задайте `KeepAlive` значение null. Функция KeepAlive автоматически отключается, для долго опрашивающего транспорта.

<a id="changetimeout"></a>

### <a name="how-to-change-timeout-and-keepalive-settings"></a>Как изменить параметры времени ожидания и проверки активности

Чтобы изменить значения по умолчанию для этих параметров, установите их `Application_Start` в вашей *Global.asax* файл, как показано в следующем примере. Значения, показанные в примере кода совпадают значения по умолчанию.

[!code-csharp[Main](handling-connection-lifetime-events/samples/sample1.cs)]

<a id="notifydisconnect"></a>

## <a name="how-to-notify-the-user-about-disconnections"></a>Как для уведомления пользователя об отключениях

В некоторых приложениях может потребоваться отобразить сообщение для пользователя, при наличии проблем с подключением. У вас есть несколько вариантов, как и когда следует это сделать. Следующие образцы кода предназначены для клиента JavaScript, с помощью созданного прокси.

- Обрабатывать `connectionSlow` событий для отображения сообщения, как только SignalR знаете о проблемах подключения, прежде чем он переходит в режим подключения.

    [!code-javascript[Main](handling-connection-lifetime-events/samples/sample2.js)]
- Обрабатывать `reconnecting` событий для отображения сообщения при SignalR известно об отключении и переводится в режим повторного подключения режим.

    [!code-javascript[Main](handling-connection-lifetime-events/samples/sample3.js)]
- Обрабатывать `disconnected` истекло время ожидания событий, чтобы отобразить сообщение при попытке повторного подключения. В этом случае является единственным способом заново установить соединение с сервером еще раз перезапустить подключения SignalR, вызвав `Start` метод, который будет создан новый идентификатор подключения. В следующем образце кода использует флаг, чтобы убедиться в том, выдается уведомление только после повторного подключения времени ожидания, не после нормальным завершением подключения SignalR, из-за вызова `Stop` метод.

    [!code-javascript[Main](handling-connection-lifetime-events/samples/sample4.js)]

<a id="continuousreconnect"></a>

## <a name="how-to-continuously-reconnect"></a>Постоянно повторное подключение к

В некоторых приложениях может потребоваться автоматически повторно установить соединение после его потеряно, истекло время ожидания попытки повторного подключения. Чтобы сделать это, можно вызвать `Start` метода из вашей `Closed` обработчик событий (`disconnected` обработчиком событий для клиентов JavaScript). Вы можете захотеть подождать определенного периода времени перед вызовом `Start` Чтобы избегать слишком часто при сервера или физического соединения будут недоступны. В следующем образце кода — для клиента JavaScript, с помощью созданного прокси.

[!code-javascript[Main](handling-connection-lifetime-events/samples/sample5.js)]

Проблемы, которые следует учитывать в мобильные клиенты — попытки непрерывной повторного подключения, когда недоступен сервер или физическое соединение может привести к разряду батареи ненужные.

<a id="disconnectclientfromserver"></a>

## <a name="how-to-disconnect-a-client-in-server-code"></a>Как отключить клиент в серверном коде

SignalR версии 1.1.1 имеет встроенный сервер API для с отключением клиентов. Существуют [планы для добавления этой функции в будущем](https://github.com/SignalR/SignalR/issues/2101). В текущем выпуске SignalR самый простой способ отключить клиент от сервера является реализация метода disconnect на стороне клиента и вызов этого метода с сервера. В следующем образце кода показан метод disconnect для клиента JavaScript, с помощью созданного прокси.

[!code-javascript[Main](handling-connection-lifetime-events/samples/sample6.js)]

> [!WARNING]
> Безопасность — ни этот метод для отключения клиентов, ни предложенное встроенные API будут отвечать ситуацию, в которой взломанных клиентов, работающих под управлением вредоносный код, так как клиенты могут повторно подключиться или взломанных кода может привести к удалению `stopClient` метода или изменения его назначение. Подходящим местом для реализации с отслеживанием состояния защиты типа "отказ в обслуживании" (DOS) — не в платформы или на уровне сервера, а также в интерфейсном инфраструктуры.
