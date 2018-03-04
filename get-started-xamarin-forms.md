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

### Presentation

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

Great! The app is up and kicking. Still it doesn't do very much so we should start adding some features into it.
Let's start with editing a warehouse item.

### Model

The main philosophy behind the framework is that the **Model** layer is the central part of any client app. It serves as a single source of truth
and allows any part of the app to access its state and change it according to the user actions/server updates, etc. It is available to any component
via dependency injection through the defined interfaces. In the begininning there's only one such interface: `IDataService` but many will follow. It's important to stress
the fact that the Model services are exposed to the **View-Model** layer by interfaces **only**. This is done to eliminate dependency on the actual model 
implementation and even allow it to be substituted for testing/demo purposes. 
Hence there are two projects in the **Model** layer: **Model.Contracts** and **Model** 

There are two main kinds of objects in this layer: **Services** which you already saw and **Entities** which model specific entities, like users, bills, comments, etc.
Let's have a closer look at the model in the skeleton:

```csharp
public interface IAppModel : IModel<Guid>, IEditableModel, IUndoRedo
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
such that it's resolved correctly in the runtime.

```csharp
public class Module : ICompositionModule<IDependencyRegistrator>
{
    public void RegisterModule(IDependencyRegistrator dependencyRegistrator)
    {
        dependencyRegistrator               
            .AddSingleton<IDataService, DataService>();
        //TODO:Add more services registration
    }
}
```

### Editing item

First of all let's define the contract of the new entity and its basic implementation

 ```csharp
public interface IWarehouseItem : IAppModel
{
    string Kind { get; }
    double Price { get; set; }
    int Quantity { get; set; }
    double TotalCost { get; }        
}

internal sealed class WarehouseItem : AppModel, IWarehouseItem
{
    public WarehouseItem(
            string kind,
            double price,
            int quantity)
    {
        Id = Guid.NewGuid();
        _kind = kind;
        _price = price;
        _quantity = quantity;
    }

    private string _kind;        
    public string Kind
    {
        get => _kind;
        set => SetProperty(ref _kind, value);
    }

    private double _price;        
    public double Price
    {
        get => _price;
        set
        {
            SetProperty(ref _price, value);
            NotifyOfPropertyChange(() => TotalCost);
        }
    }

    private int _quantity;    
    public int Quantity
    {
        get => _quantity;
        set
        {
            SetProperty(ref _quantity, value);
            NotifyOfPropertyChange(() => TotalCost);
        }
    }

    public double TotalCost => _quantity * _price;
}
 ``` 

Let's have a look of what we have here. the `IWarehouseItem` is merely a contract which describes the data the entity should hold and the
level of access for the external consumers. It shouldn't contain any implementation logic and is therefore modeled as an interface.
The `WarehouseItem` on the other hand contains the actual logic of setting the value and raising the proper notification event.
This is achieved via the `SetProperty` method.

The `ShellViewModel` should consume the models and dispatch command via the appropriate services. In our case there's only one such service `IDataService`.
So let's expose the entity via the interface and some fake implementation:
```csharp
public interface IDataService
{
    IWarehouseItem SingleItem { get; }        
}

internal sealed class DataService : NotifyPropertyChangedBase<DataService>, IDataService
{        
    public IWarehouseItem SingleItem { get; } = new WarehouseItem("PC", 25.43, 8);        
}
```

You may pay attention that it's the same model as in WPF. That's right, the **Model** is cross-platform
and may be used wherever there's a .NET runtime!

### Editing item - presentation part

With that in place we're ready to upgrade the `ShellViewModel` and `ShellView` to
allow editing the entity. The `LogoFX` framework contains a view model for this scenario:

```csharp
public class ShellViewModel : EditableObjectViewModel<IWarehouseItem>
{        
    public ShellViewModel(IDataService dataService)
        :base(dataService.SingleItem)
    {     
            
    }

    protected override async Task<bool> SaveMethod(IWarehouseItem model)
    {
        //TODO: Add custom saving logic - handle exceptions, etc.
        await Task.Delay(50);
        return true;
    }

   private ICommand _undoCommand;
   public ICommand UndoCommand => _undoCommand ?? (_undoCommand = ActionCommand.When(() => Model.CanUndo).Do(() => Model.Undo()).RequeryOnPropertyChanged(this, () => Model.CanUndo));

