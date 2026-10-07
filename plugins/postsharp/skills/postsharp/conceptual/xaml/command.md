---
uid: command
title: "Command"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on using PostSharp's CommandAttribute aspect to implement commands in MVVM pattern-based GUI applications, reducing boilerplate code and simplifying the implementation of the `CanExecute` logic."
---
# Command

In GUI applications built using the MVVM pattern, it is common for the view model class to define commands that the user can perform in the associated XAML view. These commands are implemented as nested classes of the view model classes, and require a lot of boilerplate code.

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect allows you to implement commands in your view model classes without adding nested classes, which greatly reduces boilerplate code. The aspect also integrates with the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect, which makes it much easier to implement the `CanExecute` logic. 


## Creating a simple command


### Adding a command named LoadContactsList to your view model class

1. Implement the command logic in the `ExecuteLoadContactsList` method in your view model class. This method is the equivalent of the <xref:System.Windows.Input.ICommand.Execute(System.Object)> method. 

    Optionally, the `Execute` method can have one parameter of any type, which becomes the command parameter. 


2. Add a `LoadContactsListCommand` property to your view model class and mark it with the <xref:PostSharp.Patterns.Xaml.CommandAttribute> attribute. The property must be of type <xref:System.Windows.Input.ICommand> and have both a getter and a setter. 



### Example

The following code snippet defines a command named `LoadContactsList`. 

```csharp
using System.Collections.ObjectModel;
using System.Windows.Input;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    [NotifyPropertyChanged]
    public class ContactsListViewModel
    {
        #region LoadContactsList

        [Command]
        public ICommand LoadContactsListCommand { get; private set; }

        public void ExecuteLoadContactsList()
        {
            // ...
        }

        #endregion

        #region CreateContact

        [Command]
        public ICommand CreateContactCommand { get; private set; }

        public bool CanExecuteCreateContact
        {
            get
            {
                bool canCreateContact = true;
                // ...
                return canCreateContact;
            }
        }

        public void ExecuteCreateContact()
        {
            // ...
        }

        #endregion

        #region DeleteContact

        [Command]
        public ICommand DeleteContactCommand { get; private set; }

        public bool CanExecuteDeleteContact(int contactId)
        {
            bool canDeleteContact = true;
            // ...
            return canDeleteContact;
        }

        public void ExecuteDeleteContact(int contactId)
        {
            // ...
        }

        #endregion

        #region UpdateContact

        [Command(ExecuteMethod = nameof(UpdateContact), CanExecuteMethod = nameof(CanUpdate))]
        public ICommand UpdateContactCommand { get; private set; }

        public bool CanUpdate(object newData)
        {
            bool canUpdate = true;
            // ...
            return canUpdate;
        }

        public void UpdateContact(object newData)
        {
            // ...
        }

        #endregion

        public ObservableCollection<Contact> AllContacts { get; set; }

        /*  LoadContactList impl.
            this.AllContacts = new ObservableCollection<Contact>
            {
                new Contact { Id = 1, FullName = "Contact1", Email = "Email1", Notes = "Notes1", PictureUrl = "/user1.jpg"},
                new Contact { Id = 2, FullName = "Contact2", Email = "Email2", Notes = "Notes2", PictureUrl = "/user1.jpg"},
                new Contact { Id = 3, FullName = "Contact3", Email = "Email3", Notes = "Notes3", PictureUrl = "/user1.jpg"}
            };
        */
    }
}
```

After adding a new command to your view model class, you can bind it to a UI control in your XAML code.

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


## Determining whether your command can be executed

The UI controls in the GUI application can be enabled or disabled based on what actions are available to the user in the current state of the application. This feature is usually implemented by the <xref:System.Windows.Input.ICommand.CanExecute(System.Object)> method. 

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect provides a clean way to determine the availability of the actions. For each command in your view model class, you can implement a `CanExecute` property or method that validates whether this command can be currently executed. 


### Adding a CanExecute property

To allow the view to determine the availability of the `CreateContact` command, add a `public bool` property named `CanExecuteCreateContact` to the same class. 

