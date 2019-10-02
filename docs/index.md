---
layout: post
title: Checking out WPF using .Net Core 3.0
---

Microsoft announced the general availability of .Net Core 3.0 on the 25th of September 2019; along with open sourced WPF now running in .Net Core.

Before we can check this out, Visual Studio 2019 needed to be upgraded to version 16.3.0. This upgrade also includes the .Net Core 3.0 SDK, so there is no need to install it separately.

## Step 1: Create New (and Blank) WPF .Net Core 3.0 Project

![new project](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/ab28ced189282bf54bf49741dc4908d0aaa4ab46/docs/images/wpf-dotnet-core-30/NewProject.png "Create New Project")

The source for this project is available on GitHub: [dennis-cs-chuah/test-wpf-dotnet-core-30](https://github.com/dennis-cs-chuah/test-wpf-dotnet-core-30)

The first thing I noticed was the WPF designer had gone AWOL. The XAML text editor was there and [Hot Reload](https://docs.microsoft.com/en-us/visualstudio/debugger/xaml-hot-reload?view=vs-2019) worked, but the designer was missing completely. Could it be that Microsoft shipped a faulty version?

After spending some time looking for a solution, I stumbled upon a setting:

![settings](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/ab28ced189282bf54bf49741dc4908d0aaa4ab46/docs/images/wpf-dotnet-core-30/Settings.png "Visual Studio Preview Settings")

Checking the **Use previews of the .NET Core SDK** option and restarting Visual Studio restored the WPF designer! Someone obviously forgot to remove this setting! Oops!

<div style="border: 1px solid brown">
<h3>Update</h3>

<p>It seems like Microsoft has realised their mistake and provided an update:</p>

<p>
<img title="Visual Studio Update" alt="vs update" src="https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/VSUpdate.png"/>
</p>

</div>


### Project file

The generated .Net Core project file is a lot smaller than the typical .Net Framework project files.

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>
</Project>
```

## Step 2: Test Drive

Lets take this beast for a test drive. One of the benefits touted with .Net Core 3.0 is the upgrade to C# 8.0. Indeed the Advanced Build Settings dialog in Visual Studio now reflects this change:

![advanced build settings](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/AdvancedBuildOptions.png "Advanced Build Settings")

See also: [Changes in C# 8.0](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8)

### C# 8.0 Nullable Reference Types

The biggest change touted in C# 8 is the option to turn on static Nullability check. This can be optionally turned on by project or by file. The idea is if you had a large project to migrate, you could do it one file at a time.

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

After turning it on, lets create a dummy class to try this out. This class will later be deleted.

```c#
public class TestNull {
    public static int GetLengthOfString (string stringValue) {
        return stringValue.Length;
    }

    public static void DoSomething () {
        int l = GetLengthOfString (null);
    }
}
```

In Visual Studio, the call to `GetLengthOfString` is underlined and a warning generated:

![null warning underline](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/NullCheck.png "Null warning underline")

![null warning](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/NullCheckWarning.png "Null warning")

With nullable turned on `string` is now not nullable. Calling it with a null value is an error (but the check was set to only generate a warning). To make string nullable, append a `?` after it, as in `string?`.

![check for null underline](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/CheckForNull.png "Check for null")

![check for null warning](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/CheckForNullWarning.png "Check for null warning")

Of course, we could change the logic of `GetLengthOfString` to return -1 if the string is null:

```c#
public class TestNull {
    public static int GetLengthOfString (string? stringValue) {
        return stringValue?.Length ?? -1;
    }

    public static void DoSomething () {
        int l = GetLengthOfString (null);
    }
}
```

Not terribly interesting code but nevertheless shows the power of static nullability check.

### C# 8.0 Switch Expression

Lets also look at another C# enhancement, Switch Expression. This method works out the [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) rule. Prior to version 8.0, it could be written as:

```c#
public void ShouldBeAlive (IEnumerable<Cell> neighbours) {
    int livingNeighbours = neighbours.Sum (neighbour => neighbour.IsAlive ? 1 : 0);
    switch (livingNeighbours) {
      2 when IsAlive:
      3:
        willBeAlive = true;
        break;
      default:
        willBeAlive = false;
        break;
    }
}
```

(Yes, this could be written as an `if` statement, but `switch` is used here to illistrate a point.) In C# 8.0, this could be simplified to:

```c#
public void ShouldBeAlive (IEnumerable<Cell> neighbours) {
    int livingNeighbours = neighbours.Sum (neighbour => neighbour.IsAlive ? 1 : 0);
    willBeAlive = livingNeighbours switch {
        2 when IsAlive => true,
        3 => true,
        _ => false
    };
}
```

Note that the `switch` comes after the variable and it returns a value. The discard character, `_`, acts as the `default` clause.


## Step 3: Putting it all together

### GUI

As this is just to check out WPF in .Net Core 3.0, lets build a really simple GUI.

![simple GUI](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/GUI.png "Simple GUI")

On the left, you get to specify the size of the grid and a **Start** button. On the right, there is a multiline `TextBlock` that display the game grid. Each living cell will be represented as a couple of hashes `##`.

### Model

Lets put the `ShouldBeAlive` method into a class, `Cell` with the following signature:

```c#
public class Cell {
    public Cell (bool isAlive);
    public bool IsAlive { get; }
    public void ShouldBeAlive (IEnumerable<Cell> neighbours);
    public void ApplyNextState ();
}
```

and a grid of cells:

```c#
public class GameGrid {
    public GameGrid (byte width, byte height);
    public void NextIteration ();
    public IEnumerable<IEnumerable<Cell>> GetCells ();
}
```

The source for this project is available on GitHub: [dennis-cs-chuah/test-wpf-dotnet-core-30](https://github.com/dennis-cs-chuah/test-wpf-dotnet-core-30)

### ViewModel

Create a ViewModel and bind GUI to it:

```c#
public class GameViewModel : INotifyPropertyChanged {
    public byte Width { get; set; }
    public byte Height { get; set; }
    public bool IsStarted { get; }
    public bool IsStopped { get; }
    public string Display { get; }
    public void Start (int intervalSeconds = 1);
    public void Stop ();
}
```

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="160"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>
    <StackPanel Grid.Column="0" Orientation="Vertical" Margin="4 4 6 4">
        <Label Content="Grid width"/>
        <TextBox Text="{Binding ViewModel.Width, Mode=TwoWay, FallbackValue=30}"/>
        <Label Content="Grid height"/>
        <TextBox Text="{Binding ViewModel.Height, Mode=TwoWay, FallbackValue=30}"/>
        <Button Content="Start" Margin="20 20 20 0" Click="StartGame" IsEnabled="{Binding ViewModel.IsStopped, Mode=OneWay}"/>
        <Button Content="Stop" Margin="20 20 20 0" Click="StopGame" IsEnabled="{Binding ViewModel.IsStarted, Mode=OneWay}"/>
    </StackPanel>
    <ScrollViewer Margin="4" Grid.Column="1" HorizontalScrollBarVisibility="Auto" VerticalScrollBarVisibility="Auto">
        <Border BorderThickness="1" BorderBrush="Brown">
            <TextBlock Text="{Binding ViewModel.Display, Mode=OneWay}" TextWrapping="Wrap" FontFamily="Courier New" />
        </Border>
    </ScrollViewer>
</Grid>
```

### Unit test

Write a few unit tests for the model and view model, eg.

```c#
private static readonly object[] TEST_CASES = new object[] {
    // Living cell, 2, or 3 neighbours -> continue to be alive
    new object[] { true, 0, false },
    new object[] { true, 1, false },
    new object[] { true, 2, true },
    new object[] { true, 3, true },
    new object[] { true, 4, false },
    new object[] { true, 5, false },
    new object[] { true, 6, false },
    new object[] { true, 7, false },

    // Dead cell, if exactly 3 neighbours -> will become alive (reproduction)
    new object[] { false, 0, false },
    new object[] { false, 1, false },
    new object[] { false, 2, false },
    new object[] { false, 3, true },
    new object[] { false, 4, false },
    new object[] { false, 5, false },
    new object[] { false, 6, false },
    new object[] { false, 7, false }
};

[TestCaseSource (nameof (TEST_CASES))]
public void TestNextState (bool isAlive, int livingNeighbours, bool expectedToBeAlive) {
    Cell testCell = new Cell (isAlive: isAlive);
    testCell.ShouldBeAlive (Enumerable.
        Range (0, livingNeighbours).
        Select (_ => new Cell (isAlive: true)));
    testCell.ApplyNextState ();
    Assert.AreEqual (expectedToBeAlive, testCell.IsAlive);
}
```

## Step 4 - Run it!

![sample run](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/master/docs/images/wpf-dotnet-core-30/Run.png "Sample run")

## Step 5 - Tweaking The Output

Doing a **Release** build resulted in a small EXE and DLL pair, approximately 180Kb in total size.

Doing a publish targeting win10-x64 resulted in an EXE / DLL pair of similar size.

### ReadyToRun

This option only work for self contained apps, so not surprisingly, adding ReadyToRun and publishing a self-contained app bloats up the file size to 150Mb!

Turn this option on by adding `<PublishReadyToRun>true</PublishReadyToRun>` in the project file.

### Assembly linking

Another option for self contained apps is *Assembly linking*, essentially a smart linker that only includes libraries that are referenced by the app. Using this setting, the self-contained app size reduced to 85Mb - still a hefty app for what it is doing.

Turn this option on by adding `<PublishTrimmed>true</PublishTrimmed>` in the project file.

### Single file executables

This is a more useful feature. Turn this option on either by adding `<PublishSingleFile>true</PublishSingleFile>` in the project file or through `dotnet publish /p:PublishSingleFile=true`. This option requires the runtime to be also specified. Adding the option to the project file means only one runtime can be targeted. Using the `dotnet publish` method, different executables could be created for each targeted runtime.

Publishing to win10-x64 created a single EXE file of around 180Kb.

## Rounding up



