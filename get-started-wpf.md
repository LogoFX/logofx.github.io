---
layout: page
title: Getting started
subtitle: Start lean
---

So you decided to create a new application that will look stunning and make life easier
for the users. Good for you. We're here to help ;)

You can always download the skeleton from [here](https://github.com/LogoFX/Samples.GetStarted/archive/v1.0.0.zip)

When you extract the skeleton you should see the following solution:
![alt text](../assets/skeleton-initial-structure.png)

### Launcher 

The entry point for your app is the `Launcher` project. It contains the bootstrapper 
with the minimum configuration logic and allows us to specify the app's root object:

```csharp
public sealed class AppBootstrapper : BootstrapperContainerBase<ExtendedSimpleContainerAdapter>
        .WithRootObject<ShellViewModel>
    {
        public AppBootstrapper()
            : base(new ExtendedSimpleContainerAdapter())
        {
        }              
    }
```

This bootstrapper extends the existing bootstrapper of the excellent framework [Caliburn.Micro](https://caliburnmicro.com).
We omit here its implementation for brevity but you're more than welcome to get familiar with this amazing tool.

The `ExtendedSimpleContainerAdapter` allows us to specify the kind of inversion-of-control 
container we want to use in our app for dependency injection. If you don't use dependency injection at all
(though you should ;) or already familiar with another container, don't worry. You can always replace the container here
and all non-container specific logic won't change at all!
The `ShellViewModel` is the root view model of the application we're about to write. Here we use the View-Model first approach 
of the MVVM UI pattern and therefore we have to specify the first view-model to be displayed.

### Presentation

The view and view-model parts of the app reside in the **Presentation** solution folder. We can see
two folders in the skeleton which host views and view-models respectively. We can also see the naming convention here:
*Feature*ViewModel <-> *Feature*View. This allows the underlying framework to locate the views without any additional configuration!
The root view model of any app is typically called `ShellViewModel` and serves as the container for any future components.

### Model

The main philosophy behind the framework is that the **Model** layer is the central part of any client app. It serves as a single source of truth
and allows any part of the app to access its state and change it according to the user actions/server updates, etc. It is available to any component
via dependency injection through the defined interfaces. In the begininning there's only onesuch interface: `IDataService` but many will follow. It's important to stress
the fact that the Model services are exposed to the **View-Model** layer by interfaces **only**. This is done to eliminate dependency on the actual model 
implementation and even allow it to be substituted for testing/demo purposes. 
Hence there are two projects in the **Model** layer: **Model.Contracts** and **Model** 

There are two main kinds of objects in this layer: **Services** which you already saw and **Models** which model specific entities, like users, bills, comments, etc.
Let's have a closer look at the model in the skeleton:

```csharp
public interface IAppModel : IModel<Guid>, IEditableModel
{       
}
```

```csharp
internal abstract class AppModel : EditableModel<Guid>, IAppModel
{                
}
```

Every model which represents a unique entity in the **Model** layer must have a unique identifier. 
We can specify the type of the identifier in the definition of the model and supply the value later. 
Unless there are some specific requirements/constraints we recommend using `System.Guid` for these purposes.

There's another class that actually registers the `IDataService` and its implementation `DataService` 
such that it's resolved correctly in the runtime. It will be explained later.

The framework classes contain all property notification boilerplate logic 
so we can concentrate on implementing the actuall app's needs.  

### Viewing items

Up to now the app doesn't really do much. So we would like to add some basic functionality and we will
start from viewing warehouse items feature. The first part of it should be defined in the **Model** layer.
 <!--Order: Displaying collection of items sync with obs coll; Displaying collection of items async with obs coll:Error!;Displaying coll of items async with wrap coll;Editing single item;Create/Delete single item;Undo/Redo;Login and activation;Close guard-->






