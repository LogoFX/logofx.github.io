---
layout: page
title: Getting started
subtitle: Start lean
---

Xamarin.Forms is an excellent framework for creating cross-platform applications. Here you can find
some recipes to help you get started using `LogoFX` packages. Let's start with a template project. 
You can download the skeleton [here](https://github.com/LogoFX/Samples.GetStarted/archive/v1.0.2.zip) 

When you extract the skeleton you should see the following solution:
![alt text](../assets/skeleton-initial-xamarin-forms.png)

### Launcher

There are two platform projects in this solution: Android and iOS. Also there's a launcher project
for Xamarin.Forms and the actual presentation which uses Xamarin.Forms as well. Let's have a closer look at these projects and their 
startup objects

#### Android

```csharp
[Activity(Label = "Samples.GetStarted.Droid", Icon = "@drawable/icon", Theme = "@style/MyTheme", MainLauncher = true, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
public class MainActivity : FormsApplicationActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        Xamarin.Forms.Forms.Init(this, bundle);

        LoadApplication(ContainerContext.Resolver.Resolve<FormsApp>());
    }
}
```

This is the main activity of the app. Note that immediately after initializing the Xamarin engine we load the `FormsApp` which is
a Xamarin.Forms application. It's resolved by using the dependency resolver which is initialized in the shared library (this will
be explained later).
Another part which is crucial for proper initialization of all underlying framework is described below:
```csharp
[Application]
public class Application : LogoFXApplication<FormsApp, Bootstrapper, ExtendedSimpleContainerAdapter>
{
    public Application(IntPtr javaReference, JniHandleOwnership transfer) : base(javaReference, transfer)
    {
    }

    protected override IEnumerable<Assembly> SelectAssemblies()
    {
        return
            new[]
            {                                        
                Assembly.GetAssembly(typeof(ShellViewModel))
            };
    }
}
```
It's decorated with the `Application` attribute so that it's invoked during app startup. Here we specify
the type of the Xamarin.Forms app to be loaded - `FormsApp`, the internal composition bootstrapper which will be responsible
for modules and extensions loading - `Bootstrapper`, 
and finally the type of the ioc container to be used in the app - `ExtendedSimpleContainerAdapter`.
We also specify the type of the assembly containing the root view model of the Xamarin.Forms app - `ShellViewModel`

#### iOS

```csharp
public class Application
{
    // This is the main entry point of the application.
    static void Main(string[] args)
    {
        // if you want to use a different Application Delegate class from "AppDelegate"
        // you can specify it here.
        UIApplication.Main(args, null, "AppDelegate");
    }
}

[Register("AppDelegate")]
public class AppDelegate : Xamarin.Forms.Platform.iOS.FormsApplicationDelegate
{
    private readonly ApplicationDelegate appDelegate = new ApplicationDelegate();

    public override bool FinishedLaunching(UIApplication app, NSDictionary options)
    {
        Xamarin.Forms.Forms.Init();

        LoadApplication(ContainerContext.Resolver.Resolve<FormsApp>());

        return base.FinishedLaunching(app, options);
    }
}
```

It's quite similar to Android in a sense that immediately after initializing Xamarin.Forms engine we resolve
an instance of Xamarin.Forms app. Note the creation of the `ApplicationDelegate` here:

```csharp
public class ApplicationDelegate : LogoFXApplicationDelegate<FormsApp, Bootstrapper, ExtendedSimpleContainerAdapter>
{
    protected override IEnumerable<Assembly> SelectAssemblies()
    {
        return
            new[]
            {                                        
                typeof(ShellViewModel).Assembly
            };
    }
}
```
Here we specify the type of the Xamarin.Forms app to be loaded - `FormsApp`, the internal composition bootstrapper which will be responsible
for modules and extensions loading - `Bootstrapper`, 
and finally the type of the ioc container to be used in the app - `ExtendedSimpleContainerAdapter`.
We also specify the type of the assembly containing the root view model of the Xamarin.Forms app - `ShellViewModel`. 
Again it's similar to the initialization for Android platform.

#### Xamarin.Forms

After we dealt with the platform-specific code let's have a look at the shared portion which is implemneted in `.NETStandard`:
```csharp
public class FormsApp : LogoFXApplication<ShellViewModel>
{
    public FormsApp(Bootstrapper bootstrapper, IDependencyRegistrator dependencyRegistrator) :
    base(
            bootstrapper,
            dependencyRegistrator)
    {
    }
}
```
This class represents the Xamarin.Forms app which will be run in both platforms and it basically specifies
the type of the root view model and passes along the bootstrapper and the dependency registrator.

```csharp
public class Bootstrapper : BootstrapperBase
{
    public Bootstrapper(IDependencyRegistrator dependencyRegistrator)
        :base(dependencyRegistrator)
    {

    }        

    public override string[] Prefixes
    {
        get
        {
            return new [] { "Samples" };
        }
    }
}
```

This is the actual bootstrapper. It may contain some additional logic like in this case for assemblies' prefixes
which are used for efficient modules loading.

That's all the launcher setup you need here. So let's move to the application itself.

### Application

In this particular example there are only two components to the application - view and its associated view-model.
If you're not familiar with the MVVM approach please refer to this [amazing resourse](https://www.codeproject.com/Articles/100175/Model-View-ViewModel-MVVM-Explained).
For now pay attention to the naming convention which allows us to implicitly match the view and view-model pairs.
```csharp
public class ShellViewModel
{
        
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage
    xmlns="http://xamarin.com/schemas/2014/forms" 
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" 
    x:Class="Samples.GetStarted.Forms.Presentation.Shell.Views.ShellView">
    <Grid>
        <Label Text="Get started" VerticalTextAlignment="Center" HorizontalTextAlignment="Center"/>
    </Grid>	
</ContentPage>
```

Build and run the solution. You should see something like
![alt text](../assets/skeleton-initial-xamarin-forms-result-ios.png)