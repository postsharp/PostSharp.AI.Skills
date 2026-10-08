---
uid: dependency-property
title: "Dependency Property"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document describes how to create and use XAML dependency properties in a PostSharp application. It provides examples of creating simple dependency properties, exposing the DependencyProperty object, validating values with contracts, and using a validation method."
---
# Dependency Property

XAML [dependency properties](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/dependency-properties-overview) extend the functionality of the CLR properties with features such as data binding, styling, and animation. Whenever you need to define a dependency property in your XAML application, you typically have to follow a strict implementation pattern and write a fair amount of boilerplate code. The <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect allows you to create custom dependency properties much faster, without writing repetitive code. 


## Creating a simple dependency property


### Adding a dependency property to your class

1. Add a new property to your class with a chosen name, a type, and a public getter and setter.


2. Mark your new property with the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> attribute. 


After you have marked your property with the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> attribute, you can immediately start using advanced features in your XAML code. 


### Example

The following code snippet comes from a custom control named `ContactCard`. The snippet defines a dependency property named `Phone`. The custom control defines other dependency properties not listed here. 

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```

The dependency properties can be used in XAML for advanced features. In the following code snippet, we use data binding with the dependency properties of the `ContactCard` custom control. 

```xml
<Window x:Class="Samples.Xaml.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Samples.Xaml"
        mc:Ignorable="d"
        Title="MainWindow" Height="462" Width="816">
    <Grid>
        <!-- #region LoadContactsList --> 
        <Button x:Name="LoadButton" Content="Load contacts" Command="{Binding LoadContactsListCommand}"
	        HorizontalAlignment="Left" Margin="361,10,0,0" VerticalAlignment="Top" Width="85" Height="20"/>
        <!-- #endregion -->
        <Button x:Name="CreateButton" Content="Create contact" Command="{Binding CreateContactCommand}" HorizontalAlignment="Left" Margin="361,35,0,0" VerticalAlignment="Top" Width="84" Height="20"/>
        <Button x:Name="DeleteButton" Content="Delete contact" Command="{Binding DeleteContactCommand}" HorizontalAlignment="Left" Margin="361,60,0,0" VerticalAlignment="Top" Width="86" Height="20"/>
        <local:ContactCard DataContext="{Binding SelectedValue, ElementName=ContactsDataGrid}" HorizontalAlignment="Left" Margin="10,10,0,0" VerticalAlignment="Top" Background="Bisque"/>
        <DataGrid x:Name="ContactsDataGrid"
                  HorizontalAlignment="Left" Margin="10,152,0,0" VerticalAlignment="Top" Height="248" Width="320" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Binding="{Binding Id}" ClipboardContentBinding="{x:Null}" Header="Id"/>
                <DataGridTextColumn Binding="{Binding FullName}" ClipboardContentBinding="{x:Null}" Header="Name"/>
                <DataGridTextColumn Binding="{Binding Email}" ClipboardContentBinding="{x:Null}" Header="Email"/>
                <DataGridTextColumn Binding="{Binding Notes}" ClipboardContentBinding="{x:Null}" Header="Notes"/>
            </DataGrid.Columns>
        </DataGrid>
        <!-- #region ContactCard bindings -->
        <local:ContactCard FullName="{Binding ContactName}" Phone="{Binding ContactPhone}" Email="{Binding ContactEmail}" Notes="{Binding ContactNotes}"
                           x:Name="CurrentContactCard" HorizontalAlignment="Left" Margin="478,270,0,0" VerticalAlignment="Top" Background="Bisque"/>
        <!-- #endregion -->
        <TextBox x:Name="NameTextBox" Text="{Binding ContactName}" HorizontalAlignment="Left" Height="23" Margin="678,165,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="120"/>
        <TextBox x:Name="EmailTextBox" Text="{Binding ContactEmail}" HorizontalAlignment="Left" Height="23" Margin="678,193,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="120"/>
        <TextBox x:Name="NotesTextBox" Text="{Binding ContactNotes}" HorizontalAlignment="Left" Height="23" Margin="678,221,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="120"/>
        <Label Content="Name" HorizontalAlignment="Left" Margin="620,162,0,0" VerticalAlignment="Top"/>
        <Label Content="Email" HorizontalAlignment="Left" Margin="620,193,0,0" VerticalAlignment="Top" RenderTransformOrigin="0.474,-0.077"/>
        <Label Content="Notes" HorizontalAlignment="Left" Margin="620,221,0,0" VerticalAlignment="Top"/>
    </Grid>
</Window>
```


## Exposing the DependencyProperty object

If you want to manipulate the dependency property using the XAML API in C# or VB, you need to get the <xref:System.Windows.DependencyProperty> for this specific property. One solution is to use the <xref:PostSharp.Patterns.Xaml.DependencyPropertyServices.GetDependencyProperty(System.Type,System.String)> method, but this is long and error-prone. 

A better solution is to add a `public static` property of type <xref:System.Windows.DependencyProperty> with the same name as your dependency property plus the `Property` suffix. This property is picked up by the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect and set to the proper <xref:System.Windows.DependencyProperty> value. 


### Example

The following example shows how to expose the <xref:System.Windows.DependencyProperty> for the `Phone` property. 

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```

```csharp
using System.Windows;
using System.Windows.Data;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            this.DataContext = new ContactEditViewModel();

            #region GetDependencyProperty
            this.CurrentContactCard.SetBinding(ContactCard.PhoneProperty, new Binding("ContactPhone"));
            #endregion
        }
    }
}
```


