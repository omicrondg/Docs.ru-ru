---
uid: mvc/overview/older-versions-1/controllers-and-routing/creating-an-action-cs
title: Создание действия (C#) | Документация Майкрософт
author: microsoft
description: Узнайте, как добавить новое действие контроллера ASP.NET MVC. Дополнительные сведения о требованиях для метода для действия.
ms.author: riande
ms.date: 03/02/2009
ms.assetid: cb33b28c-3025-4bd1-a1fa-eaa3af7bb56f
msc.legacyurl: /mvc/overview/older-versions-1/controllers-and-routing/creating-an-action-cs
msc.type: authoredcontent
ms.openlocfilehash: 243248ee30c6a2db7f102f7743d0393d4a6a9d24
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "41839041"
---
<a name="creating-an-action-c"></a>Создание действия (C#)
====================
по [Microsoft](https://github.com/microsoft)

> Узнайте, как добавить новое действие контроллера ASP.NET MVC. Дополнительные сведения о требованиях для метода для действия.


Чтобы объяснить, как можно создать новое действие контроллера является целью данного учебника. Дополнительные сведения о требованиях к методу действия. Вы также узнаете, как предотвратить раскрытие как действие метода.

## <a name="adding-an-action-to-a-controller"></a>Добавление действия к контроллеру

Добавьте новое действие к контроллеру, добавив новый метод к контроллеру. Например контроллер в листинге 1 содержит действия с именем Index() и действие с именем SayHello(). Оба метода доступны как действия.

**В листинге 1 - Controllers\HomeController.cs**

[!code-csharp[Main](creating-an-action-cs/samples/sample1.cs)]

Чтобы предоставить вселенной как действие, метод должны соответствовать определенным требованиям:

- Метод должен быть открытым.
- Метод не может быть статический метод.
- Метод не может быть методом расширения.
- Метод не может быть конструктор, метод получения или задания.
- Метод не может иметь открытые универсальные типы.
- Метод не является методом базового класса контроллера.
- Этот метод не может содержать **ref** или **out** параметров.

Обратите внимание на то, что не существует ограничений на тип возвращаемого значения действие контроллера. Действие контроллера может вернуть строку, DateTime, экземпляр класса Random, или void. Платформа ASP.NET MVC преобразует любой возвращаемый тип, который не является результат действия в строку и просмотру строки в браузере.

При добавлении любого метода, который не нарушает эти требования к контроллеру, метод указывается в виде действия контроллера. Следите за здесь. Действие контроллера может вызываться кем-либо подключение к Интернету. Не так, например, создать действие контроллера DeleteMyWebsite().

## <a name="preventing-a-public-method-from-being-invoked"></a>Предотвращение вызов открытого метода

Если вам нужно создать открытый метод в класс контроллера, и вы не хотите предоставлять метод как действие контроллера можно предотвратить метода вызова модуля с помощью атрибута [NonAction]. Например контроллер в листинге 2 содержит открытый метод с именем CompanySecrets(), дополнен атрибутом [NonAction].

**В листинге 2 - Controllers\WorkController.cs**

[!code-csharp[Main](creating-an-action-cs/samples/sample2.cs)]

При попытке вызвать действие контроллера CompanySecrets(), введя в адресной строке браузера /Work/CompanySecrets затем вы получите сообщение об ошибке на рис. 1.


[![Вызов метода NonAction](creating-an-action-cs/_static/image1.jpg)](creating-an-action-cs/_static/image1.png)

**Рис 01**: вызов метода NonAction ([Просмотр полноразмерного изображения](creating-an-action-cs/_static/image2.png))

> [!div class="step-by-step"]
> [Назад](creating-a-controller-cs.md)
> [Вперед](asp-net-mvc-routing-overview-vb.md)
