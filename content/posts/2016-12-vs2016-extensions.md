---
title: "Top Visual Studio 2015 Extensions I Use"
date: 2016-12-31
dateUpdated: Last Modified
permalink: /posts/vs2016-extensions/
tags:
  - Tools
layout: layouts/post.njk
---

Rich IDEs are a great asset to enhance productivity in writing and reading code.<!--excerpt-->

Visual Studio 2015 as a rich IDE helps a lot when navigating through code and files, refactoring, smart searching classes/methods/properties and much more. But still there are some useful or improved functionalities not baked in, that can be easily found as extensions.<!--excerpt-->

**Here is the list of extensions that I use every day:**

- [Refactoring Essentials](http://vsrefactoringessentials.com/) - Rich free refactoring tool for C#.
- [Add New File](https://visualstudiogallery.msdn.microsoft.com/3f820e99-6c0d-41db-aa74-a18d9623b1f3) - The fastest and easiest way to add new files to any project.
- [Easy Motion](https://github.com/jaredpar/EasyMotion) - A vim EasyMotion clone for Visual Studio. Instead of moving your hands to the arrow keys or even worse, grabbing the mouse, simple initiate an easy motion search by pressing ``Shift + Control + ;``. (*NOTE: I changed my shortcut to be bound to ``Ctrl + Shift + F`` as I can trigger it only using the left hand.*)
- [Indent Guides ](https://marketplace.visualstudio.com/items?itemName=SteveDowerMSFT.IndentGuides) - Adds vertical lines at each indent level. It can even add a vertical line to a certain character length (e.g. on 100 characters length so you know visually how long is your line of code).
- [Configuration Transform](https://marketplace.visualstudio.com/items?itemName=GolanAvraham.ConfigurationTransform) - Automatically transform web.config, app.config or any other config during the build process. Once the transformation is set, it will run on other build machines without the extension.
- [Hide Main Menu](https://marketplace.visualstudio.com/items?itemName=MatthewJohnsonMSFT.HideMainMenu) - Automatically hides the Visual Studio main menu when not in use. To show when hidden, press ``ALT`` key.
- [Rename Visual Studio Window Title ](https://marketplace.visualstudio.com/items?itemName=mayerwin.RenameVisualStudioWindowTitle) - This lightweight extension allows changing the window title of Visual Studio to include a folder tree with a configurable distance from the solution/project file. (*NOTE: The title template I use is:``[solutionName] ([configurationName]) - [documentParentPath:2:0]``*)
- [Open Command Line](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.OpenCommandLine) - Opens a command line at the root of the project by pressing ``ALT + Space``. Useful when you need to execute commands from CMD at the current project's directory.
- [ResXManager](https://marketplace.visualstudio.com/items?itemName=TomEnglert.ResXManager) - Manage localization of all ResX-Based resources in one place. Shows all resources of a solution and lets you edit the strings and their localizations in a well-arranged data grid.

*NOTE: I do evaluate performance hit on Visual Studio itself before I use or recommend an extension. There are some other good extensions too that I don't use because they slow down Visual Studio a lot. Fast and responsive IDE has higher priority than new or improved functionalities on my machine :)*

What extensions do *you* use, dear reader?
