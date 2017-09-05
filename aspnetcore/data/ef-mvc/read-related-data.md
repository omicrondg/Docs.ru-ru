---
title: "Основные ASP.NET MVC с основными EF - чтение связанных данных - 6 из 10"
author: tdykstra
description: "В этом учебнике вам чтения и отображения связанных данных — то есть данных, загружающий Entity Framework в свойствах навигации."
keywords: "Присоединяет ASP.NET Core, Entity Framework Core, связанные данные"
ms.author: tdykstra
manager: wpickett
ms.date: 03/15/2017
ms.topic: get-started-article
ms.assetid: 71fec30f-8ea7-4ca8-96e3-d2e26c5be44e
ms.technology: aspnet
ms.prod: asp.net-core
uid: data/ef-mvc/read-related-data
ms.openlocfilehash: d04b740f1ded3fb41ef1c3edd0adad276d8fcef0
ms.sourcegitcommit: d7e0df365a6112240b5560212759b1e3525850a2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/31/2017
---
# <a name="reading-related-data---ef-core-with-aspnet-core-mvc-tutorial-6-of-10"></a>Чтение связанных данных - Core EF учебнику ASP.NET Core MVC (6 из 10)

По [Tom Dykstra](https://github.com/tdykstra) и [Рик Андерсон](https://twitter.com/RickAndMSFT)

Contoso университета примера веб-приложения показано, как создавать веб-приложения ASP.NET MVC ядра с помощью основного Entity Framework и Visual Studio. Сведения о учебника серии см [в первом учебнике ряда](intro.md).

В предыдущем учебнике завершить модели данных School. В этом учебнике предстоит чтения и отображения связанных данных — то есть данных, загружающий Entity Framework в свойствах навигации.

На следующих рисунках страницы, вы будете работать с.

![Страница индекса курсов](read-related-data/_static/courses-index.png)

![Страница индекса инструкторов](read-related-data/_static/instructors-index.png)

## <a name="eager-explicit-and-lazy-loading-of-related-data"></a>Упреждающая, явные и отложенной загрузки связанных данных

Существует несколько способов объектно-реляционного сопоставления (ORM) программного обеспечения, например Entity Framework можно загрузить связанные данные в свойства навигации сущности:

* Безотложная загрузка. При чтении сущности связанные данные извлекаются вместе с ним. Обычно в результате запроса одного соединения, который получает все данные, которые необходимы. Указать упреждающую в Entity Framework Core, используя `Include` и `ThenInclude` методы.

  ![Пример Безотложная загрузка](read-related-data/_static/eager-loading.png)

  Вы можете получить некоторые данные в отдельных запросах и EF» по исправлению» свойства навигации.  То есть EF автоматически добавляет отдельно извлеченных сущностей, к которым они принадлежат свойств навигации ранее извлеченных объектов. Для запроса, который получает данные, можно использовать `Load` метод вместо метода, который возвращает список или объекта, таких как `ToList` или `Single`.

  ![Пример отдельных запросов](read-related-data/_static/separate-queries.png)

* Явная загрузка. При считывании сущности связанные данные не получены. Написать код, который извлекает данные при необходимости. Как в случае загрузки с отдельных запросов eager явную загрузку результатов в нескольких запросах отправляются в базу данных. Разница заключается в том, что с явная загрузка код задает свойства навигации для загрузки. В Entity Framework Core 1.1 можно использовать `Load` метод для выполнения явная загрузка. Пример:

  ![Пример явная загрузка](read-related-data/_static/explicit-loading.png)

* Отложенная загрузка. При считывании сущности связанные данные не получены. Однако первой попытке доступа к свойству навигации, данные, необходимые для этого свойства навигации автоматически извлекается. Отправляется запрос к базе данных каждый раз при попытке получить данные из свойства навигации в первый раз. Entity Framework Core 1.0 не поддерживает отложенную загрузку.

### <a name="performance-considerations"></a>Особенности производительности

Если известно, что требуются связанные данные для каждой сущности, которые получены, активная загрузка часто обеспечивает оптимальную производительность, поскольку одного запроса, отправляемые в базу данных обычно более эффективно, чем отдельные запросы для каждой сущности, которые получены. Например предположим, что для каждого отдела содержит десять связанных курсов. Безотложная загрузка всех связанных данных приведет к запрос одним (join) и одним кругового пути к базе данных. Отдельный запрос для курсов для каждого отдела приведет к одиннадцать циклов приема-передачи в базу данных. При высокой задержки, дополнительных циклов обращения к базе данных, особенно к падению производительности.

С другой стороны в некоторых сценариях отдельные запросы более эффективно. Безотложная загрузка всех связанных данных в одном запросе может привести к очень сложного соединения для создаваемого, который SQL Server не может эффективно обработать. Или, если необходимо получить доступ к свойствам навигации сущности только для подмножества набора сущностей, который обработка отдельных запросов могут выполняться эффективнее, так как Безотложная загрузка всего заранее бы получить больше данных, чем необходимо. Если важна производительность, рекомендуется проверить производительность оба способа, чтобы выбрать наиболее подходящую.

## <a name="create-a-courses-page-that-displays-department-name"></a>Создание страницы курсов, отображающий название отдела

Сущность курс включает свойство навигации, которое содержит сущности «отдел» отдела, назначенный курса. Чтобы отобразить имя подразделения, назначенный в списке курсов, необходимо получить имя свойства из сущности «отдел», который находится в `Course.Department` свойство навигации.

Контроллер с именем CoursesController для типа сущности курса, используя те же параметры для создания **контроллер MVC с представлениями, использующий Entity Framework** scaffolder, которые были ранее для контроллера учащихся, как показано в на следующем рисунке:

![Добавить контроллер курсов](read-related-data/_static/add-courses-controller.png)

Откройте *CoursesController.cs* и изучить `Index` метод. Автоматическое формирование шаблонов указал упреждающую для `Department` свойство навигации с помощью `Include` метод.

Замените `Index` метод следующим кодом, который использует более подходящее имя для `IQueryable` , возвращающий курса сущностей (`courses` вместо `schoolContext`):

[!code-csharp[Main](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_RevisedIndexMethod)]

Откройте *Views/Courses/Index.cshtml* и замените код шаблона с помощью следующего кода. Изменения выделены:

[!code-html[](intro/samples/cu/Views/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

Для формирования шаблонов код внесли следующие изменения:

* Изменить заголовок из индекса к курсам.

* Добавлен **номер** столбца, отображающего `CourseID` значение свойства. По умолчанию первичные ключи не являются формирования шаблонов, так как обычно они не имеют смысла для конечных пользователей. Однако в этом случае первичный ключ имеет смысл, и вы хотите отображать их.

* Изменить **отдел** столбец, отображающий название отдела. Код отображает `Name` свойства сущности «отдел», который загружается `Department` свойство навигации:

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

Запустите страницу (выберите вкладку курсов на домашней странице Contoso университета), чтобы просмотреть список с именами отдела.

![Страница индекса курсов](read-related-data/_static/courses-index.png)

## <a name="create-an-instructors-page-that-shows-courses-and-enrollments"></a>Создание страницы инструкторов курсов и регистрации

В этом разделе вы создадите представления сущность инструктора и контроллер, чтобы отобразить страницу «преподаватели»:

![Страница индекса инструкторов](read-related-data/_static/instructors-index.png)

Эта страница считывает и взаимосвязанные данные отображаются следующим образом:

* Появится список инструкторов взаимосвязанные данные из OfficeAssignment сущности. Сущности инструктора и OfficeAssignment являются отношением один к нулю или одному. Вы воспользуетесь упреждающую OfficeAssignment сущностей. Как упоминалось ранее, упреждающую обычно более эффективна при необходимости связанные данные для всех извлеченных строк таблицы первичного. В этом случае будет отображаться office назначения для всех отображаемых инструкторов.

* Когда пользователь выбирает инструктор, отображаются связанные сущности курса. В отношении многие ко многим являются сущности инструктора и курса. Безотложная загрузка будет использоваться для набора сущностей и их связанных сущностей отдела. В этом случае отдельные запросы могут оказаться эффективными, поскольку требуется курсы только для выбранных инструктора. Однако в этом примере показано, как использовать упреждающую свойств навигации в сущностях, которые сами в свойствах навигации.

* Когда пользователь выбирает курса, отображается взаимосвязанные данные из набора сущностей регистраций. Сущности курс и регистрации находятся в отношении один ко многим. Вы используете отдельные запросы для регистрации объектов и их связанные сущности студента.

### <a name="create-a-view-model-for-the-instructor-index-view"></a>Создать модель представлений для представления инструктора индекса

На странице инструкторов отображаются данные из трех разных таблицах. Таким образом вы создадите модель представления, которая включает три свойства каждого хранения данных для одной из таблиц.

В *SchoolViewModels* папке создайте *InstructorIndexData.cs* и замените существующий код следующим кодом:

[!code-csharp[Main](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="create-the-instructor-controller-and-views"></a>Создание контроллера инструктора и представления

Создайте инструкторов контроллер с действиями чтения и записи EF, как показано на следующем рисунке:

![Добавление контроллера преподавателей](read-related-data/_static/add-instructors-controller.png)

Откройте *InstructorsController.cs* и добавить с помощью инструкции для пространства имен ViewModels:

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Using)]

Замените метод индекс следующий код, чтобы сделать безотложной загрузкой связанных данных и поместить его в модели представления.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EagerLoading)]

Этот метод принимает необязательный маршрутизации данных (`id`) и параметр строки запроса (`courseID`), содержащие значения ID выбранного инструктора и выбранного курса. Параметры, предоставляемые **выберите** гиперссылок на странице.

Код начинается с создания экземпляра модели представления и помещения его в списке инструкторов. В коде задается упреждающую для `Instructor.OfficeAssignment` и `Instructor.CourseAssignments` свойства навигации. В `CourseAssignments` свойство, `Course` свойство загружен и в этом домене, `Enrollments` и `Department` свойства уже загружены и в каждом `Enrollment` сущности `Student` свойство загружено.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude)]

