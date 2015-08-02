---
layout: post
title: Getting Started With MVVM in WPF
date: 2010-01-12 21:13:02.000000000 +05:30
---
I've been working with MVVM and WPF for a couple of weeks now. I decided to log what I've learned here. Here goes a getting started tutorial with MVVM.

[Download the source code](http://dl.dropbox.com/u/8855362/blog/uploads/MvvmStarter.zip).

The Model-View-ViewModel pattern was introduced by [John Gossman](http://blogs.msdn.com/johngossman/archive/2005/10/08/478683.aspx) to effectively utilize the functionality of WPF. Since then, MVVM has been used in a number of WPF applications with very impressive results. MVVM has three components:

* *Model:* It is your data or classes that represent entities in your application. It normally contains no WPF-specific code.
* *View:* This is the User Interface element visible to the user. Its DataContext is its ViewModel.
* *ViewModel:* It contains all the data that needs to be displayed and procedures to modify the model at will. The magic about MVVM is that the ViewModel knows nothing about the View.

You see that this is very loosely coupled. The View knows the ViewModel but the ViewModel does not know the View. You can very easily replace the View without affecting the ViewModel. This is very useful in Developer/Designer teams where the Developer improves the ViewModel and the Designer enhances the View.

The fact that the ViewModel does not know anything about the View comes as a bit of surprise. There is one more surprise: a typical View in MVVM does not need a code-behind (except for the general boiler-plate code that calls the `InitializeComponent()` method from the constructor)!

You may be wondering how the view updates itself when the ViewModel changes and how it handles user interaction like button clicks etc. This is what makes MVVM specific to WPF.

The controls in the View bind themselves to the corresponding properties in the ViewModel. The changes in ViewModel will be reflected in the view, thanks to Data Binding in WPF. (Otherwise we would have had to handle every event and then update the view accordingly.)

As for user interaction, we always have had commands in WPF. MVVM leverages on this feature. Instead of writing event handling code for button clicks, we bind the buttons (or MenuItems) to Commands in the ViewModel. Every button (even the SaveCustomer, CloseTab etc.) binds itself to a command which the ViewModel exposes. This command delegates its job to a method in the ViewModel that gets the work done. But the problem is that there is no built-in command in WPF that does that. We have a RoutedCommand that targets UIElements but not methods. Here comes to the scene a new command that targets methods, the DelegateCommand or the RelayCommand. Controls can bind the RelayCommand (that the ViewModel exposes) and invoke methods in the ViewModel.
The DelegateCommand implements the `ICommand` interface and delegates the `Execute` and `CanExecute` methods in the interface to methods in the ViewModel.

```csharp
using System;
using System.Windows.Input;

namespace MvvmSample
{
    public class DelegateCommand : ICommand
    {
        readonly Action<object> _execute;
        readonly Predicate<object> _canExecute;

        public DelegateCommand(Action<object> execute) : this(execute, null)
        {
        }

        public DelegateCommand(Action<object> execute, Predicate<object> canExecute)
        {
            if (execute == null)
                throw new ArgumentNullException("execute");

            _execute = execute;
            _canExecute = canExecute;
        }

        public void Execute(object parameter)
        {
            _execute(parameter);
        }

        public bool CanExecute(object parameter)
        {
            return _canExecute == null ? true : _canExecute(parameter);
        }

        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }
    }
}
```

Now we are going to create a simple tabbed interface. When you File->New Tab, a new tab opens up. When you click File->Exit, the application closes. You may think that this is very simple but achieving this in MVVM needs a lot of ground on it.

Open a new WPF application. Change the `Window1` to `MainWindow` (I just hate Window1). Replace the Grid element in the `MainWindow` with the following markup.

```xml
<DockPanel>
    <Menu DockPanel.Dock="Top">
        <MenuItem Header="_File">
            <MenuItem Header="New _Tab" />
            <Separator />
            <MenuItem Header="E_xit" />
        </MenuItem>
    </Menu>
    <TabControl />
</DockPanel>
```

You know it simply adds a menu to the top and a tab control. Now we create a simple `MainWindowViewModel` class. Since the View binds its controls to the Properties in the ViewModel, we need to implement the INotifyPropertyChanged interface. But in this example we are not going to need it. Create an empty class `MainWindowViewModel`.

We know that the view sets its `DataContext` to the ViewModel. We'll use the `Application` class to set it for the view. Remove the `StartupUri` attribute from the Application.xaml file and override the `OnStartup` method to initialize a new `MainWindow` and a new `MainWindowViewModel`, set the `DataContext` of the `MainWindow` to the `MainViewModel`, set the `MainWindow` property of `Application` class this `MainWindow` and finally show the `MainWindow`.

```csharp
protected override void OnStartup(StartupEventArgs e)
{
    MainWindow mainWindow = new MainWindow();
    MainWindowViewModel mainWindowViewModel = new MainWindowViewModel();
    mainWindow.DataContext = mainWindowViewModel;
    base.MainWindow = mainWindow;
    mainWindow.Show();
}
```

Now we have setup the base for MVVM. But our application does nothing. When we click New Tab or Exit nothing happens. To hook this up, we need a command. Let's first implement the Exit Command. In the `MainWindowViewModel`, add a new `DelegateCommand`, `ExitCommand`.

```csharp
private DelegateCommand exitCommand;
public ICommand ExitCommand
{
    get
    {
        if(exitCommand == null)
            exitCommand = new DelegateCommand(Exit);
        return exitCommand;
    }
}

private void Exit(object obj) 
{
    Application.Current.Shutdown();
}
```

Now when the `Execute()` method on `ExitCommand` is called, Exit method is invoked. We use the Exit method to shutdown the application. Now all we have to do is bind the Exit menu item to this command. When you click Exit, the Exit method is called and the application shuts down.

```xml
<MenuItem Header="E_xit" Command="{Binding ExitCommand}" />
```

Now, let's implement the Add Tab functionality. Obviously, we need a new command, `AddTabCommand`.

```csharp
private DelegateCommand addTabCommand;
public ICommand AddTabCommand
{
    get
    {
        if (addTabCommand == null)
            addTabCommand = new DelegateCommand(AddTab);
        return addTabCommand;
    }
}

private void AddTab(object obj)
{
    throw new NotImplementedException();
}
```

Here we have a problem. The ViewModel does not know the View and does not know the `TabControl` in it. How is it going to add a tab into that `TabControl`? If it were a simple code-behind, we would have given the tab control a name and hooked up an event handler to add a new tab to the control. But what do we do now? Any guesses?

Let's use a trick. We know the View can modify itself to reflect changes in the ViewModel. Let's exploit this. We will maintain a list of tabs in the `ViewModel` and the `TabControl` binds itself to this list. Now if we add a new item in the list, a new tab is added in the view! Wonderful!

```csharp
public ObservableCollection<TabItem> Tabs { get; set; }

public MainWindowViewModel()
{
    Tabs = new ObservableCollection<TabItem>();
}
```

We need an observable collection to bind lists (It's why we did not need to implement the `INotifyPropertyChanged` explicitly here). Now, we need to bind the `TabControl` to this collection.

```xml
<TabControl ItemsSource="{Binding Tabs}" />
```

Now, the `AddTab` method adds a new `TabItem` into the list and the View updates itself.

```csharp
private void AddTab(object obj)
{
    TabItem tab = new TabItem();
    tab.Header = "New Tab";
    Tabs.Add(tab);
    tab.Focus();
}
```

The last thing to do is to hook up the Add Tab MenuItem to the `AddTabCommand`.

```xml
<MenuItem Header="New _Tab" Command="{Binding AddTabCommand}" />
```

Excellent! It's all working very well. This is all about MVVM in its simplest way. You can see how easy it is to have everything separated and implement a very loosely coupled application. MVVM lets you unit test your software and change the views very easily.

You can notice that in this example, the ViewModel *does* know something about the View. It knows that the view contains a `TabControl` and adds `TabItems` to it. You must remove this dependency. I wanted this post to be as simple as it could be. However, that will be part of my upcoming posts. Now that you have set for yourself a base, explore the fantastic world of WPF and MVVM. Good Luck!
