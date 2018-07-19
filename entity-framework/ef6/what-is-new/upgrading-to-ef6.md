---
title: Ein Upgrade auf Entitätsframework 6
author: divega
ms.date: 2016-10-23
ms.prod: entity-framework
ms.author: divega
ms.manager: avickers
ms.technology: entity-framework-6
ms.topic: article
ms.assetid: 29958ae5-85d3-4585-9ba6-550b8ec9393a
caps.latest.revision: 3
ms.openlocfilehash: d22f0d686e6b8e3922d37245386af7723bf08ba1
ms.sourcegitcommit: bdd06c9a591ba5e6d6a3ec046c80de98f598f3f3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/10/2018
ms.locfileid: "39121480"
---
# <a name="upgrading-to-entity-framework-6"></a><span data-ttu-id="a2a81-102">Ein Upgrade auf Entitätsframework 6</span><span class="sxs-lookup"><span data-stu-id="a2a81-102">Upgrading to Entity Framework 6</span></span>

<span data-ttu-id="a2a81-103">In früheren Versionen von EF war der Code in Core-Bibliotheken (in erster Linie in "System.Data.Entity.dll"), geliefert als Bestandteil von .NET Framework und Out-of-Band (OOB) Bibliotheken (in erster Linie EntityFramework.dll) in ein NuGet-Paket ausgeliefert unterteilt.</span><span class="sxs-lookup"><span data-stu-id="a2a81-103">In previous versions of EF the code was split between core libraries (primarily System.Data.Entity.dll) shipped as part of the .NET Framework and out-of-band (OOB) libraries (primarily EntityFramework.dll) shipped in a NuGet package.</span></span> <span data-ttu-id="a2a81-104">EF6 nimmt den Code aus den Core-Bibliotheken und integriert sie in den OOB-Bibliotheken.</span><span class="sxs-lookup"><span data-stu-id="a2a81-104">EF6 takes the code from the core libraries and incorporates it into the OOB libraries.</span></span> <span data-ttu-id="a2a81-105">Dies war erforderlich, damit EF die open-Source vorgenommen werden kann und er sein eigenes Lerntempo von .NET Framework entwickeln können.</span><span class="sxs-lookup"><span data-stu-id="a2a81-105">This was necessary in order to allow EF to be made open source and for it to be able to evolve at a different pace from .NET Framework.</span></span> <span data-ttu-id="a2a81-106">Die Folge davon ist, dass Anwendungen für die verschobenen Typen neu erstellt werden müssen.</span><span class="sxs-lookup"><span data-stu-id="a2a81-106">The consequence of this is that applications will need to be rebuilt against the moved types.</span></span>

<span data-ttu-id="a2a81-107">Dies sollte für Anwendungen einfach sein, die von "DbContext" verwenden als in EF 4.1 und höher geliefert.</span><span class="sxs-lookup"><span data-stu-id="a2a81-107">This should be straightforward for applications that make use of DbContext as shipped in EF 4.1 and later.</span></span> <span data-ttu-id="a2a81-108">Ein wenig mehr Aufwand ist erforderlich, für Anwendungen, die von ObjectContext nutzen, jedoch ist es immer noch nicht schwierig.</span><span class="sxs-lookup"><span data-stu-id="a2a81-108">A little more work is required for applications that make use of ObjectContext but it still isn’t hard to do.</span></span>

<span data-ttu-id="a2a81-109">Hier ist eine Prüfliste der Dinge, die Sie zum Aktualisieren einer vorhandenen Anwendung an ef6 darstellen müssen.</span><span class="sxs-lookup"><span data-stu-id="a2a81-109">Here is a checklist of the things you need to do to upgrade an existing application to EF6.</span></span>

## <a name="1-install-the-ef6-nuget-package"></a><span data-ttu-id="a2a81-110">1. Installieren Sie das EF6-NuGet-Paket</span><span class="sxs-lookup"><span data-stu-id="a2a81-110">1. Install the EF6 NuGet package</span></span>

<span data-ttu-id="a2a81-111">Sie müssen auf der neuen Entity Framework 6-Laufzeit zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="a2a81-111">You need to upgrade to the new Entity Framework 6 runtime.</span></span>

1. <span data-ttu-id="a2a81-112">Mit der rechten Maustaste auf Ihr Projekt, und wählen Sie **NuGet-Pakete verwalten...**</span><span class="sxs-lookup"><span data-stu-id="a2a81-112">Right-click on your project and select **Manage NuGet Packages...**</span></span>  
2. <span data-ttu-id="a2a81-113">Unter den **Online** wählen Sie Registerkarte **EntityFramework** , und klicken Sie auf **installieren**</span><span class="sxs-lookup"><span data-stu-id="a2a81-113">Under the **Online** tab select **EntityFramework** and click **Install**</span></span>  
   > [!NOTE]
   > <span data-ttu-id="a2a81-114">Wenn eine frühere Version des EntityFramework NuGet-Paket installiert wurde wird dies in EF6 aktualisiert.</span><span class="sxs-lookup"><span data-stu-id="a2a81-114">If a previous version of the EntityFramework NuGet package was installed this will upgrade it to EF6.</span></span>