```csharp
using System.Collections.ObjectModel;
using System.Windows.Input;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    [NotifyPropertyChanged]
    public class ContactsListViewModel
    {
        #region LoadContactsList

        [Command]
        public ICommand LoadContactsListCommand { get; private set; }

        public void ExecuteLoadContactsList()
        {
            // ...
        }

        #endregion

        #region CreateContact

        [Command]
        public ICommand CreateContactCommand { get; private set; }

        public bool CanExecuteCreateContact
        {
            get
            {
                bool canCreateContact = true;
                // ...
                return canCreateContact;
            }
        }

        public void ExecuteCreateContact()
        {
            // ...
        }

        #endregion

        #region DeleteContact

        [Command]
        public ICommand DeleteContactCommand { get; private set; }

        public bool CanExecuteDeleteContact(int contactId)
        {
            bool canDeleteContact = true;
            // ...
            return canDeleteContact;
        }

        public void ExecuteDeleteContact(int contactId)
        {
            // ...
        }

        #endregion

        #region UpdateContact

        [Command(ExecuteMethod = nameof(UpdateContact), CanExecuteMethod = nameof(CanUpdate))]
        public ICommand UpdateContactCommand { get; private set; }

        public bool CanUpdate(object newData)
        {
            bool canUpdate = true;
            // ...
            return canUpdate;
        }

        public void UpdateContact(object newData)
        {
            // ...
        }

        #endregion

        public ObservableCollection<Contact> AllContacts { get; set; }

        /*  LoadContactList impl.
            this.AllContacts = new ObservableCollection<Contact>
            {
                new Contact { Id = 1, FullName = "Contact1", Email = "Email1", Notes = "Notes1", PictureUrl = "/user1.jpg"},
                new Contact { Id = 2, FullName = "Contact2", Email = "Email2", Notes = "Notes2", PictureUrl = "/user1.jpg"},
                new Contact { Id = 3, FullName = "Contact3", Email = "Email3", Notes = "Notes3", PictureUrl = "/user1.jpg"}
            };
        */
    }
}
```

The `CanExecute` properties associated with the commands support the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. Therefore, if you add the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect to the view model class, the UI will be notified of changes in the `CanExecute` properties. 


### Adding a CanExecute method

A limitation of `CanExecute` properties is that they cannot depend on the input argument of the command. For example, with the `DeleteContact` command, you may want to validate whether the current user has the permission to delete the currently selected contact. If your availability logic depends on the input argument, you must use a `CanExecute` method instead of a property. 

To allow the UI to determine the availability of the `DeleteContact` command, add a `CanExecuteDeleteContact` method to the view model class. The method must have one parameter and return a `bool` value. The parameter type must be assignable from the type of the command's input argument. 

```csharp
using System.Collections.ObjectModel;
using System.Windows.Input;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    [NotifyPropertyChanged]
    public class ContactsListViewModel
    {
        #region LoadContactsList

        [Command]
        public ICommand LoadContactsListCommand { get; private set; }

        public void ExecuteLoadContactsList()
        {
            // ...
        }

        #endregion

        #region CreateContact

        [Command]
        public ICommand CreateContactCommand { get; private set; }

        public bool CanExecuteCreateContact
        {
            get
            {
                bool canCreateContact = true;
                // ...
                return canCreateContact;
            }
        }

        public void ExecuteCreateContact()
        {
            // ...
        }

        #endregion

        #region DeleteContact

        [Command]
        public ICommand DeleteContactCommand { get; private set; }

        public bool CanExecuteDeleteContact(int contactId)
        {
            bool canDeleteContact = true;
            // ...
            return canDeleteContact;
        }

        public void ExecuteDeleteContact(int contactId)
        {
            // ...
        }

        #endregion

        #region UpdateContact

        [Command(ExecuteMethod = nameof(UpdateContact), CanExecuteMethod = nameof(CanUpdate))]
        public ICommand UpdateContactCommand { get; private set; }

        public bool CanUpdate(object newData)
        {
            bool canUpdate = true;
            // ...
            return canUpdate;
        }

        public void UpdateContact(object newData)
        {
            // ...
        }

        #endregion

        public ObservableCollection<Contact> AllContacts { get; set; }

        /*  LoadContactList impl.
            this.AllContacts = new ObservableCollection<Contact>
            {
                new Contact { Id = 1, FullName = "Contact1", Email = "Email1", Notes = "Notes1", PictureUrl = "/user1.jpg"},
                new Contact { Id = 2, FullName = "Contact2", Email = "Email2", Notes = "Notes2", PictureUrl = "/user1.jpg"},
                new Contact { Id = 3, FullName = "Contact3", Email = "Email3", Notes = "Notes3", PictureUrl = "/user1.jpg"}
            };
        */
    }
}
```