Так как представление всегда требует OfficeAssignment сущности, значительно эффективнее извлекать, в одном запросе. При инструктор выделено на веб-странице, так что один запрос лучше, чем несколько запросов только в том случае, если эта страница отображается чаще с курсом выбран чем без курса сущности не требуется.

Код повторяется `CourseAssignments` и `Course` так, как требуется два свойства из `Course`. Первая строка `ThenInclude` вызывает возвращает `CourseAssignment.Course`, `Course.Enrollments`, и `Enrollment.Student`.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=3-6)]

На этом этапе в коде другой `ThenInclude` бы для свойства навигации `Student`, которой не требуется. Но вызывающий `Include` начинает ее заново с `Instructor` свойства, поэтому вам придется проходить по цепочке еще раз, указав это время `Course.Department` вместо `Course.Enrollments`.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=7-9)]

Следующий код выполняется, когда был выбран инструктор. Выбранный инструктора извлекается из списка инструкторов в модели представления. Модель представления `Courses` свойство затем загружен с курса сущностей из этого инструктора `CourseAssignments` свойство навигации.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?range=56-62)]

`Where` Метод возвращает коллекцию, но в этом случае условия, переданные в результат метода только одну инструктора сущность возвращается. `Single` Метод преобразует коллекцию в одной сущности инструктора предоставляет доступ к этой сущности `CourseAssignments` свойство. `CourseAssignments` Свойство содержит `CourseAssignment` сущности, с которых будет осуществляться только связанные `Course` сущностей.