<span data-ttu-id="a2a81-115">Alternativ können Sie den folgenden Befehl aus Paket-Manager-Konsole ausführen:</span><span class="sxs-lookup"><span data-stu-id="a2a81-115">Alternatively, you can run the following command from Package Manager Console:</span></span>

``` powershell
Install-Package EntityFramework
```

## <a name="2-ensure-that-assembly-references-to-systemdataentitydll-are-removed"></a><span data-ttu-id="a2a81-116">2. Stellen Sie sicher, dass Assemblyverweise "System.Data.Entity.dll" entfernt werden</span><span class="sxs-lookup"><span data-stu-id="a2a81-116">2. Ensure that assembly references to System.Data.Entity.dll are removed</span></span>

<span data-ttu-id="a2a81-117">Das EF6-NuGet-Paket installieren, sollten alle Verweise auf System.Data.Entity aus Ihrem Projekt automatisch für Sie entfernen.</span><span class="sxs-lookup"><span data-stu-id="a2a81-117">Installing the EF6 NuGet package should automatically remove any references to System.Data.Entity from your project for you.</span></span>

## <a name="3-swap-any-ef-designer-edmx-models-to-use-ef-6x-code-generation"></a><span data-ttu-id="a2a81-118">3. Austauschen von EF-Designer (EDMX) Modelle zur Verwendung von EF 6.x-codegenerierung</span><span class="sxs-lookup"><span data-stu-id="a2a81-118">3. Swap any EF Designer (EDMX) models to use EF 6.x code generation</span></span>

<span data-ttu-id="a2a81-119">Wenn Sie alle Modelle, die mit dem EF Designer erstellt haben, müssen Sie so aktualisieren Sie den Code Generation Vorlagen um kompatible EF6-Code zu generieren.</span><span class="sxs-lookup"><span data-stu-id="a2a81-119">If you have any models created with the EF Designer, you will need to update the code generation templates to generate EF6 compatible code.</span></span>

> [!NOTE]
> <span data-ttu-id="a2a81-120">Es stehen derzeit nur Vorlagen mit EF 6.x DbContext Generator für Visual Studio 2012 und 2013.</span><span class="sxs-lookup"><span data-stu-id="a2a81-120">There are currently only EF 6.x DbContext Generator templates available for Visual Studio 2012 and 2013.</span></span>

1. <span data-ttu-id="a2a81-121">Löschen Sie vorhandenen Code generieren-Vorlagen.</span><span class="sxs-lookup"><span data-stu-id="a2a81-121">Delete existing code-generation templates.</span></span> <span data-ttu-id="a2a81-122">Diese Dateien werden im Regelfall  **\<Edmx_file_name\>TT** und  **\<Edmx_file_name\>. Context.tt** und geschachtelt unter Ihrer Edmx-Datei im Projektmappen-Explorer.</span><span class="sxs-lookup"><span data-stu-id="a2a81-122">These files will typically be named **\<edmx_file_name\>.tt** and **\<edmx_file_name\>.Context.tt** and be nested under your edmx file in Solution Explorer.</span></span> <span data-ttu-id="a2a81-123">Sie können die Vorlagen auswählen, im Projektmappen-Explorer, und drücken Sie die **Del** Taste, um sie zu löschen.</span><span class="sxs-lookup"><span data-stu-id="a2a81-123">You can select the templates in Solution Explorer and press the **Del** key to delete them.</span></span>  
   > [!NOTE]
   > <span data-ttu-id="a2a81-124">In Websiteprojekte werden die Vorlagen nicht geschachtelt unter der Edmx-Datei, sondern zusammen mit der sie im Projektmappen-Explorer aufgeführt.</span><span class="sxs-lookup"><span data-stu-id="a2a81-124">In Web Site projects the templates will not be nested under your edmx file, but listed alongside it in Solution Explorer.</span></span>  

   > [!NOTE]
   > <span data-ttu-id="a2a81-125">In VB.NET-Projekten müssen Sie zum Aktivieren von "Alle Dateien anzeigen", um die geschachtelte Vorlagendateien angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="a2a81-125">In VB.NET projects you will need to enable 'Show All Files' to be able to see the nested template files.</span></span>
