---
title: Asynchrone Abfragen und speichern – EF6
author: divega
ms.date: 2016-10-23
ms.prod: entity-framework
ms.author: divega
ms.manager: avickers
ms.technology: entity-framework-6
ms.topic: article
ms.assetid: d56e6f1d-4bd1-4b50-9558-9a30e04a8ec3
caps.latest.revision: 3
ms.openlocfilehash: 655676029b3a4e42bbe257ff6091179930b1814e
ms.sourcegitcommit: 390f3a37bc55105ed7cc5b0e0925b7f9c9e80ba6
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/09/2018
ms.locfileid: "39121327"
---
# <a name="async-query-and-save"></a><span data-ttu-id="0cb0f-102">Asynchrone Abfragen und speichern</span><span class="sxs-lookup"><span data-stu-id="0cb0f-102">Async query and save</span></span>
> [!NOTE]
> <span data-ttu-id="0cb0f-103">**Nur EF6 und höher:** Die Features, APIs usw., die auf dieser Seite erläutert werden, wurden in Entity Framework 6 eingeführt.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-103">**EF6 Onwards Only** - The features, APIs, etc. discussed in this page were introduced in Entity Framework 6.</span></span> <span data-ttu-id="0cb0f-104">Wenn Sie eine frühere Version verwenden, gelten manche Informationen nicht.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-104">If you are using an earlier version, some or all of the information does not apply.</span></span>

