# The Omen Den Shared Services

## This library contains some simple implementations of the services defined in the Interface library
- _[The Omen Den Shared Interface Library](https://github.com/theomenden/TheOmenDen.Shared.Interfaces)_
- Note: _These libraries have dependencies on [The Omen Den Shared Library](https://github.com/theomenden/TheOmenDen.Shared)_
  
## The goal with this library is to provide simple abstract services for calling external APIs and provide `Scrutor` based registration for any sub-services that inherit from them.

### Of course, while we have the implementations defined in this library, we also have the dependency injection defined. The way it _should_ work is by inheriting from either the `ApiServiceBase` or the `ApiStreamServiceBase` abstract classes, and then calling the `IserviceCollection` extension `AddTheOmenDenHttpServices` to allow `scrutor` to register them into the dependency injection container.
- This was done in an attempt to streamline and smooth out my poor memory when it comes to registering services.
- Note the use of `Polly` in this library to define automatic retry policies for `HttpClients`.
- We also have a class - HttpClientConfiguration for your convinience when it comes to registering named clients via appsettings, or some other form of `Action` "passing"
  
## As usual, here are the rationales for the services
### 1. `ApiServiceBase<T>`
- Provides CRUD functionality based off of HTTP Methods
- Allows for injection of a named `IHttpClientFactory` and implements the `IOptions<T>` pattern. 
- The idea here is with the `IHttpClientFactory`, using a "fresh" `HttpClient` per request thus saving on costs, and allowing for better life cycle management.
- All methods return a `Task<ApiResponse<TResponse>>` of some sort, the reasoning for this is two fold
    1. To allow for better handling when a request fails to process for any reason
    2. To allow for better tracability and response deserialization.
- There are two methods for stream deserialization to help smooth out issues with response contents, and to allow the deserialization into a string for error handling.
### 2. `ApiStreamServiceBase<T>`
 - Provides ReadOnly streaming functionality from an external API
- As with the previous implementation we return resposnes based off of the `ApiResponse` object.
- This decision is made more evident when we call `StreamApiResultsAsync` since we can now handle each request failure within an `await foreach` loop on the consumer side. 
- The main justification for this approach is for extensibility and better error reporting. 
- We do our best to maintain our available resources by limiting the exceptions thrown, preferring to capture them, and returning objects that function similarly.
- As with the previous implementation these methods are all defined as `virtual`, allowing for you to override them on a case by case basis.

#ToDos
1. Allow better Dependency injection
   - Add support for various Injection frameworks such as AutoFac and SimpleInjector
2. Better memory usage
   - As with everything, we try to optimize what we do as much as possible - here we just need to make sure the resources we use are properly disposed.
3. Provide source generators and Json generators for the responses that we deserialize.
   - This will allow us to autogenerate services that are efficient and can be automatically registered from user supplied code.
   - On the JSON side of things, this will allow for more efficient serialization and deserialization of requests and responses. 
