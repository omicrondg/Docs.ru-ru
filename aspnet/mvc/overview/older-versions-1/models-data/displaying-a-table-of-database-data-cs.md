---
uid: mvc/overview/older-versions-1/models-data/displaying-a-table-of-database-data-cs
title: Отображение таблицы данных в базе данных (C#) | Документация Майкрософт
author: microsoft
description: В этом руководстве описано я продемонстрирую два метода для отображения набора записей базы данных. Показать два метода форматирования набора записей базы данных в HTML-ta...
ms.author: riande
ms.date: 10/07/2008
ms.assetid: d6e758b6-6571-484d-a132-34ee6c47747a
msc.legacyurl: /mvc/overview/older-versions-1/models-data/displaying-a-table-of-database-data-cs
msc.type: authoredcontent
ms.openlocfilehash: d0d3f6a574a4b923d5da73ccb2ab3bfbd6f305ef
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "41836973"
---
<a name="displaying-a-table-of-database-data-c"></a>Отображение таблицы данных в базе данных (C#)
====================
по [Microsoft](https://github.com/microsoft)

[Загрузить PDF-файл](http://download.microsoft.com/download/1/1/f/11f721aa-d749-4ed7-bb89-a681b68894e6/ASPNET_MVC_Tutorial_11_CS.pdf)

> В этом руководстве описано я продемонстрирую два метода для отображения набора записей базы данных. Я демонстрируются два способа форматирования набора записей базы данных в таблице HTML. Во-первых я расскажу, как можно форматировать записи базы данных непосредственно в представлении. Далее я продемонстрирую, как воспользоваться преимуществами частичных представлений при форматировании записи базы данных.


Этот учебник призван объяснить, как можно отобразить HTML-таблицы базы данных в приложении ASP.NET MVC. Во-первых вы узнаете, как использовать средства формирования шаблонов, включенных в Visual Studio для создания представления, автоматически отображается набор записей. Далее вы научитесь использовать разделяемый в качестве шаблона при форматировании записи базы данных.

## <a name="create-the-model-classes"></a>Создание классов модели

Мы хотим отобразить набор записей из таблицы базы данных фильмов. В таблице базы данных фильмов содержит следующие столбцы:

<a id="0.3_table01"></a>


| **Имя столбца** | **Тип данных** | **Разрешить значения NULL** |
| --- | --- | --- |
| Идентификатор | Int | False |
| Заголовок | Nvarchar(200) | False |
| Директор | NVarchar(50) | False |
| DateReleased | DateTime | False |


Чтобы представить таблицы фильмы в приложении ASP.NET MVC, нам нужно создать класс модели. В этом руководстве мы используем Microsoft Entity Framework для создания в классах модели.

> [!NOTE] 
> 
> В этом руководстве мы используем Microsoft Entity Framework. Тем не менее важно понимать, что можно использовать множество различных технологий для взаимодействия с базой данных из приложения ASP.NET MVC, включая LINQ to SQL, NHibernate или ADO.NET.


Выполните следующие действия, чтобы запустить мастер моделей EDM.

1. Щелкните правой кнопкой мыши папку Models в окне обозревателя решений и выберите пункт меню **добавить, новый элемент**.
2. Выберите **данных** категорию и выберите **ADO.NET Entity Data Model** шаблона.
3. Назовите модель данных *MoviesDBModel.edmx* и нажмите кнопку **добавить** кнопки.

После нажатия кнопки Добавить появится мастер модели EDM (см. рис. 1). Выполните следующие действия, чтобы завершить работу мастера.

1. В **Выбор содержимого модели** выберите **создать из базы данных** параметр.
2. В **Выбор подключения к данным** шаг, используйте *MoviesDB.mdf* подключение к данным и имя *MoviesDBEntities* параметры подключения. Нажмите кнопку **Далее** кнопки.
3. В **Choose Your Database Objects** шаг, разверните узел таблицы, выберите таблицу фильмов. Введите пространство имен *моделей* и нажмите кнопку **Готово** кнопки.


[![Создание LINQ для классов SQL](displaying-a-table-of-database-data-cs/_static/image1.jpg)](displaying-a-table-of-database-data-cs/_static/image1.png)

**Рис 01**: создание классов LINQ to SQL ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image2.png))


После завершения мастера модели EDM, откроется в конструкторе моделей EDM. Конструктор должен отображать сущность фильмов (см. рис. 2).


[![В конструкторе моделей EDM](displaying-a-table-of-database-data-cs/_static/image2.jpg)](displaying-a-table-of-database-data-cs/_static/image3.png)

**Рис. 02**: конструктора моделей EDM ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image4.png))