2. <span data-ttu-id="a2a81-126">Fügen Sie die entsprechende EF 6.x Code Generation Vorlage hinzu.</span><span class="sxs-lookup"><span data-stu-id="a2a81-126">Add the appropriate EF 6.x code generation template.</span></span> <span data-ttu-id="a2a81-127">Öffnen des Modells in der EF-Designer, mit der rechten Maustaste auf die Entwurfsoberfläche, und wählen **Codegenerierungselement hinzufügen...**</span><span class="sxs-lookup"><span data-stu-id="a2a81-127">Open your model in the EF Designer, right-click on the design surface and select **Add Code Generation Item...**</span></span>
    - <span data-ttu-id="a2a81-128">Bei Verwendung der DbContext-API (empfohlen) klicken Sie dann **EF 6.x DbContext Generator** stehen unterhalb der **Daten** Registerkarte.</span><span class="sxs-lookup"><span data-stu-id="a2a81-128">If you are using the DbContext API (recommended) then **EF 6.x DbContext Generator** will be available under the **Data** tab.</span></span>  
      > [!NOTE]
      > <span data-ttu-id="a2a81-129">Wenn Sie Visual Studio 2012 verwenden, müssen Sie die Entity Framework 6-Tools für diese Vorlage installieren.</span><span class="sxs-lookup"><span data-stu-id="a2a81-129">If you are using Visual Studio 2012, you will need to install the EF 6 Tools to have this template.</span></span> <span data-ttu-id="a2a81-130">Finden Sie unter [Entity Framework erhalten](~/ef6/fundamentals/install.md) Details.</span><span class="sxs-lookup"><span data-stu-id="a2a81-130">See [Get Entity Framework](~/ef6/fundamentals/install.md) for details.</span></span>  

    - <span data-ttu-id="a2a81-131">Wenn Sie die ObjectContext-API verwenden Sie benötigen, wählen Sie die **Online** Registerkarte und suchen Sie nach **EF 6.x EntityObject Generator**.</span><span class="sxs-lookup"><span data-stu-id="a2a81-131">If you are using the ObjectContext API then you will need to select the **Online** tab and search for **EF 6.x EntityObject Generator**.</span></span>  
3. <span data-ttu-id="a2a81-132">Wenn Sie alle Anpassungen angewendet haben den Code-Generierung-Vorlagen müssen Sie sie erneut auf den aktualisierten Vorlagen anzuwenden.</span><span class="sxs-lookup"><span data-stu-id="a2a81-132">If you applied any customizations to the code generation templates you will need to re-apply them to the updated templates.</span></span>

## <a name="4-update-namespaces-for-any-core-ef-types-being-used"></a><span data-ttu-id="a2a81-133">4. Aktualisieren Sie die Namespaces für alle Core EF-Typen, die verwendet wird</span><span class="sxs-lookup"><span data-stu-id="a2a81-133">4. Update namespaces for any core EF types being used</span></span>

<span data-ttu-id="a2a81-134">Die Namespaces für "DbContext" und "Code First-Typen wurden nicht geändert werden.</span><span class="sxs-lookup"><span data-stu-id="a2a81-134">The namespaces for DbContext and Code First types have not changed.</span></span> <span data-ttu-id="a2a81-135">Dies bedeutet, dass für viele Anwendungen, mit denen EF 4.1 oder höher werden nicht müssen Sie nichts ändern.</span><span class="sxs-lookup"><span data-stu-id="a2a81-135">This means for many applications that use EF 4.1 or later you will not need to change anything.</span></span>

<span data-ttu-id="a2a81-136">Typen wie ObjectContext, die früher in "System.Data.Entity.dll" wurden in neue Namespaces verschoben.</span><span class="sxs-lookup"><span data-stu-id="a2a81-136">Types like ObjectContext that were previously in System.Data.Entity.dll have been moved to new namespaces.</span></span> <span data-ttu-id="a2a81-137">Dies bedeutet, Sie müssen möglicherweise Aktualisieren Ihrer *mit* oder *Import* Direktiven für EF6 den Build.</span><span class="sxs-lookup"><span data-stu-id="a2a81-137">This means you may need to update your *using* or *Import* directives to build against EF6.</span></span>

<span data-ttu-id="a2a81-138">Die allgemeine Regel zum Namespace-Änderungen ist, dass für jeden Typ im System.Data.* in System.Data.Entity.Core.* verschoben wird.</span><span class="sxs-lookup"><span data-stu-id="a2a81-138">The general rule for namespace changes is that any type in System.Data.* is moved to System.Data.Entity.Core.*.</span></span> <span data-ttu-id="a2a81-139">Das heißt, fügen Sie einfach **Entity.Core.**</span><span class="sxs-lookup"><span data-stu-id="a2a81-139">In other words, just insert **Entity.Core.**</span></span> <span data-ttu-id="a2a81-140">nach dem "System.Data".</span><span class="sxs-lookup"><span data-stu-id="a2a81-140">after System.Data.</span></span> <span data-ttu-id="a2a81-141">Zum Beispiel:</span><span class="sxs-lookup"><span data-stu-id="a2a81-141">For example:</span></span>

