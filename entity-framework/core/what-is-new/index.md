---
title: "Neue Features in EF Core 2.0 – EF Core"
author: divega
ms.author: divega
ms.date: 10/27/2016
ms.assetid: C21F89EE-FB08-4ED9-A2A0-76CB7656E6E4
ms.technology: entity-framework-core
uid: core/what-is-new/index
ms.openlocfilehash: a0e54fe85e810793913b9c5fb5e745419a0983b2
ms.sourcegitcommit: 01a75cd483c1943ddd6f82af971f07abde20912e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/27/2017
---
# <a name="new-features-in-ef-core-20"></a><span data-ttu-id="b9e1f-102">Neue Features in EF Core 2.0</span><span class="sxs-lookup"><span data-stu-id="b9e1f-102">New features in EF Core 2.0</span></span>

## <a name="net-standard-20"></a><span data-ttu-id="b9e1f-103">.NET-Standard 2.0</span><span class="sxs-lookup"><span data-stu-id="b9e1f-103">.NET Standard 2.0</span></span>
<span data-ttu-id="b9e1f-104">Für EF Core wird nun die Zielversion für .NET Standard 2.0 festgelegt, was bedeutet, dass EF Core mit .NET Core 2.0, .NET Framework 4.6.1 und anderen Bibliotheken, die.NET Standard 2.0 implementieren, kompatibel ist.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-104">EF Core now targets .NET Standard 2.0, which means it can work with .NET Core 2.0, .NET Framework 4.6.1, and other libraries that implement .NET Standard 2.0.</span></span>
<span data-ttu-id="b9e1f-105">Weitere Informationen zu unterstützten Implementierungen finden Sie unter [Unterstützte .NET-Implementierungen](../platforms/index.md).</span><span class="sxs-lookup"><span data-stu-id="b9e1f-105">See [Supported .NET Implementations](../platforms/index.md) for more details on what is supported.</span></span>

## <a name="modeling"></a><span data-ttu-id="b9e1f-106">Modellierung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-106">Modeling</span></span>

### <a name="table-splitting"></a><span data-ttu-id="b9e1f-107">Tabellenaufteilung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-107">Table splitting</span></span>

<span data-ttu-id="b9e1f-108">Nun können mindestens zwei Entitätstypen derselben Tabelle zugeordnet werden, wobei Primärschlüsselspalten gemeinsam genutzt werden und jede Zeile mindestens zwei Entitäten entspricht.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-108">It is now possible to map two or more entity types to the same table where the primary key column(s) will be shared and each row will correspond to two or more entities.</span></span>

<span data-ttu-id="b9e1f-109">Für die Durchführung der Tabellenaufteilung muss eine identifizierende Beziehung (wobei die Fremdschlüsseleigenschaften den Primärschlüssel bilden) zwischen allen Entitätstypen konfiguriert werden, die die Tabelle gemeinsam nutzen:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-109">To use table splitting an identifying relationship (where foreign key properties form the primary key) must be configured between all of the entity types sharing the table:</span></span>

``` csharp
modelBuilder.Entity<Product>()
    .HasOne(e => e.Details).WithOne(e => e.Product)
    .HasForeignKey<ProductDetails>(e => e.Id);
modelBuilder.Entity<Product>().ToTable("Products");
modelBuilder.Entity<ProductDetails>().ToTable("Products");
```

### <a name="owned-types"></a><span data-ttu-id="b9e1f-110">Eigene Typen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-110">Owned types</span></span>

<span data-ttu-id="b9e1f-111">Ein eigener Entitätstyp kann denselben CLR-Typ mit einem anderen eigenen Entitätstyp verwenden, aber da dieser nicht allein durch den CLR-Typ identifiziert werden kann, muss eine Navigation von einem anderen Entitätstyp zu diesem erfolgen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-111">An owned entity type can share the same CLR type with another owned entity type, but since it cannot be identified just by the CLR type there must be a navigation to it from another entity type.</span></span> <span data-ttu-id="b9e1f-112">Die Entität, die die definierende Navigation enthält, ist der Besitzer.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-112">The entity containing the defining navigation is the owner.</span></span> <span data-ttu-id="b9e1f-113">Beim Abfragen des Besitzers werden standardmäßig eigene Typen eingeschlossen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-113">When querying the owner the owned types will be included by default.</span></span>

