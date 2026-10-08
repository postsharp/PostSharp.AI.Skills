---
uid: caching-invalidation
redirect_from:
  - caching/caching-invalidation
title: "Removing Items From the Cache"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to remove items from the cache in PostSharp, using either direct invalidation, the InvalidateCache aspect, or by invalidating cache items imperatively."
---
# Removing Items From the Cache

When a method updates an entity, it must also remove any cache item that depends on this entity. One way to achieve this goal is to require the update methods to know precisely which cached methods are dependent on the entity that has been modified, and to remove these methods (with proper arguments) from the cache. We call this scenario *direct* cache invalidation. 

The benefit of direct invalidation is that it does not require a lot of resources on the caching backend. However, this approach has a big disadvantage: it exhibits an imperfect separation of concerns. Update methods need to have a precise knowledge of cached methods (typically read methods), so update methods need to be modified whenever a read method is added. For an approach with better abstraction, see <xref:caching-dependencies>. 


## Invalidating cache items using the [InvalidateCache] aspect

You can add the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> aspect to a method (called the *invalidating method*) to cause any call to this method to remove from the cache the value of one or more other methods. Parameters of both methods are matched by name and type. If any parameter of the cached method cannot be matched with a parameter of the invalidating method, you will get a build error (unless the parameter has the <xref:PostSharp.Patterns.Caching.NotCacheKeyAttribute> custom attribute). The order of parameters is not considered. 

> [!NOTE]
> By default, the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> aspect looks for the cached method in the current type. You can specify a different type using the alternative constructor of the custom attribute. When you invalidate a non-static method (unless the instance has been excluded from the cache key by setting the <xref:PostSharp.Patterns.Caching.CacheAttribute.IgnoreThisParameter> property to `true`), you can do it only from a non-static method of the same type or of a derived type.
>
> If there are more invalidated methods of the same name for one invalidating method, a build error is emitted. To enable invalidation of all the matching overloads by the one invalidating method, set the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute.AllowMultipleOverloads> property to `true`. 


### Example

In this example, the `Update` method invalidates the cached return value of the `GetValue` method. 

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using PostSharp.Patterns.Caching;
using PostSharp.Patterns.Caching.Backends;

namespace PostSharp.Samples.Caching.Invalidation
{
    class Database
    {
        private Dictionary<int, string> data = new Dictionary<int, string>();

        [Cache]
        public string GetValue( int id )
        {
            Console.WriteLine( $">> Retrieving {id} from the database..." );
            Thread.Sleep( 1000 );
            return this.data[id];
        }

        [InvalidateCache( nameof(GetValue) )]
        public void Update( int id, string value )
        {
            this.data[id] = value;
        }
    }

    class Program
    {
        static void Main( string[] args )
        {
            CachingServices.DefaultBackend = new MemoryCachingBackend();

            Database db = new Database();

            db.Update( 1, "first" );

            Console.WriteLine( "Retrieving value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            Console.WriteLine( "Retrieving value of 1 for the 2nd time should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            db.Update( 1, "second" );

            Console.WriteLine( "Retrieving updated value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );
        }
    }
}
```

The output of this sample is:

```text
Retrieving value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: first
Retrieving value of 1 for the 2nd time should NOT hit the database.
Retrieved: first
Retrieving updated value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: second
```


## Invalidating cache items imperatively

Instead of annotating invalidating methods with a custom attribute, you can call one of the overloads of the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation.Invalidate``1(System.Func{``0})> method. 


### Example

In this example, the cached return value of the `GetValue` method is invalidated by calling one of the overloads of the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation.Invalidate``1(System.Func{``0})> method. 

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using PostSharp.Patterns.Caching;
using PostSharp.Patterns.Caching.Backends;

namespace PostSharp.Samples.Caching.ImperativeInvalidation
{
    class Database
    {
        private Dictionary<int, string> data = new Dictionary<int, string>();

        [Cache]
        public string GetValue( int id )
        {
            Console.WriteLine( $">> Retrieving {id} from the database..." );
            Thread.Sleep( 1000 );
            return this.data[id];
        }

        public void Update( int id, string value )
        {
            this.data[id] = value;
        }
    }

    class Program
    {
        static void Main( string[] args )
        {
            CachingServices.DefaultBackend = new MemoryCachingBackend();

            Database db = new Database();

            db.Update( 1, "first" );

            Console.WriteLine( "Retrieving value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            Console.WriteLine( "Retrieving value of 1 for the 2nd time should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            db.Update( 1, "second" );
            CachingServices.Invalidation.Invalidate( db.GetValue, 1 );

            Console.WriteLine( "Retrieving updated value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );
        }
    }
}
```

The output of this sample is:

```text
Retrieving value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: first
Retrieving value of 1 for the 2nd time should NOT hit the database.
Retrieved: first
Retrieving updated value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: second
```


