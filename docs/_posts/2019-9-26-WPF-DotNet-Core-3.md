---
layout: post
title: Checking out WPF using .Net Core 3.0
---

# Checking out WPF using .Net Core 3.0

Microsoft announced the general availability of .Net Core 3.0 on the 25th of September 2019 (yesterday); along with open sourced WPF now running in .Net Core. I thought I will check this out.

The first step involves upgrading Visual Studio 2019 to version 16.3.0. This upgrade includes .Net Core 3.0 SDK, so there no need to install it separately.

## Step 1: Create New (and Blank) WPF .Net Core 3.0 Project

![new project](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/ab28ced189282bf54bf49741dc4908d0aaa4ab46/docs/images/wpf-dotnet-core-30/NewProject.png "Create New Project")

The source for this project is available on GitHub: [dennis-cs-chuah/test-wpf-dotnet-core-30](https://github.com/dennis-cs-chuah/test-wpf-dotnet-core-30)

The first thing I noticed was the WPF designer has gone AWOL. I mean the XAML editor is there, and when you run the application in the debugger, and change the XAML, the change is reflected in the application, but the designer is missing completely. Could it be that Microsoft shipped a faulty version?

After wasting many minutes looking for a solution, I stumbled upon a setting:

![settings](https://raw.githubusercontent.com/dennis-cs-chuah/csharp-blog/ab28ced189282bf54bf49741dc4908d0aaa4ab46/docs/images/wpf-dotnet-core-30/Settings.png "Visual Studio Preview Settings")

Checking the **Use previews of the .NET Core SDK** option and restarting Visual Studio restored the WPF designer! Someone obviously forgot to remove this setting! Oops!

## Step 2: Testdrive