<span data-ttu-id="b9e1f-114">Gemäß den Konventionen wird ein Schattenprimärschlüssel für den eigenen Typ erstellt und mithilfe der Tabellenaufteilung der gleichen Tabelle wie der des Besitzers zugeordnet.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-114">By convention a shadow primary key will be created for the owned type and it will be mapped to the same table as the owner by using table splitting.</span></span> <span data-ttu-id="b9e1f-115">Dies ermöglicht die Verwendung von eigenen Typen, ähnlich wie bei den in EF 6 verwendeten komplexen Typen:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-115">This allows to use owned types similarly to how complex types are used in EF6:</span></span>

``` csharp
modelBuilder.Entity<Order>().OwnsOne(p => p.OrderDetails, cb =>
    {
        cb.OwnsOne(c => c.BillingAddress);
        cb.OwnsOne(c => c.ShippingAddress);
    });

public class Order
{
    public int Id { get; set; }
    public OrderDetails OrderDetails { get; set; }
}

public class OrderDetails
{
    public StreetAddress BillingAddress { get; set; }
    public StreetAddress ShippingAddress { get; set; }
}

public class StreetAddress
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

### <a name="model-level-query-filters"></a><span data-ttu-id="b9e1f-116">Abfragefilter auf Modellebene</span><span class="sxs-lookup"><span data-stu-id="b9e1f-116">Model-level query filters</span></span>

<span data-ttu-id="b9e1f-117">EF Core 2.0 beinhaltet ein neues Feature, das als Abfragefilter auf Modellebene bezeichnet wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-117">EF Core 2.0 includes a new feature we call Model-level query filters.</span></span> <span data-ttu-id="b9e1f-118">Dieses Feature ermöglicht es, LINQ-Abfrageprädikate (einen booleschen Ausdruck, der typischerweise an den LINQ-Abfrageoperator „Where“ übergeben wird) in direktem Zusammenhang mit Entitätstypen im Metadatenmodell (normalerweise in OnModelCreating) zu definieren.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-118">This feature allows LINQ query predicates (a boolean expression typically passed to the LINQ Where query operator) to be defined directly on Entity Types in the metadata model (usually in OnModelCreating).</span></span> <span data-ttu-id="b9e1f-119">Solche Filter werden automatisch auf alle diese Entitätstypen betreffenden LINQ-Abfragen (z.B. indirekt referenzierte Entitätstypen) angewendet, beispielsweise durch include-Verweise oder direkte Verweise auf Navigationseigenschaften.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-119">Such filters are automatically applied to any LINQ queries involving those Entity Types, including Entity Types referenced indirectly, such as through the use of Include or direct navigation property references.</span></span> <span data-ttu-id="b9e1f-120">Zu den häufigsten Anwendungsfällen dieses Features zählen Folgende:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-120">Some common applications of this feature are:</span></span>

- <span data-ttu-id="b9e1f-121">Vorläufiges Löschen – Ein Entitätstyp definiert eine IsDeleted-Eigenschaft.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-121">Soft delete - An Entity Types defines an IsDeleted property.</span></span>
- <span data-ttu-id="b9e1f-122">Mehrinstanzenfähigkeit – Ein Entitätstyp definiert eine TenantId-Eigenschaft.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-122">Multi-tenancy - An Entity Type defines a TenantId property.</span></span>

<span data-ttu-id="b9e1f-123">Im Folgenden wird ein einfaches Beispiel vorgestellt, das das Feature für die zwei oben genannten Szenarien veranschaulicht:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-123">Here is a simple example demonstrating the feature for the two scenarios listed above:</span></span>

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    public int TenantId { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Post>().HasQueryFilter(
            p => !p.IsDeleted
            && p.TenantId == this.TenantId );
    }
}
```
<span data-ttu-id="b9e1f-124">Wir definieren einen Filter auf Modellebene, der Mehrinstanzenfähigkeit sowie das vorläufige Löschen für Instanzen des Entitätstyps ```Post``` implementiert.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-124">We define a model-level filter that implements multi-tenancy and soft-delete for instances of the ```Post``` Entity Type.</span></span> <span data-ttu-id="b9e1f-125">Beachten Sie die Verwendung einer DbContext-Instanzeigenschaft: ```TenantId```.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-125">Note the use of a DbContext instance level property: ```TenantId```.</span></span> <span data-ttu-id="b9e1f-126">Filter auf Modellebene verwenden den Wert aus der entsprechenden Kontextinstanz.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-126">Model-level filters will use the value from the correct context instance.</span></span> <span data-ttu-id="b9e1f-127">Damit ist die Instanz gemeint, die die Abfrage ausführt.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-127">I.e. the one that is executing the query.</span></span>

