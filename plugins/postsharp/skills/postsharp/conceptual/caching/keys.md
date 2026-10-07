---
uid: caching-keys
redirect_from:
  - caching/caching-keys
title: "Customizing Cache Keys"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on customizing cache keys in PostSharp, including default key generation logic, overriding the ToString method, implementing a custom cache key formatter, excluding parameters from a cache key, changing the maximal length of a cache key, and implementing a custom cache key builder."
---
# Customizing Cache Keys

Each value returned by a cached method is indexed by a cache key. By default, the cache key is composed of the full method name, the parameter types, and the parameter values. By default, parameters are formatted using the `ToString` method. PostSharp allows you to completely customize how the cache key is generated. 


## Default cache key generation logic

By default, the key is composed of the following elements of the method call:

* the full name of the declaring type (including generic parameters, if any),

* the method name,

* the method generic parameters, if any,

* the formatted caller object (unless the method is static),

* a comma-separated list of all method arguments including the full type of the parameter and the formatted parameter value,

* if the backend supports it, a global prefix that allows using the same caching server with several applications (see, for example, <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix>). 

> [!WARNING]
> By default, the cache key of a value is built using the `ToString` method, but the default implementation of the `ToString` method does not return a unique string for custom types. You must override the `ToString` method or implement a formatter for all types of parameters used in a cached method. 


## Overriding the ToString method

By default, the cache key of a value is built using the <xref:System.Object.ToString> method. You can override the <xref:System.Object.ToString> method so that it returns a distinct value for each distinct instance of the type. 


## Implementing a custom cache key formatter

Since the <xref:System.Object.ToString> method is used in different contexts than just caching, using it as the cache key might be inappropriate in your case. In this situation, you can implement the <xref:PostSharp.Patterns.Formatters.IFormattable> interface or the <xref:PostSharp.Patterns.Formatters.Formatter`1> class. 

See <xref:custom-formatter> for details. 


## Excluding parameters from a cache key


### Using [NotCacheKey]

To exclude a method parameter from the cache key, add the <xref:PostSharp.Patterns.Caching.NotCacheKeyAttribute> custom attribute to the parameter. 


### Using [IgnoreThisParameter]

To exclude the value of the `this` parameter of an instance method from the cache key, set the <xref:PostSharp.Patterns.Caching.CacheAttribute.IgnoreThisParameter> parameter of the aspect to `true`. 


### Example

In this example, the `this` and `callId` parameters of the `GetNumber` method are not part of the cache key. 

```csharp
using System;
using System.Threading;
using PostSharp.Patterns.Caching;
using PostSharp.Patterns.Caching.Backends;

namespace PostSharp.Samples.Caching.ExcludingParameters
{
    class Database
    {
        [Cache( IgnoreThisParameter = true )]
        public int GetNumber( int id, [NotCacheKey] int callId )
        {
            Console.WriteLine( $">> Retrieving {id} from the database, call ID {callId}..." );
            Thread.Sleep( 1000 );
            return id;
        }
    }

    class Program
    {
        static void Main( string[] args )
        {
            CachingServices.DefaultBackend = new MemoryCachingBackend();

            int callId = 0;

            Database db1 = new Database();
            Database db2 = new Database();

            Console.WriteLine( "Retrieving value of 1 for the 1st time from DB 1 should hit the database." );
            Console.WriteLine( "Retrieved: " + db1.GetNumber( 1, callId++ ) );

            Console.WriteLine( "Retrieving value of 1 for the 2nd time from DB 1 passing different call ID should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db1.GetNumber( 1, callId++ ) );

            Console.WriteLine( "Retrieving value of 1 for the 1st time from DB 2 should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db2.GetNumber( 1, callId++ ) );
        }
    }
}
```

The output of this sample is:

```text
Retrieving value of 1 for the 1st time from DB 1 should hit the database.
>> Retrieving 1 from the database, call ID 0...
Retrieved: 1
Retrieving value of 1 for the 2nd time from DB 1 passing different call ID should NOT hit the database.
Retrieved: 1
Retrieving value of 1 for the 1st time from DB 2 should NOT hit the database.
Retrieved: 1
```


## Changing the maximal length of a cache key

The maximal length of a cache key is 2048 characters by default.

To change the maximal length of a cache key, create a new instance of the <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder> class, passing the new value of the maximal length to the constructor, and assign the new instance to the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultKeyBuilder> property. 

> [!WARNING]
> If you need large cache keys, we suggest you also hash the cache key before submitting it to the caching backend. To hash the cache key, pass a <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyHashingAlgorithm> value other than `None` to the constructor of the <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder> class. You can also implement a custom cache key builder (see below).

The following example sets the maximal length of a cache key to 4096 characters:

```csharp
CachingServices.DefaultKeyBuilder = new CacheKeyBuilder(4096);
```


## Implementing a custom cache key builder

All the options described above change how a parameter value is formatted. To customize the other parts of the cache key, you can override the methods of the <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder> class. 


### Overriding the cache key building logic

1. Create a new class and derive it from the <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder> class. 


2. Override the virtual methods as needed.

    To completely override the key building process, override the <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.BuildMethodKey(System.Reflection.MethodInfo,System.Collections.Generic.IList{System.Object},System.Object)> and/or <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.BuildDependencyKey(System.Object)> methods. The default implementation of these two methods uses <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.AppendType(PostSharp.Patterns.Formatters.UnsafeStringBuilder,System.Type)>, <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.AppendMethod(PostSharp.Patterns.Formatters.UnsafeStringBuilder,System.Reflection.MethodInfo)>, <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.AppendArgument(PostSharp.Patterns.Formatters.UnsafeStringBuilder,System.Type,System.Object)> and <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder.AppendObject(PostSharp.Patterns.Formatters.UnsafeStringBuilder,System.Object)> helper methods. If you need to override how a type, a method, an argument or an object is formatted, you can override just some of these methods. 


3. Assign an instance of your new cache key builder to the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultKeyBuilder> property. 