Нам нужно внести одно изменение, перед продолжением. Мастер EDM создает класс с именем модели *фильмы* , представляющий таблицу базы данных фильмов. Так как мы будем использовать класс фильмов для представления определенного фильма, нам нужно изменить имя класса, чтобы быть *фильма* вместо *фильмы* (единственного числа, а не во множественном числе).

Дважды щелкните имя класса в области конструктора и измените имя класса из фильмов на фильм. После внесения этого изменения, нажмите кнопку **Сохранить** кнопку для создания класса Movie (значок дискеты).

## <a name="create-the-movies-controller"></a>Создание контроллера Movies

Теперь, когда у нас есть способ представления нашей записей базы данных, мы создадим контроллер, который возвращает коллекцию фильмов. В окне обозревателя решений Visual Studio щелкните правой кнопкой мыши папку Controllers и выберите пункт меню **Add, контроллера** (см. рис. 3).


[![Контроллер меню "Добавить"](displaying-a-table-of-database-data-cs/_static/image3.jpg)](displaying-a-table-of-database-data-cs/_static/image5.png)

**Рис 03**: добавить контроллер меню ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image6.png))


Когда **Добавление контроллера** откроется диалоговое окно, введите имя контроллера шаблонов для MovieController (см. рис. 4). Нажмите кнопку **добавить** кнопку, чтобы добавить новый контроллер.


[![Диалоговое окно добавления контроллера](displaying-a-table-of-database-data-cs/_static/image4.jpg)](displaying-a-table-of-database-data-cs/_static/image7.png)

**Рис. 04**: диалоговое окно добавления контроллера ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image8.png))


Нам нужно изменить действие Index(), предоставляемые контроллера Movie, чтобы он возвращал набора записей базы данных. Изменение контроллера, чтобы он выглядел контроллер в листинге 1.

**В листинге 1 — Controllers\MovieController.cs**

[!code-csharp[Main](displaying-a-table-of-database-data-cs/samples/sample1.cs)]

В листинге 1 MoviesDBEntities класс используется для представления MoviesDB базы данных. Чтобы использовать этот класс, вам потребуется импортировать пространство имен MvcApplication1.Models следующим образом:

с помощью MvcApplication1.Models;

Выражение *сущностей. MovieSet.ToList()* возвращает набор всех фильмов из таблицы базы данных фильмов.

## <a name="create-the-view"></a>Создание представления

Чтобы воспользоваться преимуществами формирования шаблонов, предоставляемых Visual Studio — самый простой способ отображения набора записей базы данных в таблице HTML.

Выполните сборку своего приложения, выбрав параметр меню **создать, собрать решение**. Необходимо построить приложение, прежде чем открывать **Добавление представления** диалогового окна или классы данных, не будет отображаться в диалоговом окне.

Щелкните правой кнопкой мыши действие Index() и выберите пункт меню **Добавление представления** (см. рис. 5).


[![Добавление представления](displaying-a-table-of-database-data-cs/_static/image5.jpg)](displaying-a-table-of-database-data-cs/_static/image9.png)

**05 рис**: Добавление представления ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image10.png))


В **Добавление представления** диалоговом окне установите флажок "с меткой" **создать строго типизированное представление**. Выберите класс Movie как **просмотреть класс данных**. Выберите *списка* как **просматривать содержимое** (см. рис. 6). Выбор этих параметров создаст представление со строгой типизацией, который отображает список фильмов.


[![Добавление представления диалогового окна](displaying-a-table-of-database-data-cs/_static/image6.jpg)](displaying-a-table-of-database-data-cs/_static/image11.png)