- <span data-ttu-id="a2a81-142">System.Data.EntityException = > "System.Data". **Entity.Core.** EntityException</span><span class="sxs-lookup"><span data-stu-id="a2a81-142">System.Data.EntityException => System.Data.**Entity.Core.** EntityException</span></span>  
- <span data-ttu-id="a2a81-143">System.Data.Objects.ObjectContext = > "System.Data". **Entity.Core.** Objects.ObjectContext</span><span class="sxs-lookup"><span data-stu-id="a2a81-143">System.Data.Objects.ObjectContext => System.Data.**Entity.Core.** Objects.ObjectContext</span></span>  
- <span data-ttu-id="a2a81-144">System.Data.Objects.DataClasses.RelationshipManager = > "System.Data". **Entity.Core.** Objects.DataClasses.RelationshipManager</span><span class="sxs-lookup"><span data-stu-id="a2a81-144">System.Data.Objects.DataClasses.RelationshipManager => System.Data.**Entity.Core.** Objects.DataClasses.RelationshipManager</span></span>  

<span data-ttu-id="a2a81-145">Diese Typen sind der *Core* Namespaces, da sie nicht direkt für die meisten "DbContext"-basierten Anwendungen verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="a2a81-145">These types are in the *Core* namespaces because they are not used directly for most DbContext-based applications.</span></span> <span data-ttu-id="a2a81-146">Einige Typen, die Teil von "System.Data.Entity.dll" waren häufig und direkt noch für "DbContext"-basierten Anwendungen verwendet werden und daher nicht verschoben wurden in die *Core* Namespaces.</span><span class="sxs-lookup"><span data-stu-id="a2a81-146">Some types that were part of System.Data.Entity.dll are still used commonly and directly for DbContext-based applications and so have not been moved into the *Core* namespaces.</span></span> <span data-ttu-id="a2a81-147">Diese lauten wie folgt:</span><span class="sxs-lookup"><span data-stu-id="a2a81-147">These are:</span></span>

- <span data-ttu-id="a2a81-148">System.Data.EntityState = > "System.Data". **Entität.** "EntityState"</span><span class="sxs-lookup"><span data-stu-id="a2a81-148">System.Data.EntityState => System.Data.**Entity.** EntityState</span></span>  
- <span data-ttu-id="a2a81-149">System.Data.Objects.DataClasses.EdmFunctionAttribute = > "System.Data". **Entity.DbFunctionAttribute**</span><span class="sxs-lookup"><span data-stu-id="a2a81-149">System.Data.Objects.DataClasses.EdmFunctionAttribute => System.Data.**Entity.DbFunctionAttribute**</span></span>  
  > [!NOTE]
  > <span data-ttu-id="a2a81-150">Diese Klasse wurde umbenannt; eine Klasse mit dem alten Namen noch vorhanden ist und funktioniert, aber es nun als veraltet markiert.</span><span class="sxs-lookup"><span data-stu-id="a2a81-150">This class has been renamed; a class with the old name still exists and works, but it now marked as obsolete.</span></span>  
- <span data-ttu-id="a2a81-151">System.Data.Objects.EntityFunctions = > "System.Data". **Entity.DbFunctions**</span><span class="sxs-lookup"><span data-stu-id="a2a81-151">System.Data.Objects.EntityFunctions => System.Data.**Entity.DbFunctions**</span></span>  
  > [!NOTE]
  > <span data-ttu-id="a2a81-152">Diese Klasse wurde umbenannt; eine Klasse mit dem alten Namen noch vorhanden ist und funktioniert, aber es nun als veraltet markiert.)</span><span class="sxs-lookup"><span data-stu-id="a2a81-152">This class has been renamed; a class with the old name still exists and works, but it now marked as obsolete.)</span></span>  
- <span data-ttu-id="a2a81-153">Räumliche Klassen (z. B. DbGeography, DbGeometry) wurden verschoben aus System.Data.Spatial = > "System.Data". **Entität.** Räumliche</span><span class="sxs-lookup"><span data-stu-id="a2a81-153">Spatial classes (for example, DbGeography, DbGeometry) have moved from System.Data.Spatial => System.Data.**Entity.** Spatial</span></span>

> [!NOTE]
> <span data-ttu-id="a2a81-154">Einige Typen in der System.Data-Namespace sind in "System.Data.dll", die keiner EF-Assembly ist.</span><span class="sxs-lookup"><span data-stu-id="a2a81-154">Some types in the System.Data namespace are in System.Data.dll which is not an EF assembly.</span></span> <span data-ttu-id="a2a81-155">Diese Typen nicht verschoben haben und deren Namespaces bleiben daher unverändert.</span><span class="sxs-lookup"><span data-stu-id="a2a81-155">These types have not moved and so their namespaces remain unchanged.</span></span>