Вы используете `Single` метод в коллекции, если известно коллекции будет иметь только один элемент. Один метод создает исключение, если возвращается пустая коллекция, переданы в него, или если существует более одного элемента. Альтернативным вариантом является `SingleOrDefault`, который возвращает значение по умолчанию (null в данном случае), если коллекция пуста. Однако в этом случае, по-прежнему приведет к исключению (попытки найти `Courses` свойство на указатель null), и сообщение об исключении менее четко указывает на причину проблемы. При вызове `Single` метод, можно также передать в предложении Where условие вместо вызова метода `Where` метод отдельно:

```csharp
.Single(i => i.ID == id.Value)
```

вместо следующего кода:

```csharp
.Where(I => i.ID == id.Value).Single()
```

Далее Если был выбран курса, выбранного курса извлекается из списка курсов в модели представления. Затем модели представления `Enrollments` свойство загружается с регистрации сущностей из этого курса `Enrollments` свойство навигации.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?range=64-69)]

### <a name="modify-the-instructor-index-view"></a>Изменить представление Index инструктора

В *Views/Instructors/Index.cshtml*, замените код шаблона в следующий код. Изменения выделены.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=1-64&highlight=1,3-7,18-19,41-54,56)]

Следующие изменения, внесенные в существующий код:

* Класс модели, чтобы изменить `InstructorIndexData`.

