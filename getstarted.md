---
layout: page
title: Getting started
subtitle: Start lean
---

So you decided to create a new application that will look stunning and make life easier
for the users. Good for you. We're here to help ;)

You can always download the skeleton from [here](https://github.com/LogoFX/Samples.GetStarted/archive/v1.0.0.zip)

Every great application starts small. 
In our case we want to show a simple text inside the first view.
This can be done in a few simple steps:

Open Visual Studio or any other IDE you're familiar with.

Create a new client-side .NET project for the platform of your choice. 
We will use WPF here for the sake of simplicity but you can use anything.

If you have chosen WPF you should see `App.xaml`, `App.xaml.cs` and `MainWindow.xaml` files

Now we would like to use the **Bootstrapping** 
component of the **LogoFX** framework to make things easier
and more extensible:

Install the package via package manager or command-line: 
`Install-Package LogoFX.Client.Bootstrapping`
This is the main bootstrapping package. 
In this particular example we will install one more package
which will be covered in another topic. 
For now install it by typing 
`Install-Package LogoFX.Client.Bootstrapping.Adapters.SimpleContainer`

Then go ahead and add a file named `AppBootstrapper.cs`. This file should include the following code
```csharp
using LogoFX.Client.Bootstrapping.Adapters.SimpleContainer;
using LogoFX.Client.Bootstrapping.Samples.WPF.ViewModels;

namespace LogoFX.Client.Bootstrapping.Samples.WPF
{
   public class AppBootstrapper : 
   BootstrapperContainerBase<ExtendedSimpleContainerAdapter>
   .WithRootObject<ShellViewModel>
   {
      public AppBootstrapper()
         :base (new ExtendedSimpleContainerAdapter())
      {
			
      }
   }
}
```

Go ahead and try to compile the code. It won't compile complaining about missing `ShellViewModel`.
Indeed we haven't added our first view-model to the application (If you're not familiar with MVVM, please
refer to this [amazing resourse](https://www.codeproject.com/Articles/100175/Model-View-ViewModel-MVVM-Explained))

Let's add the view-model and its view. Create a folder named `ViewModels` 
at the root level of your application and add a file there named `ShellViewModel.cs`:

```csharp
namespace LogoFX.Client.Bootstrapping.Samples.WPF.ViewModels
{
    public class ShellViewModel
    {
        public string DisplayName => "Samples.Bootstrapping";
    }
}
```

The view is still missing so we should add it as well. Create a folder named `Views`
at the root level of your application and add a file there named `ShellView.xaml` 
(add it via *Create Window* option)
Delete the code-behind portion of this file and the following markup to the file itself:
```xml
<Window 
x:Class="LogoFX.Client.Bootstrapping.Samples.WPF.Views.ShellView"
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
xmlns:viewModels="clr-namespace:LogoFX.Client.Bootstrapping.Samples.WPF.ViewModels"
mc:Ignorable="d"
d:DataContext="{d:DesignInstance viewModels:ShellViewModel}"
Title="{Binding DisplayName, Mode=OneTime}"
Height="350" Width="350">

    <Grid>
        <TextBlock Text="Hello Bootstrapping" HorizontalAlignment="Center" VerticalAlignment="Center" FontSize="24" />
    </Grid>

</Window>
```

The only missing part is the link between the app's entry point and the bootstrapper:
Delete the `MainWindow.xaml` and modify the `App.xaml` and `App.xaml.cs` files:

`App.xaml`
```xml
<Application 
x:Class="LogoFX.Client.Bootstrapping.Samples.WPF.App"
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" />
```
   
`App.xaml.cs`   
```csharp
namespace LogoFX.Client.Bootstrapping.Samples.WPF
{    
   public partial class App
   {
      public App()
      {
         var appBootstrapper = new AppBootstrapper();
         appBootstrapper.Initialize();
      }
   }
}
```

That's it! Build the solution and run it. You should see something like this:

![alt text](../assets/samples-bootstrapping-final-result.png)


