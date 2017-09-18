---
title: "Новые возможности ASP.NET Core 1.1"
author: rick-anderson
description: "Новые возможности ASP.NET Core 1.1"
keywords: ASP.NET Core, bower
ms.author: riande
manager: wpickett
ms.date: 02/14/2017
ms.topic: article
ms.assetid: 062f8353-d1bc-4e99-a821-c1d1bb162c47
ms.technology: aspnet
ms.prod: aspnet-core
uid: aspnetcore-1.1
ms.openlocfilehash: 7fdb00bc64cb20bd0e658b3a81814059404476d2
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/11/2017
---
# <a name="whats-new-in-aspnet-core-11"></a><span data-ttu-id="037bc-104">Новые возможности ASP.NET Core 1.1</span><span class="sxs-lookup"><span data-stu-id="037bc-104">What's new in ASP.NET Core 1.1</span></span>

<span data-ttu-id="037bc-105">В состав ASP.NET Core 1.1 входят следующие новые функции и компоненты:</span><span class="sxs-lookup"><span data-stu-id="037bc-105">ASP.NET Core 1.1 includes the following new features:</span></span>

- [<span data-ttu-id="037bc-106">ПО промежуточного слоя для переопределения URL-адресов</span><span class="sxs-lookup"><span data-stu-id="037bc-106">URL Rewriting Middleware</span></span>](https://docs.microsoft.com/aspnet/core/fundamentals/url-rewriting)
- [<span data-ttu-id="037bc-107">ПО промежуточного слоя для кэширования ответов</span><span class="sxs-lookup"><span data-stu-id="037bc-107">Response Caching Middleware</span></span>](https://docs.microsoft.com/aspnet/core/performance/caching/middleware)
- [<span data-ttu-id="037bc-108">Просмотр компонентов как вспомогательных функций тегов</span><span class="sxs-lookup"><span data-stu-id="037bc-108">View Components as Tag Helpers</span></span>](xref:mvc/views/view-components#invoking-a-view-component-as-a-tag-helper)
- [<span data-ttu-id="037bc-109">ПО промежуточного слоя в качестве фильтров MVC</span><span class="sxs-lookup"><span data-stu-id="037bc-109">Middleware as MVC filters</span></span>](xref:mvc/controllers/filters#using-middleware-in-the-filter-pipeline)
- [<span data-ttu-id="037bc-110">Поставщик TempData на основе файлов cookie</span><span class="sxs-lookup"><span data-stu-id="037bc-110">Cookie-based TempData provider</span></span>](xref:fundamentals/app-state#cookie-based-tempdata-provider )
- [<span data-ttu-id="037bc-111">Поставщик ведения журнала службы приложений Azure</span><span class="sxs-lookup"><span data-stu-id="037bc-111">Azure App Service logging provider</span></span>](xref:fundamentals/logging#appservice)
- [<span data-ttu-id="037bc-112">Поставщик конфигурации Azure Key Vault</span><span class="sxs-lookup"><span data-stu-id="037bc-112">Azure Key Vault configuration provider</span></span>](xref:security/key-vault-configuration)
- [<span data-ttu-id="037bc-113">Репозитории ключей защиты данных для хранилищ Azure и Redis</span><span class="sxs-lookup"><span data-stu-id="037bc-113">Azure and Redis Storage Data Protection Key Repositories</span></span>](xref:security/data-protection/implementation/key-storage-providers#azure-and-redis)
- [<span data-ttu-id="037bc-114">Сервер WebListener для Windows</span><span class="sxs-lookup"><span data-stu-id="037bc-114">WebListener Server for Windows</span></span>](xref:fundamentals/servers/weblistener)
- [<span data-ttu-id="037bc-115">Поддержка WebSocket</span><span class="sxs-lookup"><span data-stu-id="037bc-115">WebSockets support</span></span>](xref:fundamentals/websockets)

## <a name="choosing-between-versions-10-and-11-of-aspnet-core"></a><span data-ttu-id="037bc-116">Выбор между версиями ASP.NET Core 1.0 и 1.1</span><span class="sxs-lookup"><span data-stu-id="037bc-116">Choosing between versions 1.0 and 1.1 of ASP.NET Core</span></span>

<span data-ttu-id="037bc-117">ASP.NET Core 1.1 имеет более широкий набор возможностей, чем 1.0.</span><span class="sxs-lookup"><span data-stu-id="037bc-117">ASP.NET Core 1.1 has more features than 1.0.</span></span> <span data-ttu-id="037bc-118">Как правило, мы рекомендуем использовать последнюю версию.</span><span class="sxs-lookup"><span data-stu-id="037bc-118">In general, we recommend you use the latest version.</span></span>

## <a name="additional-information"></a><span data-ttu-id="037bc-119">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="037bc-119">Additional Information</span></span>

- [<span data-ttu-id="037bc-120">Заметки о выпуске ASP.NET Core 1.1.0</span><span class="sxs-lookup"><span data-stu-id="037bc-120">ASP.NET Core 1.1.0 Release Notes</span></span>](https://github.com/aspnet/Home/releases/tag/1.1.0)
- <span data-ttu-id="037bc-121">Если вы хотите отслеживать ход работы и планы команды разработчиков ASP.NET Core, смотрите еженедельные выпуски [ASP.NET Community Standup](https://live.asp.net/).</span><span class="sxs-lookup"><span data-stu-id="037bc-121">If you’d like to connect with the ASP.NET Core development team’s progress and plans, tune in to the weekly [ASP.NET Community Standup](https://live.asp.net/).</span></span>