* Изменить заголовок страницы из **индекс** для **инструкторов**.

* Добавлен **Office** столбец, отображающий `item.OfficeAssignment.Location` только тогда, когда `item.OfficeAssignment` не имеет значение null. (Так как один к нулю или одному связь, могут не существовать связанной сущности OfficeAssignment.)

  ```html
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* Добавлен **курсы** столбец, отображающий курсы при изучении каждого инструктора. В разделе [явные строки перехода с `@:` ](xref:mvc/views/razor#explicit-line-transition-with-label) для получения дополнительных сведений о данного синтаксиса razor.

* Добавленный код, который динамически добавляет `class="success"` для `tr` инструктора выбранного элемента. Это задает цвет фона для строк, выделенных с помощью класса начальной загрузки.

  ```html
  string selectedRow = "";
  if (item.ID == (int?)ViewData["InstructorID"])
  {
      selectedRow = "success";
  }
  <tr class="@selectedRow">
  ```

* Добавлена новая гиперссылка с меткой **выберите** непосредственно перед другие ссылки в каждой строке, что вызывает идентификатор выбранного инструктора отправку `Index` метод.

  ```html
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

Запустите приложение и перейдите на вкладку инструкторов. На странице отображается свойство Location для связанных сущностей OfficeAssignment и по пустой ячейке таблице при нет связанной сущности OfficeAssignment.

![Страница индекса инструкторов ничего не выбраны](read-related-data/_static/instructors-index-no-selection.png)

В *Views/Instructors/Index.cshtml* файла после закрытия таблицы элемента (в конце файла), добавьте следующий код. Этот код отображает список связанных с инструктор при выборе инструктор курсов.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=66-101)]

Этот код считывает `Courses` свойство модели представления для отображения списка курсов. Он также предоставляет **выберите** гиперссылку, которая отправляет идентификатор курса для `Index` метода действия.

Запустите страницу и выберите инструктор. Вы увидите на таблице, которая отображает курсов, которые назначены выбранной инструктора, и для каждого курса вы увидите имя подразделения, назначенный.

![Инструкторов индекс страницы инструктора выбран](read-related-data/_static/instructors-index-instructor-selected.png)

После только что добавленного блока кода добавьте следующий код. Это список учащихся, которые участвуют в курса при выборе этого курса.

[!code-html[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=103-125)]

Этот код считывает свойство регистраций модели представления для отображения списка учащихся, зарегистрированных в этом курсе.

Запустите страницу и выберите инструктор. Выберите курс, чтобы просмотреть список зарегистрированных студентов и их оценки.

![Инструктора страницы индекса инструкторов и выбранного курса](read-related-data/_static/instructors-index.png)

## <a name="explicit-loading"></a>Явная загрузка

При получении списка инструкторов в *InstructorsController.cs*, указан упреждающую для `CourseAssignments` свойства навигации.

Предположим, что ожидается пользователям лишь изредка могут понадобиться регистраций в выбранной инструктора и курс. В этом случае может потребоваться загрузить данные регистрации только в том случае, если она запрошена. Чтобы увидеть пример того, как сделать явная загрузка, замените `Index` метод следующим кодом, который удаляет упреждающую для регистрации и явно загружает свойства. Изменения в коде, выделяются.

[!code-csharp[Main](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ExplicitLoading&highlight=23-29)]

Новый код удаляет *ThenInclude* вызывает метод для регистрации данных из кода, которая извлекает сущности инструктора. Если выбраны инструктора и курс, выделенный код извлекает сущности регистрации выбранного курса и студента сущности для каждой регистрации.

Теперь запустите страницу индекса инструктора и вы увидите не отличается от отображаемых на странице, несмотря на то, что вы изменили способ загрузки данных.

## <a name="summary"></a>Сводка

Теперь использовалась упреждающую с одного запроса и несколько запросов на чтение связанных данных в свойства навигации. В следующем уроке будет рассмотрено обновления связанных данных.

>[!div class="step-by-step"]
>[Назад](complex-data-model.md)
>[Вперед](update-related-data.md)  