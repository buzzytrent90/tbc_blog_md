---
post_number: "1979"
title: "Dark Theme"
slug: "dark_theme"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'levels', 'revit-api', 'sheets', 'windows']
source_file: "1979_dark_theme.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1979_dark_theme.html"
---

### New Roadmaps, Dark Theme Possibility Looming
Happy New Year of the Rabbit, xīnnián hǎo, 新年好!
![Happy New Year of the Rabbit!](img/2023_year_of_the_rabbit_1.jpg "Happy New Year of the Rabbit!")
The Spring Festival is coming up this weekend, starting on Sunday, January 22, celebrating
the [Chinese New Year](https://en.wikipedia.org/wiki/Chinese_New_Year) and
another [Year of the Rabbit](https://en.wikipedia.org/wiki/Rabbit_(zodiac)).
In the lunar calendar, 2023 is a Water Rabbit Year.
The sign of the Rabbit is a symbol of longevity, peace, and prosperity in Chinese culture.
2023 is predicted to be a year of hope, especially after the long pandemic period.
Wishing all of us lots of health, energy and happiness in the new year!
In this new year, the Revit development team has another topic to share with us:
- [Dark theme possibility looming](#2)
- [Dark theme switching](#2.1)
- [Dark theme API](#2.2)
- [Dark theme icons](#2.3)
- [Code example: handling themed ribbon icons](#2.4)
- [Dark theme additional notes](#2.5)
- [Autodesk icon guidelines snapshot](#2.6)
- [Autodesk AEC Public Roadmaps](#3)
#### Dark Theme Possibility Looming
We recently announced internal thoughts
on [possibly converting the internal representation of Revit element ids from 32 to 64 bit in a future release of Revit](https://thebuildingcoder.typepad.com/blog/2022/11/64-bit-element-ids-maybe.html).
In a similar vein, here is another internal topic being pondered.
Please note the important safe harbor statement concerning these thoughts:
> Roadmaps are plans, not promises.
We’re as excited as you to see new functionality make it into the products, but the development, releases, and timing of any features or functionality remains at our sole discretion.
These updates should not be used to make purchasing decisions.
So, the possibility that I would like to present today concerns supporting the Dark Theme and how to handle it in a Revit add-in:
#### Dark Theme Switching
Setting the UI Active Theme will switch the appearance of the Ribbon between light grey and dark blue, with three options:
- Light
- Dark
- Use system setting
– Windows supports light and dark colour schemes.
If you choose this option, Revit will use the Windows colour scheme and switch to a matching theme accordingly.
Light:
![Dark theme – light](img/dark_theme_2.png "Dark theme – light")
Dark:
![Dark theme – dark](img/dark_theme_3.png "Dark theme – dark")
The UI Active Theme options can define other colour settings to override the default ones:
![Dark theme](img/dark_theme_1.png "Dark theme")
#### Dark Theme API
New properties and events may be added for dark theme support:
- ThemeChangedEventArgs – Arguments for the ThemeChanged event
- UIThemeManager.CurrentTheme – Allows you to set /get the overall theme for the Revit session
- UIThemeManager.FollowSystemColorTheme – Allows you to set /get if the overall theme follows operating system color theme
- UIThemeManager.CurrentCanvasTheme – Allows you to set/get a canvas theme for the current Revit session (as opposed to the default theme)
- ColorOption – Allows you to set/get the colours in the current canvas theme
#### Dark Theme Icons
Here are samples of the default dark theme ribbon background and button colour settings:
Light ribbon background:
![Dark theme – light ribbon background](img/dark_theme_4.png "Dark theme – light ribbon background")
Dark ribbon background:
![Dark theme – light ribbon background](img/dark_theme_5.png "Dark theme – light ribbon background")
Light ribbon buttons:
![Dark theme – light ribbon buttons](img/dark_theme_6.png "Dark theme – light ribbon buttons")
Dark ribbon buttons:
![Dark theme – dark ribbon buttons](img/dark_theme_7.png "Dark theme – dark ribbon buttons")
- Small button size: 16x16px
- Large button size: 32x32px
- Resolution: 96 DPI
- Icons
#### Code Example: Handling Themed Ribbon Icons

18. internal class TestRibbon : IExternalApplication
19. {
20. private PushButton m_ribbonBtn;
21. public Result OnStartup(UIControlledApplication application)
22. {
23. var ribbonPanel = application.CreateRibbonPanel("33900745-04F5-4CC2-9BAC-3230716E3A54", "Test");
24. var buttonData = new PushButtonData("Test", "Test", typeof(CmdEntry).Assembly.Location, typeof(CmdEntry).FullName);
25. buttonData.AvailabilityClassName = typeof(CmdEntry).FullName;
26. m_ribbonBtn = ribbonPanel.AddItem(buttonData) as PushButton;
27. updateImageByTheme();
28. application.ThemeChanged += ThemeChanged;
29. return Result.Succeeded;
30. }
31. private void setButtonImage(string pic, string largePic)
32. {
33. var assemblyLocation = typeof(TestRibbon).Assembly.Location;
34. var assemblyDirectory = Path.GetDirectoryName(assemblyLocation);
35. var imagePath = Path.Combine(assemblyDirectory, pic);
36. var largeImagePath = Path.Combine(assemblyDirectory, largePic);
37. if (File.Exists(imagePath))
38. m_ribbonBtn.Image = new System.Windows.Media.Imaging.BitmapImage(new Uri(imagePath));
39. if (File.Exists(largeImagePath))
40. m_ribbonBtn.LargeImage = new System.Windows.Media.Imaging.BitmapImage(new Uri(largeImagePath));
41. }
42. private void updateImageByTheme()
43. {
44. UITheme theme = UIThemeManager.CurrentTheme;
45. switch (theme)
46. {
47. case UITheme.Dark:
48. setButtonImage("dark.png", "darkLarge.png");
49. break;
50. case UITheme.Light:
51. setButtonImage("light.png", "lightLarge.png");
52. break;
53. }
54. }
55. private void ThemeChanged(object sender, Autodesk.Revit.UI.Events.ThemeChangedEventArgs e)
56. {
57. updateImageByTheme();
58. }
59. }

#### Dark Theme Additional Notes
Please note that only the 1st level UI supports the dark theme option.
#### Autodesk Icon Guidelines Snapshot
Prompted by [Luiz'](https://thebuildingcoder.typepad.com/blog/2023/01/dark-theme-possibility-looming.html#comment-6093239283)
and [Gábor's comments below](https://thebuildingcoder.typepad.com/blog/2023/01/dark-theme-possibility-looming.html#comment-6093873493),
I acquired and posted an up-to-date snapshot of the current state of the Autodesk icon guidelines including the images above:
- [Autodesk icon guidelines PDF](https://thebuildingcoder.typepad.com/icon/2023-01-20_icon_design_guidelines.pdf)
- [Zip file including badges and PNG instructiuons](https://thebuildingcoder.typepad.com/icon/2023-01-20_icon_design_guideline.zip)
Thank you for asking, Luiz and Gábor.
#### Autodesk AEC Public Roadmaps
For more exciting news on possible upcoming product enhancements, check out
the [Autodesk AEC Public Roadmaps](https://blogs.autodesk.com/revit/roadmap) with dozens of items in each of the sections:
- Revit – Architecture
- Revit – Structures
- Revit – MEP
- Dynamo Public Roadmap