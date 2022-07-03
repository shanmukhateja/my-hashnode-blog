## A cross-platform GUI under 2 minutes with .NET via AvaloniaUI

In this post let's build a Hello World application using [AvaloniaUI](https://www.avaloniaui.net/) which is a C# library for building cross-platform desktop apps with XAML dialect.

## Requirements

1. .NET Core SDK  [Link](https://docs.microsoft.com/en-us/dotnet/core/install/windows)

2. Visual Studio 2019

## Getting Started

**Windows:**

1. Create "Avalonia MVVM Application" in Visual Studio: ![](https://www.avaloniaui.net/docs/quickstart/images/new-project-dialog.png)

2. Open `Views/MainWindow.cs.xaml`. This is XAML code and is developed by Microsoft and is heavily used in UWP apps (Windows 10 apps). XAML is used to create graphical UI in .NET Core.

3. Run the project. If everything is correctly setup, you will see a Window with Hello World text on it. Congratulations! It worked on first try.

** Linux/Mac:**

Let's see this in action on another platform. You can run this project in Linux/Mac platform after setting up .NET Core and Avalonia and running the command:

```
cd <project_dir>
$ dotnet run
``` 

You should now see the same window on a different platform!  Let's change the Button text. Open `ViewModels\MainWindowViewModel.cs` and update text of `Greeting` variable. Run the project and you should see the text has changed!

## Conclusion:

This is easy way to get started with cross-platform GUI apps via AvaloniaUI using .NET Core. If you wish to explore more, head over to [Avalonia Docs](https://www.avaloniaui.net/docs/quickstart/) 

Leave a comment or @ me on [Twitter](https://twitter.com/shanmukhateja94) if you need help. I plan on writing more articles covering this if there's interest. 

Happy New Year 2021 in advance folks :)

## Credits

Pic credits: Avalonia Docs