## Validating the value of the dependency property with contracts

PostSharp Code Contracts (see <xref:contracts>) provide a convenient way to validate the values of the dependency properties. To add the validation to your dependency property, you need to apply a contract attribute to that property. 


### Example

The following code snippet shows how to validate a dependency property using a code contract.

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```


## Validating the value of the dependency property with a validation method

If you need more complex validation for your dependency property, you can implement it in a dedicated validation method. To define a validation method for the `Email` dependency property, add a new method named `ValidateEmail` to the same class where the property is declared. The method must accept one argument of a type that is assignable from the property type and return a `bool` value. 

The following list shows the method signatures you can use when implementing the validation method, where `TPropertyType` is the type of the dependency property and `TDeclaringType` is the class where your property is declared. 

* `static bool ValidatePropertyName(TPropertyType value)`
* `static bool ValidatePropertyName(DependencyProperty property, TPropertyType value)`
* `static bool ValidatePropertyName(TDeclaringType instance, TPropertyType value)`
* `static bool ValidatePropertyName(DependencyProperty property, TDeclaringType instance, TPropertyType value)`
* `bool ValidatePropertyName(TPropertyType value)`
* `bool ValidatePropertyName(DependencyProperty property, TPropertyType value)`

### Example

The following code snippet shows how to validate a dependency property using a validation method.

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```


## Reacting to the changes of the dependency property value

The WPF property system can automatically notify you about the dependency property value changes via callback methods. This can be useful when, for example, you need to update the visual presentation of your custom UI control in response to a change of its property. This section shows how you can define a property change callback method with the dependency property pattern of PostSharp.

To define a property change callback method for the `PictureUrl` dependency property, add a new method named `OnPictureUrlChanged` to the same class where the property is declared. The method does not have to accept any arguments and must have a `void` return type. Implement your property change handling logic inside this new method. 

The following list shows the method signatures you can use when implementing the property change callback method. `TDeclaringType` is the class where your property is declared. 

* `static void OnPropertyNameChanged()`
* `static void OnPropertyNameChanged(DependencyProperty property)`
* `static void OnPropertyNameChanged(TDeclaringType instance)`
* `static void OnPropertyNameChanged(DependencyProperty property, TDeclaringType instance)`
* `void OnPropertyNameChanged()`
* `void OnPropertyNameChanged(DependencyProperty property)`

### Example

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```


## Implementing INotifyPropertyChanged

You may also want to notify the users of your class when a dependency property value changes. In this case, you would normally need to implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface in your class and raise the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event. PostSharp helps you to automate this task using the <xref:inotifypropertychanged> pattern. To raise the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event every time any of the dependency properties in your class changes its value, mark your class with the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> attribute. 

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```


## Overriding the naming conventions

The <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect follows a predefined naming convention when looking for methods and properties associated with the dependency property in your class. You can override the naming convention and choose your own member names by setting properties on the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute>. The following table shows the default naming convention and the properties used to override member names. 


### Dependency property pattern's naming conventions

| Member kind | Default name | Example | Override property |
|-------------|--------------|---------|-------------------|
| Value validation method | Validate *PropertyName*  | `ValidatePhoneNumber` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.ValidateValueMethod> |
| Property changed callback method | On *PropertyName* Changed  | `OnPhoneNumberChanged` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.PropertyChangedMethod> |
| Registration property | *PropertyName* Property  | `PhoneNumberProperty` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.RegistrationProperty> |


### Example

```csharp
using System.Security.Policy;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using PostSharp.Patterns.Contracts;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    #region ContactCard header

    [NotifyPropertyChanged]
    public partial class ContactCard : UserControl
    {
        // ...

        #endregion

        private const int MAX_LENGTH = 250;
        private static Regex EmailRegex = new Regex(@"^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,6}$");

        public ContactCard()
        {
            InitializeComponent();
        }

        #region PictureUrl property

        [DependencyProperty]
        public string PictureUrl { get; set; }

        private void OnPictureUrlChanged()
        {
            this.ProfileImage.Source = this.LoadImageFromUrl(this.PictureUrl);
        }

        #endregion

        private BitmapImage LoadImageFromUrl(string url)
        {
            return null;
        }

        #region FullName property

        [DependencyProperty]
        [NotEmpty]
        public string FullName { get; set; }

        #endregion

        #region Email property

        [DependencyProperty]
        public string Email { get; set; }

        private bool ValidateEmail(string value)
        {
            return EmailRegex.IsMatch(value);
        }

        #endregion

        #region Phone property with registration
        #region Phone property
        [DependencyProperty]
        public string Phone { get; set; }
        #endregion

        public static DependencyProperty PhoneProperty { get; private set; }
        #endregion

        #region Notes property

        [DependencyProperty(ValidateValueMethod = "ValidateStringMaxLength" )]
        public string Notes { get; set; }

        private bool ValidateStringMaxLength(string value)
        {
            if (string.IsNullOrEmpty(value))
                return true;

            return value.Length <= MAX_LENGTH;
        }

        #endregion
    }
}
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute>
<br><xref:System.Windows.DependencyProperty>
<br><xref:PostSharp.Patterns.Xaml.DependencyPropertyServices>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>
<br><xref:System.ComponentModel.INotifyPropertyChanged>
<br><xref:PostSharp.Patterns.Contracts.NotEmptyAttribute>

**Other Resources**

[Dependency Properties Overview](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/dependency-properties-overview)
<br><xref:contracts>
<br><xref:inotifypropertychanged>