**Рис 06**: диалоговое окно Add View ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image12.png))


После того как вы щелкнете **добавить** автоматически создается кнопка, представление в листинге 2. Это представление содержит код, необходимый для итерации по коллекции фильмов и отображения всех свойств фильма.

**В листинге 2 — Views\Movie\Index.aspx**

[!code-aspx[Main](displaying-a-table-of-database-data-cs/samples/sample2.aspx)]

Можно запустить приложение, выбрав параметр меню **отладка, начать отладку** (или нажмите клавишу F5). Запуск приложения запускает Internet Explorer. Если перейти к URL-адрес /Movie вы увидите страницу, на рис. 7.


[![Таблица фильмов](displaying-a-table-of-database-data-cs/_static/image7.jpg)](displaying-a-table-of-database-data-cs/_static/image13.png)

**Рис 07**: таблицы фильмов ([Просмотр полноразмерного изображения](displaying-a-table-of-database-data-cs/_static/image14.png))


Если вас не устраивает, никаких сведений о внешний вид сетки, записей базы данных на рис. 7 можно просто изменить представление Index. Например, можно изменить *DateReleased* заголовок *Дата выпуска* путем изменения в представление Index.

## <a name="create-a-template-with-a-partial"></a>Создание шаблона с частичной

Когда представление получает слишком сложным, рекомендуется начать разделение представления в частичных представлений. Использование частичных представлений делает проще в понимании и обслуживании представлений. Мы создадим разделяемый, можно использовать как шаблон для форматирования каждой записи базы данных фильмов.

Выполните следующие действия для создания разделяемого.

1. Щелкните правой кнопкой мыши папку Views\Movie и выберите пункт меню **Добавление представления**.
2. Установите флажок "с меткой" *Создать разделяемое представление (ASCX)*.
3. Присвойте имя разделяемого *MovieTemplate*.
4. Установите флажок "с меткой" **создать строго типизированное представление**.
5. Выберите фильм в качестве *просмотреть класс данных*.
6. Выберите пустой как *просматривать содержимое*.
7. Нажмите кнопку **добавить** кнопку, чтобы добавить разделяемого в проект.

После выполнения этих действий, измените частичного MovieTemplate, как показано в листинге 3.

**Листинг 3 – Views\Movie\MovieTemplate.ascx**

[!code-aspx[Main](displaying-a-table-of-database-data-cs/samples/sample3.aspx)]

Partial в листинге 3 содержится шаблон для записи одной строки.

Измененный представление Index в листинге 4 использует MovieTemplate частичной.

**Листинг 4 — Views\Movie\Index.aspx**

[!code-aspx[Main](displaying-a-table-of-database-data-cs/samples/sample4.aspx)]

Представление в листинге 4 содержит цикл foreach, выполняющий перебор всех фильмов. Для каждого фильма частичного MovieTemplate используется для форматирования фильма. MovieTemplate подготавливается к просмотру путем вызова вспомогательного метода RenderPartial().

Измененный представление Index отображает те же HTML-таблицы записей базы данных. Однако представление значительно упрощена.


Метод RenderPartial() отличается от большинство других вспомогательных методов, поскольку он не возвращает строки. Таким образом, необходимо вызвать метод RenderPartial() с помощью &lt;% Html.RenderPartial(); %&gt; вместо &lt;% = Html.RenderPartial(); %&gt;.


## <a name="summary"></a>Сводка

Цель этого учебника было объяснить, как можно отобразить набор записей базы данных в таблице HTML. Во-первых вы узнали, как для возвращения набора записей базы данных из действия контроллера, используя преимущества Microsoft Entity Framework. Далее вы узнали, как использовать формирование шаблонов Visual Studio для создания представления, отображающий коллекцию элементов автоматически. Наконец вы узнали, как для упрощения просмотра, используя преимущества частичной. Вы узнали, как использовать разделяемый в качестве шаблона, таким образом, можно отформатировать каждой записи базы данных.

> [!div class="step-by-step"]
> [Назад](creating-model-classes-with-linq-to-sql-cs.md)
> [Вперед](performing-simple-validation-cs.md)