   private ICommand _redoCommand;
   public ICommand RedoCommand => _redoCommand ?? (_redoCommand = ActionCommand.When(() => Model.CanRedo).Do(() => Model.Redo()).RequeryOnPropertyChanged(this, () => Model.CanRedo));
}
```

This requires adding two packages: `LogoFX.Client.Mvvm.Commanding.Core` and `LogoFX.Client.Mvvm.ViewModel.Extensions.Core` to the `Presentation.Shell` project.
As you can see the amount of code needed to be added to implement this feature is fairly small.
Below is the view portion of the feature:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage
    xmlns="http://xamarin.com/schemas/2014/forms" 
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" 
    x:Class="Samples.GetStarted.Forms.Presentation.Shell.Views.ShellView">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="20" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <StackLayout Margin="0, 20" Grid.Row="0">
            <Label Text="Editing single item"
               FontSize="30"
               FontAttributes="Bold"
               HorizontalOptions="Center" />

            <TableView Intent="Form"
                   Margin="10, 0">
                <TableRoot>
                    <TableSection Title="Warehouse item">
                        <ViewCell>
                            <StackLayout Orientation="Horizontal" Padding="15, 0">
                                <Label Text ="Kind" />  
                                <Label Text="{Binding Model.Kind, Mode=OneWay}" Margin="50, 0" />
                            </StackLayout>                                          
                        </ViewCell>
                        <EntryCell Label="Price"
                               Text="{Binding Model.Price, Mode=TwoWay}"
                               Placeholder = "Enter the price here" />
                        <EntryCell Label="Quantity"
                               Text="{Binding Model.Quantity, Mode=TwoWay}"
                               Placeholder = "Enter the quantity here" />
                        <ViewCell>
                            <StackLayout Orientation="Horizontal" Padding="15, 0">
                                <Label Text ="Total cost" />  
                                <Label Text="{Binding Model.TotalCost, Mode=OneWay}" Margin="10, 0" />
                            </StackLayout>                                          
                        </ViewCell>
                    </TableSection>
                </TableRoot>
            </TableView>
            <Label      
                   TextColor="Red"
                   HorizontalOptions="Center"
                   Text="{Binding Path=Model.Error, Mode=OneWay}" />
        </StackLayout>
        <Grid Grid.Row="2" Margin="15, 0">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="Auto" />
                <ColumnDefinition Width="*" />
            </Grid.ColumnDefinitions>

            <Button Grid.Column="0"
                    Text="Save"
                    WidthRequest="50"
                    HeightRequest="50"
                    Command="{Binding ApplyCommand, Mode=OneWay}"
                    Margin="5,0,0,0"
                    VerticalOptions="Center"                    >
            </Button>

            <Button Grid.Column="1"                
                    Margin="5,0,0,0"
                    Text="Cancel"
                    Command="{Binding CancelChangesCommand, Mode=OneWay}"
                    WidthRequest="50"
                    HeightRequest="50"
                    VerticalOptions="Center">
            </Button>

            <Button Grid.Column="2"                
                    Margin="5,0,0,0"
                    Text="Undo"
                    Command="{Binding UndoCommand, Mode=OneWay}"
                    WidthRequest="50"
                    HeightRequest="50"
                    VerticalOptions="Center">
            </Button>

            <Button Grid.Column="3"                
                    Margin="5,0,0,0"
                    Text="Redo"
                    Command="{Binding RedoCommand, Mode=OneWay}"
                    WidthRequest="50"
                    HeightRequest="50"
                    VerticalOptions="Center">
            </Button>          
        </Grid>
    </Grid>   
</ContentPage>
```

That's it. Let's build the solution and see what we have:
![alt text](../assets/tutorial-editing-single-item-initial-ios.png)

Pay attention that editing values will enable/disable the correspondent buttons
like in the most common editing scenarios. All this functionality is ready out of the box!

### Editing item - validation

The last missing piece would be the validation and error messages. It's pretty common to have client-side
validation defined by the property metadata. In this case we will use attributes:
```csharp
public class DoublePositiveValidation : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        try
        {
            var number = (double)value;
            if (number < 0.0)
            {
                return new ValidationResult(ErrorMessage);
            }
        }
        catch (Exception)
        {
            return new ValidationResult("Number is invalid");
        }
        return ValidationResult.Success;
    }
}

public class NumberValidation : ValidationAttribute
{
    public NumberValidation()
    {
        Minimum = int.MinValue;
        Maximum = int.MaxValue;
    }

    public int Minimum { get; set; }

    public int Maximum { get; set; }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        var number = (int)value;

        if (number < Minimum || number > Maximum)
        {
            return new ValidationResult(ErrorMessage);
        }

        return ValidationResult.Success;
    }
}

public class StringValidation : ValidationAttribute
{
    public StringValidation()
    {
        MaxLength = 256;
    }

    public int MaxLength { get; set; }
    public bool IsNulOrEmptyAllowed { get; set; }
    public bool IsAlphaNumeric { get; set; }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        var str = value as string;

        var isValid = IsNulOrEmptyAllowed || !string.IsNullOrEmpty(str);
        if (!isValid)
        {
            return new ValidationResult(string.Format("{0} should not be empty.", validationContext.ObjectInstance));
        }

        if (str != null)
        {
            var length = str.Length;
            isValid = length <= MaxLength;
            if (!isValid)
            {
                return new ValidationResult(string.Format("Provided string is {0} chars length. Maximal length allowed is {1}.", length, MaxLength));
            }

            isValid = !IsAlphaNumeric || str.Replace("-", "").All(char.IsLetterOrDigit);
            if (!isValid)
            {
                return new ValidationResult("Only alphanumeric characters or '-' are allowed.");
            }
        }
        return ValidationResult.Success;
    }
}

internal sealed class WarehouseItem : AppModel, IWarehouseItem
{
    public WarehouseItem(
            string kind,
            double price,
            int quantity)
    {
        Id = Guid.NewGuid();
        _kind = kind;
        _price = price;
        _quantity = quantity;
    }

    private string _kind;  
    [StringValidation(IsNulOrEmptyAllowed = false, MaxLength = 63)]      
    public string Kind
    {
        get => _kind;
        set => SetProperty(ref _kind, value);
    }

    private double _price;  
    [DoublePositiveValidation(ErrorMessage = "Price must be positive.")]     
    public double Price
    {
        get => _price;
        set
        {
            SetProperty(ref _price, value);
            NotifyOfPropertyChange(() => TotalCost);
        }
    }

    private int _quantity;   
    [NumberValidation(Minimum = 1, ErrorMessage = "Quantity must be positive.")] 
    public int Quantity
    {
        get => _quantity;
        set
        {
            SetProperty(ref _quantity, value);
            NotifyOfPropertyChange(() => TotalCost);
        }
    }

    public double TotalCost => _quantity * _price;
}
```

If you try to modify the values you should see the appropriate error message
and the buttons will be automatically disabled until you fix the value.
![alt text](../assets/tutorial-editing-single-item-validation-ios.png)

### Summary

So far we have seen the power and simplicity of `LogoFX` with regard to item editing lifecycle
and the app bootstrapping. Stay tuned for more ;)