<span data-ttu-id="b9e1f-128">Filter können für einzelne LINQ-Abfragen mit dem IgnoreQueryFilters()-Operator deaktiviert werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-128">Filters may be disabled for individual LINQ queries using the IgnoreQueryFilters() operator.</span></span>

#### <a name="limitations"></a><span data-ttu-id="b9e1f-129">Einschränkungen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-129">Limitations</span></span>

- <span data-ttu-id="b9e1f-130">Navigationsverweise sind unzulässig.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-130">Navigation references are not allowed.</span></span> <span data-ttu-id="b9e1f-131">Dieses Feature kann basierend auf Feedback hinzugefügt werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-131">This feature may be added based on feedback.</span></span>
- <span data-ttu-id="b9e1f-132">Filter können nur im Stammentitätstyp einer Hierarchie definiert werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-132">Filters can only be defined on the root Entity Type of a hierarchy.</span></span>

### <a name="database-scalar-function-mapping"></a><span data-ttu-id="b9e1f-133">Zuordnung von Skalarfunktionen für Datenbanken</span><span class="sxs-lookup"><span data-stu-id="b9e1f-133">Database scalar function mapping</span></span>

<span data-ttu-id="b9e1f-134">Preview 2 enthält einen wichtigen Beitrag von [Paul Middleton](https://github.com/pmiddleton), die die Zuordnung von Skalarfunktionen für Datenbanken zu Methodenstubs ermöglicht, sodass sie in LINQ-Abfragen verwendet und in SQL übersetzt werden können.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-134">Preview 2 includes an important contribution from [Paul Middleton](https://github.com/pmiddleton) which enables mapping database scalar functions to method stubs so that they can be used in LINQ queries and translated to SQL.</span></span>

<span data-ttu-id="b9e1f-135">Im Folgenden wird kurz beschrieben, wie das Feature verwendet werden kann:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-135">Here is a brief description of how the feature can be used:</span></span>

<span data-ttu-id="b9e1f-136">Deklarieren Sie eine statische Methode für den Typ `DbContext`, und kommentieren Sie ihn mit `DbFunctionAttribute`:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-136">Declare a static method on your `DbContext` and annotate it with `DbFunctionAttribute`:</span></span>

``` csharp
public class BloggingContext : DbContext
{
    [DbFunction]
    public static int PostReadCount(int blogId)
    {
        throw new Exception();
    }
}
```

<span data-ttu-id="b9e1f-137">Derartige Methoden werden automatisch registriert.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-137">Methods like this are automatically registered.</span></span> <span data-ttu-id="b9e1f-138">Sobald eine Methode registriert wurde, können Sie sie beliebig in Ihren Abfragen verwenden:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-138">Once a method has been registered you can use it anywhere in your queries:</span></span>

``` csharp
var query =
    from p in context.Posts
    where BloggingContext.PostReadCount(p.Id) > 5
    select p;
```

<span data-ttu-id="b9e1f-139">Im Folgenden sollten Sie einige Aspekte berücksichtigen:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-139">A few things to note:</span></span>

- <span data-ttu-id="b9e1f-140">Der Name der Methode wird gemäß den Konventionen als Name einer Funktion (in diesem Fall einer benutzerdefinierten Funktion) bei der SQL-Generierung verwendet, Sie können den Namen und das Schema bei der Methodenregistrierung jedoch überschreiben.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-140">By convention the name of the method is used as the name of a function (in this case a user defined function) when generating the SQL, but you can override the name and schema during method registration</span></span>
- <span data-ttu-id="b9e1f-141">Derzeit werden nur Skalarfunktionen unterstützt.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-141">Currently only scalar functions are supported</span></span>
- <span data-ttu-id="b9e1f-142">Sie müssen die zugeordnete Funktion in der Datenbank erstellen. Bei EF Core-Migrationen werden diese beispielsweise nicht erstellt.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-142">You must create the mapped function in the database, e.g. EF Core migrations will not take care of creating it</span></span>

### <a name="self-contained-type-configuration-for-code-first"></a><span data-ttu-id="b9e1f-143">Eigenständige Typkonfiguration für den Code First</span><span class="sxs-lookup"><span data-stu-id="b9e1f-143">Self-contained type configuration for code first</span></span>

<span data-ttu-id="b9e1f-144">In EF 6 war es möglich, die Code First-Konfiguration eines bestimmten Entitätstyps durch Ableitung von *EntityTypeConfiguration* zu kapseln.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-144">In EF6 it was possible to encapsulate the code first configuration of a specific entity type by deriving from *EntityTypeConfiguration*.</span></span> <span data-ttu-id="b9e1f-145">In EF Core 2.0 wird dieses Muster wieder eingeführt:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-145">In EF Core 2.0 we are bringing this pattern back:</span></span>

``` csharp
class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
  public void Configure(EntityTypeBuilder<Customer> builder)
  {
     builder.HasKey(c => c.AlternateKey);
     builder.Property(c => c.Name).HasMaxLength(200);
   }
}

...
// OnModelCreating
builder.ApplyConfiguration(new CustomerConfiguration());
```

## <a name="high-performance"></a><span data-ttu-id="b9e1f-146">Hohe Leistung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-146">High Performance</span></span>

### <a name="dbcontext-pooling"></a><span data-ttu-id="b9e1f-147">DbContext-Pooling</span><span class="sxs-lookup"><span data-stu-id="b9e1f-147">DbContext pooling</span></span>

<span data-ttu-id="b9e1f-148">Das grundlegende Muster für die Verwendung von EF Core in einer ASP.NET Core-Anwendung besteht in der Regel darin, einen benutzerdefinierten DbContext-Typ im Abhängigkeitsinjektionssystem zu registrieren und später Instanzen dieses Typs über Konstruktorparameter in Controllern abzurufen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-148">The basic pattern for using EF Core in an ASP.NET Core application usually involves registering a custom DbContext type into the dependency injection system and later obtaining instances of that type through constructor parameters in controllers.</span></span> <span data-ttu-id="b9e1f-149">Das bedeutet, dass für jede Anforderung eine neue DbContext-Instanz erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-149">This means a new instance of the DbContext is created for each requests.</span></span>

<span data-ttu-id="b9e1f-150">In Version 2.0 stellen wir eine neue Möglichkeit vor, um benutzerdefinierte DbContext-Typen im Abhängigkeitsinjektionssystem zu registrieren, in dem transparent ein Pool von wiederverwendbaren DbContext-Instanzen eingeführt wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-150">In version 2.0 we are introducing a new way to register custom DbContext types in dependency injection which transparently introduces a pool of reusable DbContext instances.</span></span> <span data-ttu-id="b9e1f-151">Um DbContext-Pooling zu verwenden, verwenden Sie bei der Dienstregistrierung ```AddDbContextPool``` anstelle von ```AddDbContext```:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-151">To use DbContext pooling, use the ```AddDbContextPool``` instead of ```AddDbContext``` during service registration:</span></span>

``` csharp
services.AddDbContextPool<BloggingContext>(
    options => options.UseSqlServer(connectionString));
```

<span data-ttu-id="b9e1f-152">Wird diese Methode verwendet, wird beim Anfordern einer DbContext-Instanz durch einen Controller zunächst geprüft, ob eine Instanz im Pool vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-152">If this method is used, at the time a DbContext instance is requested by a controller we will first check if there is an instance available in the pool.</span></span> <span data-ttu-id="b9e1f-153">Sobald die Anforderungsverarbeitung abgeschlossen ist, werden alle Zustände der Instanz zurückgesetzt, und die Instanz selbst wird an den Pool zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-153">Once the request processing finalizes, any state on the instance is reset and the instance is itself returned to the pool.</span></span>

<span data-ttu-id="b9e1f-154">Dies ist konzeptionell vergleichbar mit der Funktionsweise des Verbindungspoolings bei ADO.NET-Anbietern und bietet den Vorteil, dass ein Teil der Initialisierungskosten der DbContext-Instanz eingespart werden kann.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-154">This is conceptually similar to how connection pooling operates in ADO.NET providers and has the advantage of saving some of the cost of initialization of DbContext instance.</span></span>

### <a name="limitations"></a><span data-ttu-id="b9e1f-155">Einschränkungen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-155">Limitations</span></span>

<span data-ttu-id="b9e1f-156">Bei der neuen Methode werden einige Einschränkungen zu den Vorgängen in der ```OnConfiguring()```-Methode des DbContext-Typs eingeführt.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-156">The new method introduces a few limitations on what can be done in the ```OnConfiguring()``` method of the DbContext.</span></span>

> [!WARNING]  
> <span data-ttu-id="b9e1f-157">Verwenden Sie kein DbContext-Pooling, wenn Sie Ihren eigenen Zustand (z.B. private Felder) in Ihrer abgeleiteten DbContext-Klasse verwalten, da dieser nicht für alle Anforderungen genutzt werden sollte.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-157">Avoid using DbContext Pooling if you maintain your own state (e.g. private fields) in your derived DbContext class that should not be shared across requests.</span></span> <span data-ttu-id="b9e1f-158">EF Core setzt nur den Zustand zurück, der bekannt ist, bevor eine DbContext-Instanz zum Pool hinzugefügt wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-158">EF Core will only reset the state that is aware of before adding a DbContext instance to the pool.</span></span>

### <a name="explicitly-compiled-queries"></a><span data-ttu-id="b9e1f-159">Explizit kompilierte Abfragen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-159">Explicitly compiled queries</span></span>

<span data-ttu-id="b9e1f-160">Dies ist das zweite optionale Leistungsfeature, das vor allem für die Verwendung in Szenarien mit umfangreicher Skalierung optimiert wurde.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-160">This is the second opt-in performance features designed to offer benefits in high-scale scenarios.</span></span>

<span data-ttu-id="b9e1f-161">Manuell oder explizit kompilierte Abfrage-APIs waren in vorherigen EF-Versionen und auch in LINQ to SQL verfügbar. So können Anwendungen die Übersetzung von Abfragen zwischenspeichern, sodass sie nur einmal berechnet und mehrmals ausgeführt werden können.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-161">Manual or explicitly compiled query APIs have been available in previous versions of EF and also in LINQ to SQL to allow applications to cache the translation of queries so that they can be computed only once and executed many times.</span></span>

<span data-ttu-id="b9e1f-162">Obwohl EF Core generell auch automatisch Abfragen basierend auf einer verschlüsselten Darstellung von Abfrageausdrücken kompilieren und zwischenspeichern kann, kann mithilfe dieses Mechanismus ein geringfügiger Leistungsgewinn erzielt werden, indem die Berechnung des Hashwerts sowie die Cachesuche umgangen werden. Dies ermöglicht die Verwendung einer bereits kompilierten Abfrage in der Anwendung durch Aufrufen eines Delegaten.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-162">Although in general EF Core can automatically compile and cache queries based on a hashed representation of the query expressions, this mechanism can be used to obtain a small performance gain by bypassing the computation of the hash and the cache lookup, allowing the application to use an already compiled query through the invocation of a delegate.</span></span>

``` csharp
// Create an explicitly compiled query
private static Func<CustomerContext, int, Customer> _customerById =
    EF.CompileQuery((CustomerContext db, int id) =>
        db.Customers
            .Include(c => c.Address)
            .Single(c => c.Id == id));

// Use the compiled query by invoking it
using (var db = new CustomerContext())
{
   var customer = _customerById(db, 147);
}
```

## <a name="change-tracking"></a><span data-ttu-id="b9e1f-163">Änderungsnachverfolgung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-163">Change Tracking</span></span>

### <a name="attach-can-track-a-graph-of-new-and-existing-entities"></a><span data-ttu-id="b9e1f-164">Nachverfolgen eines Graphen von neuen und vorhandenen Entitäten durch Anfügen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-164">Attach can track a graph of new and existing entities</span></span>

<span data-ttu-id="b9e1f-165">EF Core unterstützt die automatische Generierung von Schlüsselwerten durch eine Vielzahl von Mechanismen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-165">EF Core supports automatic generation of key values through a variety of mechanisms.</span></span> <span data-ttu-id="b9e1f-166">Bei Verwendung dieses Features wird ein Wert generiert, wenn die Schlüsseleigenschaft der CLR-Standardwert ist (normalerweise Null).</span><span class="sxs-lookup"><span data-stu-id="b9e1f-166">When using this feature, a value is generated if the key property is the CLR default--usually zero or null.</span></span> <span data-ttu-id="b9e1f-167">Das bedeutet, dass ein Graph von Entitäten an `DbContext.Attach` oder `DbSet.Attach` übergeben werden kann und EF Core Entitäten markiert, die bereits auf `Unchanged` festgelegt wurden, während Entitäten ohne Schlüsselsatz mit `Added` markiert werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-167">This means that a graph of entities can be passed to `DbContext.Attach` or `DbSet.Attach` and EF Core will mark those entities that have a key already set as `Unchanged` while those entities that do not have a key set will be marked as `Added`.</span></span> <span data-ttu-id="b9e1f-168">Dies vereinfacht das Anfügen eines Graphen mit gemischten neuen und bestehenden Entitäten, wenn generierte Schlüssel verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-168">This makes it easy to attach a graph of mixed new and existing entities when using generated keys.</span></span> <span data-ttu-id="b9e1f-169">`DbContext.Update` und `DbSet.Update` funktionieren auf die gleiche Weise, außer dass Entitäten mit dem Schlüsselsatz `Modified` statt `Unchanged` markiert sind.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-169">`DbContext.Update` and `DbSet.Update` work in the same way, except that entities with a key set are marked as `Modified` instead of `Unchanged`.</span></span>

## <a name="query"></a><span data-ttu-id="b9e1f-170">Abfrage</span><span class="sxs-lookup"><span data-stu-id="b9e1f-170">Query</span></span>

### <a name="improved-linq-translation"></a><span data-ttu-id="b9e1f-171">Verbesserte LINQ-Übersetzung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-171">Improved LINQ Translation</span></span>

<span data-ttu-id="b9e1f-172">Dies ermöglicht die erfolgreiche Ausführung von weiteren Abfragen, wobei ein größerer Teil der Logik in der Datenbank ausgewertet wird (statt im Speicher) und weniger Daten unnötig aus der Datenbank abgerufen werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-172">Enables more queries to successfully execute, with more logic being evaluated in the database (rather than in-memory) and less data unnecessarily being retrieved from the database.</span></span>

### <a name="groupjoin-improvements"></a><span data-ttu-id="b9e1f-173">GroupJoin-Verbesserungen</span><span class="sxs-lookup"><span data-stu-id="b9e1f-173">GroupJoin improvements</span></span>

<span data-ttu-id="b9e1f-174">Durch diese Änderungen wird das SQL-Skript verbessert, das für Gruppenverknüpfungen generiert wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-174">This work improves the SQL that is generated for group joins.</span></span> <span data-ttu-id="b9e1f-175">Gruppenverknüpfungen sind oftmals das Ergebnis von Unterabfragen zu optionalen Navigationseigenschaften.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-175">Group joins are most often a result of sub-queries on optional navigation properties.</span></span>

### <a name="string-interpolation-in-fromsql-and-executesqlcommand"></a><span data-ttu-id="b9e1f-176">Zeichenfolgeninterpolation in FromSql und ExecuteSqlCommand</span><span class="sxs-lookup"><span data-stu-id="b9e1f-176">String interpolation in FromSql and ExecuteSqlCommand</span></span>

<span data-ttu-id="b9e1f-177">Mit C# 6 wurde Zeichenfolgeninterpolation eingeführt, ein Feature, mit dem C#-Ausdrücke direkt in Zeichenfolgenliterale eingebettet werden können, was eine komfortable Möglichkeit zum Erstellen von Zeichenfolgen zur Runtime bietet.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-177">C# 6 introduced String Interpolation, a feature that allows C# expressions to be directly embedded in string literals, providing a nice way of building strings at runtime.</span></span> <span data-ttu-id="b9e1f-178">In EF Core 2.0 haben wir eine spezielle Unterstützung für interpolierte Zeichenfolgen zu unseren zwei primären APIs hinzugefügt, die SQL-Rohzeichenfolgen akzeptieren: ```FromSql``` und ```ExecuteSqlCommand```.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-178">In EF Core 2.0 we added special support for interpolated strings to our two primary APIs that accept raw SQL strings: ```FromSql``` and ```ExecuteSqlCommand```.</span></span> <span data-ttu-id="b9e1f-179">Diese neue Unterstützung ermöglicht die sichere Verwendung von C#-Zeichenfolgeninterpolation.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-179">This new support allows C# string interpolation to be used in a 'safe' manner.</span></span> <span data-ttu-id="b9e1f-180">Denn es wird vor häufigen Fehlern durch Angriffe durch Einschleusung von SQL-Befehlen geschützt, die auftreten können, wenn SQL dynamisch zur Runtime erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-180">I.e. in a way that protects against common SQL injection mistakes that can occur when dynamically constructing SQL at runtime.</span></span>

<span data-ttu-id="b9e1f-181">Im Folgenden ein Beispiel:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-181">Here is an example:</span></span>

``` csharp
var city = "London";
var contactTitle = "Sales Representative";

using (var context = CreateContext())
{
    context.Set<Customer>()
        .FromSql($@"
            SELECT *
            FROM ""Customers""
            WHERE ""City"" = {city} AND
                ""ContactTitle"" = {contactTitle}")
            .ToArray();
  }
```

<span data-ttu-id="b9e1f-182">In diesem Beispiel sind zwei Variablen in die SQL-Formatzeichenfolge eingebettet.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-182">In this example, there are two variables embedded in the SQL format string.</span></span> <span data-ttu-id="b9e1f-183">EF Core erzeugt die folgende SQL-Syntax:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-183">EF Core will produce the following SQL:</span></span>

```sql
@p0='London' (Size = 4000)
@p1='Sales Representative' (Size = 4000)

SELECT *
FROM ""Customers""
WHERE ""City"" = @p0
    AND ""ContactTitle"" = @p1
```

### <a name="effunctionslike"></a><span data-ttu-id="b9e1f-184">EF.Functions.Like()</span><span class="sxs-lookup"><span data-stu-id="b9e1f-184">EF.Functions.Like()</span></span>

<span data-ttu-id="b9e1f-185">Wir haben die EF.Functions-Eigenschaft hinzugefügt, mit denen EF Core oder Anbieter Methoden definieren können, die Datenbankfunktionen oder -operatoren zuordnen, sodass diese in LINQ-Abfragen aufgerufen werden können.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-185">We have added the EF.Functions property which can be used by EF Core or providers to define methods that map to database functions or operators so that those can be invoked in LINQ queries.</span></span> <span data-ttu-id="b9e1f-186">Das erste Beispiel einer solchen Methode ist „Like()“:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-186">The first example of such a method is Like():</span></span>

``` csharp
var aCustomers =
    from c in context.Customers
    where EF.Functions.Like(c.Name, "a%");
    select c;
```

<span data-ttu-id="b9e1f-187">Beachten Sie, dass „Like()“ über eine In-Memory-Implementierung verfügt, die für die Arbeit mit einer In-Memory Database oder bei der Auswertung des Prädikats auf der Clientseite auftreten muss.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-187">Note that Like() comes with an in-memory implementation, which can be handy when working against an in-memory database or when evaluation of the predicate needs to occur con the client side.</span></span>

## <a name="database-management"></a><span data-ttu-id="b9e1f-188">Datenbankverwaltung</span><span class="sxs-lookup"><span data-stu-id="b9e1f-188">Database management</span></span>

### <a name="pluralization-hook-for-dbcontext-scaffolding"></a><span data-ttu-id="b9e1f-189">Pluralisierungshook für den DbContext-Gerüstbau</span><span class="sxs-lookup"><span data-stu-id="b9e1f-189">Pluralization hook for DbContext Scaffolding</span></span>

<span data-ttu-id="b9e1f-190">In EF Core 2.0 wird ein neuer *IPluralizer*-Dienst eingeführt, mit dem Entitätstypnamen in den Singular und DbSet-Namen in den Plural gesetzt werden.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-190">EF Core 2.0 introduces a new *IPluralizer* service that is used to singularize entity type names and pluralize DbSet names.</span></span> <span data-ttu-id="b9e1f-191">Die Standardimplementierung ist eine No-Op-Bedingung. Somit ist dies lediglich ein Hook, mit dem Benutzer mühelos eigene Pluralisierer verbinden können.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-191">The default implementation is a no-op, so this is just a hook where folks can easily plug in their own pluralizer.</span></span>

<span data-ttu-id="b9e1f-192">Im Folgenden wird gezeigt, wie es für einen Entwickler aussieht, wenn er seinen eigenen Pluralisierer als Hook integriert:</span><span class="sxs-lookup"><span data-stu-id="b9e1f-192">Here is what it looks like for a developer to hook in their own pluralizer:</span></span>

``` csharp
public class MyDesignTimeServices : IDesignTimeServices
{
    public void ConfigureDesignTimeServices(IServiceCollection services)
    {
        services.AddSingleton<IPluralizer, MyPluralizer>();
    }
}

public class MyPluralizer : IPluralizer
{
    public string Pluralize(string name)
    {
        return Inflector.Inflector.Pluralize(name) ?? name;
    }

    public string Singularize(string name)
    {
        return Inflector.Inflector.Singularize(name) ?? name;
    }
}
```

## <a name="others"></a><span data-ttu-id="b9e1f-193">Sonstige</span><span class="sxs-lookup"><span data-stu-id="b9e1f-193">Others</span></span>

### <a name="move-adonet-sqlite-provider-to-sqlitepclraw"></a><span data-ttu-id="b9e1f-194">Migrieren des ADO.NET SQLite-Anbieters nach SQLitePCL.raw</span><span class="sxs-lookup"><span data-stu-id="b9e1f-194">Move ADO.NET SQLite provider to SQLitePCL.raw</span></span>
<span data-ttu-id="b9e1f-195">Dies bietet uns eine zuverlässigere Lösung in „Microsoft.Data.Sqlite“ für die Verteilung von nativen SQLite-Binärdateien auf verschiedene Plattformen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-195">This gives us a more robust solution in Microsoft.Data.Sqlite for distributing native SQLite binaries on different platforms.</span></span>

### <a name="only-one-provider-per-model"></a><span data-ttu-id="b9e1f-196">Verwenden von nur einem Anbieter pro Modell</span><span class="sxs-lookup"><span data-stu-id="b9e1f-196">Only one provider per model</span></span>
<span data-ttu-id="b9e1f-197">Dies erweitert erheblich die Möglichkeiten der Anbieter bezüglich der Interaktion mit dem Modell und vereinfacht die Verwendung von Konventionen, Anmerkungen und Fluent-APIs mit verschiedenen Anbietern.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-197">Significantly augments how providers can interact with the model and simplifies how conventions, annotations and fluent APIs work with different providers.</span></span>

<span data-ttu-id="b9e1f-198">EF Core 2.0 erstellt nun eine andere [IModel](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IModel.cs)-Klasse für die verschiedenen verwendeten Anbieter.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-198">EF Core 2.0 will now build a different [IModel](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/Metadata/IModel.cs) for each different provider being used.</span></span> <span data-ttu-id="b9e1f-199">Dies geschieht in der Regel auf eine für die Anwendung transparente Weise.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-199">This is usually transparent to the application.</span></span> <span data-ttu-id="b9e1f-200">Dies hat eine Vereinfachung der untergeordneten Metadaten-APIs ermöglicht, sodass Zugriffe auf *gemeinsame relationale Metadatenkonzepte* immer durch Aufrufen von `.Relational` statt `.SqlServer`, `.Sqlite` usw. erfolgen.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-200">This has facilitated a simplification of lower-level metadata APIs such that any access to *common relational metadata concepts* is always made through a call to `.Relational` instead of `.SqlServer`, `.Sqlite`, etc.</span></span>

### <a name="consolidated-logging-and-diagnostics"></a><span data-ttu-id="b9e1f-201">Konsolidierte Protokollierung und Diagnose</span><span class="sxs-lookup"><span data-stu-id="b9e1f-201">Consolidated Logging and Diagnostics</span></span>

<span data-ttu-id="b9e1f-202">Die Mechanismen zur Protokollierung (basierend auf ILogger) und Diagnose (basierend auf DiagnosticSource) nutzen nun weitere Codes gemeinsam.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-202">Logging (based on ILogger) and Diagnostics (based on DiagnosticSource) mechanisms now share more code.</span></span>

<span data-ttu-id="b9e1f-203">Die Ereignis-IDs für an eine ILogger-Schnittstelle gesendete Nachrichten haben sich in 2.0 geändert.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-203">The event IDs for messages sent to an ILogger have changed in 2.0.</span></span> <span data-ttu-id="b9e1f-204">Die Ereignis-IDs sind nun für alle EF Core-Codes eindeutig.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-204">The event IDs are now unique across EF Core code.</span></span> <span data-ttu-id="b9e1f-205">Diese Nachrichten entsprechen nun auch dem Standardmuster für die strukturierte Protokollierung, das z.B. von MVC eingesetzt wird.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-205">These messages now also follow the standard pattern for structured logging used by, for example, MVC.</span></span>

<span data-ttu-id="b9e1f-206">Die Protokollierungskategorien haben sich ebenfalls geändert.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-206">Logger categories have also changed.</span></span> <span data-ttu-id="b9e1f-207">Nun gibt es eine bekannte Gruppe von Kategorien, auf die über [DbLoggerCategory](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/DbLoggerCategory.cs) zugegriffen werden kann.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-207">There is now a well-known set of categories accessed through [DbLoggerCategory](https://github.com/aspnet/EntityFramework/blob/dev/src/EFCore/DbLoggerCategory.cs).</span></span>

<span data-ttu-id="b9e1f-208">DiagnosticSource-Ereignisse verwenden nun dieselbe Ereignis-ID-Namen wie die der entsprechenden `ILogger`-Nachrichten.</span><span class="sxs-lookup"><span data-stu-id="b9e1f-208">DiagnosticSource events now use the same event ID names as the corresponding `ILogger` messages.</span></span>