<span data-ttu-id="0cb0f-105">EF6-Unterstützung für asynchrone Abfrage, und speichern die [Async und await-Schlüsselwörtern](https://msdn.microsoft.com/library/vstudio/hh191443.aspx) , die in .NET 4.5 eingeführt wurden.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-105">EF6 introduced support for asynchronous query and save using the [async and await keywords](https://msdn.microsoft.com/library/vstudio/hh191443.aspx) that were introduced in .NET 4.5.</span></span> <span data-ttu-id="0cb0f-106">Während der Asynchronie nicht alle Anwendungen profitieren können, kann es verwendet werden, um die Client-Reaktionsfähigkeit und die Server-Skalierbarkeit zu verbessern, bei der Verarbeitung langer "," Netzwerk "oder" e/A Aufgaben.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-106">While not all applications may benefit from asynchrony, it can be used to improve client responsiveness and server scalability when handling long-running, network or I/O-bound tasks.</span></span>

## <a name="when-to-really-use-async"></a><span data-ttu-id="0cb0f-107">Wann Asynchronität wirklich sinnvoll</span><span class="sxs-lookup"><span data-stu-id="0cb0f-107">When to really use async</span></span>

<span data-ttu-id="0cb0f-108">Der Zweck dieser exemplarischen Vorgehensweise werden die Async-Konzepte in einer Weise vorgestellt, die es einfach macht, beobachten Sie den Unterschied zwischen der Ausführung des asynchronen und synchronen Programms.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-108">The purpose of this walkthrough is to introduce the async concepts in a way that makes it easy to observe the difference between asynchronous and synchronous program execution.</span></span> <span data-ttu-id="0cb0f-109">Diese exemplarische Vorgehensweise dient nicht zur Veranschaulichung einer der wichtigsten Szenarios, bei denen asynchronen Programmierung Vorteile bietet.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-109">This walkthrough is not intended to illustrate any of the key scenarios where async programming provides benefits.</span></span>

<span data-ttu-id="0cb0f-110">Asynchrone Programmierung ist in erster Linie auf freizugeben, des aktuellen verwalteten Threads (Thread ausgeführte .NET Code) eine andere Aktion ausführen, während er für einen Vorgang wartet, die keine Compute-Zeit erfordert ausgelegt, von einem verwalteten Thread.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-110">Async programming is primarily focused on freeing up the current managed thread (thread running .NET code) to do other work while it waits for an operation that does not require any compute time from a managed thread.</span></span> <span data-ttu-id="0cb0f-111">Z. B. während eine Abfrage die Datenbank-Engine verarbeitet liegt kein vom .NET Code durchgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-111">For example, whilst the database engine is processing a query there is nothing to be done by .NET code.</span></span>

<span data-ttu-id="0cb0f-112">In Client-Anwendungen (Windows Forms, WPF, usw.) kann der aktuelle Thread verwendet werden, um die Benutzeroberfläche reaktionsfähig halten, während der asynchrone Vorgang ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-112">In client applications (WinForms, WPF, etc.) the current thread can be used to keep the UI responsive while the async operation is performed.</span></span> <span data-ttu-id="0cb0f-113">In Server-Anwendungen (ASP.NET usw.) des Threads verwendet werden kann, um andere eingehende Anforderungen - verarbeiten kann dies die arbeitsspeicherbelegung reduzieren und/oder Erhöhung des Durchsatzes des Servers.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-113">In server applications (ASP.NET etc.) the thread can be used to process other incoming requests - this can reduce memory usage and/or increase throughput of the server.</span></span>

<span data-ttu-id="0cb0f-114">Verwenden von Async in den meisten Fällen hat keine spürbare Vorteile und kann auch nachteilig sein.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-114">In most applications using async will have no noticeable benefits and even could be detrimental.</span></span> <span data-ttu-id="0cb0f-115">Verwenden Sie Tests, profilerstellung und gesundem Menschenverstand zu tun, um die Auswirkungen von Async in Ihr jeweiliges Szenario zu messen, bevor Sie endgültig.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-115">Use tests, profiling and common sense to measure the impact of async in your particular scenario before committing to it.</span></span>

<span data-ttu-id="0cb0f-116">Hier sind einige weitere Ressourcen, die Informationen zu asynchronen:</span><span class="sxs-lookup"><span data-stu-id="0cb0f-116">Here are some more resources to learn about async:</span></span>

-   [<span data-ttu-id="0cb0f-117">Übersicht über die Brandon Bray Async/await in .NET 4.5</span><span class="sxs-lookup"><span data-stu-id="0cb0f-117">Brandon Bray’s overview of async/await in .NET 4.5</span></span>](http://blogs.msdn.com/b/dotnet/archive/2012/04/03/async-in-4-5-worth-the-await.aspx)
-   <span data-ttu-id="0cb0f-118">[Asynchrone Programmierung](https://msdn.microsoft.com/library/hh191443.aspx) Seiten in der MSDN-Bibliothek</span><span class="sxs-lookup"><span data-stu-id="0cb0f-118">[Asynchronous Programming](https://msdn.microsoft.com/library/hh191443.aspx) pages in the MSDN Library</span></span>
-   <span data-ttu-id="0cb0f-119">[Wie Sie erstellen ASP.NET Web-Anwendungen mit Async](http://channel9.msdn.com/events/teched/northamerica/2013/dev-b337) (enthält eine Demo der erhöhten Durchsatz)</span><span class="sxs-lookup"><span data-stu-id="0cb0f-119">[How to Build ASP.NET Web Applications Using Async](http://channel9.msdn.com/events/teched/northamerica/2013/dev-b337) (includes a demo of increased server throughput)</span></span>

## <a name="create-the-model"></a><span data-ttu-id="0cb0f-120">Erstellen des Modells</span><span class="sxs-lookup"><span data-stu-id="0cb0f-120">Create the model</span></span>

<span data-ttu-id="0cb0f-121">Wir verwenden die [Code First-Workflow](~/ef6/modeling/code-first/workflows/new-database.md) zum Erstellen des Modells und generieren die Datenbank, jedoch auch die asynchrone Funktionalität mit allen EF-Modellen, einschließlich der mit dem EF Designer erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-121">We’ll be using the [Code First workflow](~/ef6/modeling/code-first/workflows/new-database.md) to create our model and generate the database, however the asynchronous functionality will work with all EF models including those created with the EF Designer.</span></span>

-   <span data-ttu-id="0cb0f-122">Erstellen einer Konsolenanwendung und nennen sie **AsyncDemo**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-122">Create a Console Application and call it **AsyncDemo**</span></span>
-   <span data-ttu-id="0cb0f-123">Hinzufügen des EntityFramework NuGet-Pakets</span><span class="sxs-lookup"><span data-stu-id="0cb0f-123">Add the EntityFramework NuGet package</span></span>
    -   <span data-ttu-id="0cb0f-124">Im Projektmappen-Explorer mit der Maustaste auf die **AsyncDemo** Projekt</span><span class="sxs-lookup"><span data-stu-id="0cb0f-124">In Solution Explorer, right-click on the **AsyncDemo** project</span></span>
    -   <span data-ttu-id="0cb0f-125">Wählen Sie **NuGet-Pakete verwalten...**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-125">Select **Manage NuGet Packages…**</span></span>
    -   <span data-ttu-id="0cb0f-126">Wählen Sie in das Dialogfeld "NuGet-Pakete verwalten" die **Online** Registerkarte, und wählen Sie die **EntityFramework** Paket</span><span class="sxs-lookup"><span data-stu-id="0cb0f-126">In the Manage NuGet Packages dialog, Select the **Online** tab and choose the **EntityFramework** package</span></span>
    -   <span data-ttu-id="0cb0f-127">Klicken Sie auf **installieren**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-127">Click **Install**</span></span>
-   <span data-ttu-id="0cb0f-128">Hinzufügen einer **"Model.cs"** Klasse durch die folgende Implementierung</span><span class="sxs-lookup"><span data-stu-id="0cb0f-128">Add a **Model.cs** class with the following implementation</span></span>

``` csharp
    using System.Collections.Generic;
    using System.Data.Entity;

    namespace AsyncDemo
    {
        public class BloggingContext : DbContext
        {
            public DbSet<Blog> Blogs { get; set; }
            public DbSet<Post> Posts { get; set; }
        }

        public class Blog
        {
            public int BlogId { get; set; }
            public string Name { get; set; }

            public virtual List<Post> Posts { get; set; }
        }

        public class Post
        {
            public int PostId { get; set; }
            public string Title { get; set; }
            public string Content { get; set; }

            public int BlogId { get; set; }
            public virtual Blog Blog { get; set; }
        }
    }
```

 

## <a name="create-a-synchronous-program"></a><span data-ttu-id="0cb0f-129">Erstellen Sie eine synchrone Anwendung</span><span class="sxs-lookup"><span data-stu-id="0cb0f-129">Create a synchronous program</span></span>

<span data-ttu-id="0cb0f-130">Nun, da wir ein EF-Modell verfügen, einen Code zu schreiben, der er verwendet, um einige Daten zugreift.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-130">Now that we have an EF model, let's write some code that uses it to perform some data access.</span></span>

-   <span data-ttu-id="0cb0f-131">Ersetzen Sie den Inhalt der **"Program.cs"** durch den folgenden Code</span><span class="sxs-lookup"><span data-stu-id="0cb0f-131">Replace the contents of **Program.cs** with the following code</span></span>

``` csharp
    using System;
    using System.Linq;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                PerformDatabaseOperations();

                Console.WriteLine();
                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static void PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    db.SaveChanges();

                    // Query for all blogs ordered by name
                    var blogs = (from b in db.Blogs
                                orderby b.Name
                                select b).ToList();

                    // Write all blogs out to Console
                    Console.WriteLine();
                    Console.WriteLine("All blogs:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" " + blog.Name);
                    }
                }
            }
        }
    }
```

<span data-ttu-id="0cb0f-132">Dieser Code Ruft die **PerformDatabaseOperations** Methode hierdurch eine neue **Blog** mit der Datenbank und dann Ruft alle **Blogs** aus der Datenbank und gibt sie an der **Konsole**.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-132">This code calls the **PerformDatabaseOperations** method which saves a new **Blog** to the database and then retrieves all **Blogs** from the database and prints them to the **Console**.</span></span> <span data-ttu-id="0cb0f-133">Anschließend schreibt das Programm ein Zitat des Tages die **Konsole**.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-133">After this, the program writes a quote of the day to the **Console**.</span></span>

<span data-ttu-id="0cb0f-134">Da der Code Syncronous ist, können wir den folgende Ausführungsablauf beobachten, wenn das Programm ausgeführt wird:</span><span class="sxs-lookup"><span data-stu-id="0cb0f-134">Since the code is syncronous, we can observe the following execution flow when we run the program:</span></span>

1.  <span data-ttu-id="0cb0f-135">**"SaveChanges"** beginnt mit der neuen push **Blog** in der Datenbank</span><span class="sxs-lookup"><span data-stu-id="0cb0f-135">**SaveChanges** begins to push the new **Blog** to the database</span></span>
2.  <span data-ttu-id="0cb0f-136">**"SaveChanges"** abgeschlossen ist</span><span class="sxs-lookup"><span data-stu-id="0cb0f-136">**SaveChanges** completes</span></span>
3.  <span data-ttu-id="0cb0f-137">Abfrage für alle **Blogs** an die Datenbank gesendet wird</span><span class="sxs-lookup"><span data-stu-id="0cb0f-137">Query for all **Blogs** is sent to the database</span></span>
4.  <span data-ttu-id="0cb0f-138">Abfrage zurückgibt und die Ergebnisse werden geschrieben, um **Konsole**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-138">Query returns and results are written to **Console**</span></span>
5.  <span data-ttu-id="0cb0f-139">Zitat des Tages geschrieben **Konsole**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-139">Quote of the day is written to **Console**</span></span>

![SyncOutput](~/ef6/media/syncoutput.png) 

 

## <a name="making-it-asynchronous"></a><span data-ttu-id="0cb0f-141">Asynchronen</span><span class="sxs-lookup"><span data-stu-id="0cb0f-141">Making it asynchronous</span></span>

<span data-ttu-id="0cb0f-142">Nun, wir unser Programm verfügbar sind und ausgeführt haben, können wir beginnen, machen verwenden das neue Async und await-Schlüsselwörtern.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-142">Now that we have our program up and running, we can begin making use of the new async and await keywords.</span></span> <span data-ttu-id="0cb0f-143">Wir haben die folgenden Änderungen an "Program.cs" vorgenommen.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-143">We've made the following changes to Program.cs</span></span>

1.  <span data-ttu-id="0cb0f-144">Zeile 2: Die mit-Anweisung für die **System.Data.Entity** Namespace erhalten wir Zugriff auf die EF-Async-Erweiterungsmethoden.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-144">Line 2: The using statement for the **System.Data.Entity** namespace gives us access to the EF async extension methods.</span></span>
2.  <span data-ttu-id="0cb0f-145">Zeile 4: Die mit-Anweisung für die **System.Threading.Tasks** Namespace kann wir verwenden die **Aufgabe** Typ.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-145">Line 4: The using statement for the **System.Threading.Tasks** namespace allows us to use the **Task** type.</span></span>
3.  <span data-ttu-id="0cb0f-146">Zeile 12 und 18: wir als Task, der überwacht den Status der Erfassung **PerformSomeDatabaseOperations** (Zeile 12), und klicken Sie dann blockiert die Ausführung des Programms für diese Aufgabe auf vollständige einmal alle Aufgaben für die Anwendung (Zeile 18) erfolgt.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-146">Line 12 & 18: We are capturing as task that monitors the progress of **PerformSomeDatabaseOperations** (line 12) and then blocking program execution for this task to complete once all the work for the program is done (line 18).</span></span>
4.  <span data-ttu-id="0cb0f-147">Zeile 25: Wir haben Update **PerformSomeDatabaseOperations** als markiert werden **Async** und Zurückgeben einer **Aufgabe**.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-147">Line 25: We've update **PerformSomeDatabaseOperations** to be marked as **async** and return a **Task**.</span></span>
5.  <span data-ttu-id="0cb0f-148">Zeile 35: Wir nun die asynchrone Version von "SaveChanges" aufrufen und dessen Abschluss warten.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-148">Line 35: We're now calling the Async version of SaveChanges and awaiting it's completion.</span></span>
6.  <span data-ttu-id="0cb0f-149">Zeile 42: Wir nun die asynchrone Version von "ToList" aufrufen und Warten auf das Ergebnis.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-149">Line 42: We're now calling hte Async version of ToList and awaiting on the result.</span></span>

<span data-ttu-id="0cb0f-150">Eine umfassende Liste der verfügbaren Erweiterungsmethoden im Namespace System.Data.Entity finden Sie in der QueryableExtensions-Klasse.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-150">For a comprehensive list of available extension methods in the System.Data.Entity namespace, refer to the QueryableExtensions class.</span></span> <span data-ttu-id="0cb0f-151">*Sie auch müssen hinzufügen "Verwenden von System.Data.Entity" mit Anweisungen.*</span><span class="sxs-lookup"><span data-stu-id="0cb0f-151">*You’ll also need to add “using System.Data.Entity” to your using statements.*</span></span>

``` csharp
    using System;
    using System.Data.Entity;
    using System.Linq;
    using System.Threading.Tasks;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                var task = PerformDatabaseOperations();

                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                task.Wait();

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static async Task PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    Console.WriteLine("Calling SaveChanges.");
                    await db.SaveChangesAsync();
                    Console.WriteLine("SaveChanges completed.");

                    // Query for all blogs ordered by name
                    Console.WriteLine("Executing query.");
                    var blogs = await (from b in db.Blogs
                                orderby b.Name
                                select b).ToListAsync();

                    // Write all blogs out to Console
                    Console.WriteLine("Query completed with following results:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" - " + blog.Name);
                    }
                }
            }
        }
    }
```

<span data-ttu-id="0cb0f-152">Nun, da der Code den asynchronen ist, können wir einen anderen Ausführungsfluss beobachten, wenn das Programm ausgeführt wird:</span><span class="sxs-lookup"><span data-stu-id="0cb0f-152">Now that the code is asyncronous, we can observe a different execution flow when we run the program:</span></span>

1.  <span data-ttu-id="0cb0f-153">**"SaveChanges"** beginnt mit der neuen push **Blog** in der Datenbank *nach dem Senden des Befehls in der Datenbank nicht mehr compute-Zeit ist erforderlich, für den aktuellen verwalteten Thread. Die **PerformDatabaseOperations** Methodenrückgabe (auch wenn die Ausführung noch nicht abgeschlossen) und Programmablauf in der Main-Methode wird fortgesetzt.*</span><span class="sxs-lookup"><span data-stu-id="0cb0f-153">**SaveChanges** begins to push the new **Blog** to the database *Once the command is sent to the database no more compute time is needed on the current managed thread. The **PerformDatabaseOperations** method returns (even though it hasn't finished executing) and program flow in the Main method continues.*</span></span>
2.  <span data-ttu-id="0cb0f-154">**Zitat des Tages in die Konsole geschrieben**
    *da keine weiteren Aufgaben für die Sie in der Main-Methode ist, wird der verwaltete Thread blockiert, die Wartezeit aufrufen, bis der Datenbankvorgang abgeschlossen ist. Sobald der Vorgang abgeschlossen ist, die restlichen unsere **PerformDatabaseOperations** * ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-154">**Quote of the day is written to Console**
*Since there is no more work to do in the Main method, the managed thread is blocked on the Wait call until the database operation completes. Once it completes, the remainder of our **PerformDatabaseOperations*** will be executed.</span></span>
3.  <span data-ttu-id="0cb0f-155">**"SaveChanges"** abgeschlossen ist</span><span class="sxs-lookup"><span data-stu-id="0cb0f-155">**SaveChanges** completes</span></span>
4.  <span data-ttu-id="0cb0f-156">Abfrage für alle **Blogs** wird gesendet, um die Datenbank *in diesem Fall der verwaltete Thread eine andere Aktion ausführen, während die Abfrage in der Datenbank verarbeitet werden kann. Da alle anderen Ausführung abgeschlossen ist, wird der Thread gerade jedoch auf den Aufruf warten angehalten.*</span><span class="sxs-lookup"><span data-stu-id="0cb0f-156">Query for all **Blogs** is sent to the database *Again, the managed thread is free to do other work while the query is processed in the database. Since all other execution has completed, the thread will just halt on the Wait call though.*</span></span>
5.  <span data-ttu-id="0cb0f-157">Abfrage zurückgibt und die Ergebnisse werden geschrieben, um **Konsole**</span><span class="sxs-lookup"><span data-stu-id="0cb0f-157">Query returns and results are written to **Console**</span></span>

![AsyncOutput](~/ef6/media/asyncoutput.png) 

 

## <a name="the-takeaway"></a><span data-ttu-id="0cb0f-159">Die Schlussfolgerung</span><span class="sxs-lookup"><span data-stu-id="0cb0f-159">The takeaway</span></span>

<span data-ttu-id="0cb0f-160">Wir haben jetzt gesehen, wie einfach es ist, Erstellen von asynchronen Methoden von EF verwenden.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-160">We now saw how easy it is to make use of EF’s asynchronous methods.</span></span> <span data-ttu-id="0cb0f-161">Obwohl die Vorteile der asynchronen sehr deutlich, dass mit einer einfachen Konsolen-app möglicherweise nicht, können diese Strategien in Situationen angewendet werden, in dem lang andauernden oder netzwerkgebunden Aktivitäten möglicherweise andernfalls die Anwendung blockieren oder dazu führen, dass eine große Anzahl von threads Erhöhen Sie den Speicherbedarf.</span><span class="sxs-lookup"><span data-stu-id="0cb0f-161">Although the advantages of async may not be very apparent with a simple console app, these same strategies can be applied in situations where long-running or network-bound activities might otherwise block the application, or cause a large number of threads to increase the memory footprint.</span></span>