## Overriding naming conventions

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect follows a predefined naming convention when looking for methods and properties associated with the command in your view model class. You can override the naming convention and choose your own member names by setting properties on the <xref:PostSharp.Patterns.Xaml.CommandAttribute>. The following table shows the default naming convention and the properties used to override member names. 


### Command pattern's naming conventions

| Member kind | Default name | Example | Override property |
|-------------|--------------|---------|-------------------|
| Command property | *CommandName* -or- *CommandName* Command  | `UpdateContact` -or- `UpdateContactCommand`  | N/A |
| Execute method | Execute *CommandName* -or- *CommandName*  | `ExecuteUpdateContact` -or- `UpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.ExecuteMethod> |
| CanExecute method | CanExecute *CommandName* -or- Can *CommandName*  | `CanExecuteUpdateContact` -or- `CanUpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.CanExecuteMethod> |
| CanExecute property | CanExecute *CommandName* -or- Can *CommandName*  | `CanExecuteUpdateContact` -or- `CanUpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.CanExecuteProperty> |

The following example shows a command that does not follow the naming conventions.

```csharp
using System.Collections.ObjectModel;
using System.Windows.Input;
using PostSharp.Patterns.Model;
using PostSharp.Patterns.Xaml;

namespace Samples.Xaml
{
    [NotifyPropertyChanged]
    public class ContactsListViewModel
    {
        #region LoadContactsList

        [Command]
        public ICommand LoadContactsListCommand { get; private set; }

        public void ExecuteLoadContactsList()
        {
            // ...
        }

        #endregion

        #region CreateContact

        [Command]
        public ICommand CreateContactCommand { get; private set; }

        public bool CanExecuteCreateContact
        {
            get
            {
                bool canCreateContact = true;
                // ...
                return canCreateContact;
            }
        }

        public void ExecuteCreateContact()
        {
            // ...
        }

        #endregion

        #region DeleteContact

        [Command]
        public ICommand DeleteContactCommand { get; private set; }

        public bool CanExecuteDeleteContact(int contactId)
        {
            bool canDeleteContact = true;
            // ...
            return canDeleteContact;
        }

        public void ExecuteDeleteContact(int contactId)
        {
            // ...
        }

        #endregion

        #region UpdateContact

        [Command(ExecuteMethod = nameof(UpdateContact), CanExecuteMethod = nameof(CanUpdate))]
        public ICommand UpdateContactCommand { get; private set; }

        public bool CanUpdate(object newData)
        {
            bool canUpdate = true;
            // ...
            return canUpdate;
        }

        public void UpdateContact(object newData)
        {
            // ...
        }

        #endregion

        public ObservableCollection<Contact> AllContacts { get; set; }

        /*  LoadContactList impl.
            this.AllContacts = new ObservableCollection<Contact>
            {
                new Contact { Id = 1, FullName = "Contact1", Email = "Email1", Notes = "Notes1", PictureUrl = "/user1.jpg"},
                new Contact { Id = 2, FullName = "Contact2", Email = "Email2", Notes = "Notes2", PictureUrl = "/user1.jpg"},
                new Contact { Id = 3, FullName = "Contact3", Email = "Email3", Notes = "Notes3", PictureUrl = "/user1.jpg"}
            };
        */
    }
}
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Xaml.CommandAttribute>
<br><xref:System.Windows.Input.ICommand>

**Other Resources**

<xref:inotifypropertychanged>
