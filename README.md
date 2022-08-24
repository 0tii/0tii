# 📝 Mini Blog

Just starting a collection of small but useful ideas / tips / tricks / patterns to combat modern problems elegantly. This 'blog' might grow over time, or stay small, it really depends.

## 🏭 Async Factory [C#]

During development of a hybrid library to interact with the browser game *Gladiatus* I found myself having to asynchronously populate class instances at instantiation time. Since constructors return an instance of their type, by nature they can not return a `Task` and therefore can not be asynchronous. Hence `await` in class constructors is not possible and class members that need to be initialized with fetched data can not be populated from the constructor.

I spent a few hours thinking about this and came up with an elegant solution - I can abstract the factory pattern to allow for asynchronous population!

We start by making our `constructor` `private` and creating a `public` `static` `async` method that we can call to create an instance for us. It could look something like this:

```C#
public sealed class Container
{
    string _localData;

    private Container(string localData)
    {
        _localData = localData;
    }

    public static async Task<Container> CreateInstanceAsync(string localData)
    {
        Container container = new(localData);
        return container;
    }
}
```

This does not take advantage of asynchronicity yet, so far we only got a bare-bones factory that returns an instance of `Container` (it basically just statically wraps the constructor).

Now we just add a `private` `async` instance method to take care of our initialization and asynchronously `await` that on the instance our `static` method `CreateInstanceAsync` creates before returning:

```C#
public sealed class Container
{
    string _localData; //data that is availabe at runtime
    string? _remoteData; //data that has to be fetched

    private Container(string localData)
    {
        //local data can be passed to constructor
        _localData = localData;
    }

    private async Task<Container> InitializeAsync()
    {
        //here we can populate any data asynchronously
        _remoteData = await SomeClass.FetchRemoteDataAsync();
        return this;
    }

    public static async Task<Container> CreateInstanceAsync(string localData)
    {
        Container container = new(localData);
        //we can now await the initialization method on the instance
        return await container.InitializeAsync();
    }
}
```

And voila! We can now create a fully populated class instance simply by awaiting the factory method `CreateInstanceAsync` without having to worry about synchronicity issues or forgetting to call a dreaded `Initialize()` on a `new` instance.
____________________


<img align="left" width="450" height="150" src="https://github-readme-stats.vercel.app/api?username=0tii&show_icons=true&theme=tokyonight" />

<img align="right" width="450" height="150" src="https://github-readme-stats.vercel.app/api/top-langs/?username=0tii&layout=compact" />

