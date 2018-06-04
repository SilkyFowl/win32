---
title: How to Create an Internet Explorer-Style Toolbar
description: One of the key user interface features of Windows Internet Explorer is the toolbar. It not only gives users access to a wide array of features, it also allows users to customize its layout according to their personal preferences.
ms.assetid: d24969ec-4dea-44c6-b045-5611de8f1cce
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# How to Create an Internet Explorer-Style Toolbar

One of the key user interface features of Windows Internet Explorer is the toolbar. It not only gives users access to a wide array of features, it also allows users to customize its layout according to their personal preferences.

The following screen shot shows the Internet Explorer toolbar, and highlights some of the key features.

![screen shot of the windows internet explorer toolbar, with labels for eight features](images/howto1a.jpg)

This toolbar essentially consists of a rebar control with four bands: three toolbars and a menu bar. Because it is implemented with the common controls API, developers can create toolbars with any or all of its features. This topic discusses the essential features of the Internet Explorer toolbar and how to implement them in your application.

-   [The Rebar Control](#the-rebar-control)
    -   [Implementing the Rebar Control](#implementing-the-rebar-control)
-   [The Toolbars](#how-to-create-an-internet-explorer-style-toolbar)
    -   [Drop-down Buttons](#drop-down-buttons)
    -   [List-Style Buttons](#list-style-buttons)
    -   [Chevrons](#handling-chevrons)
    -   [Hot-Tracking](#hot-tracking)
-   [Related topics](#related-topics)

## The Rebar Control

The underlying structure of the Internet Explorer toolbar is provided by a [rebar control](rebar-controls.md). This control provides a way for users to customize the arrangement of a collection of tools. Each rebar contains one or more *bands*, which are typically long, narrow rectangles that contain a child window, commonly a toolbar control.

The rebar control displays its bands in a rectangular area, typically at the top of the window. This rectangle is subdivided into one or more strips that are the height of a band. Each band can be on a separate strip, or multiple bands can be placed on the same strip.

A rebar control provides users with two ways to arrange their tools:

-   Each band usually has a *gripper* at its left edge. Grippers are used when two or more bands on a single strip exceed the width of the window. By dragging the gripper to the left or right, users can control how much space is allocated to each band.
-   Users can move the bands within the rebar's display rectangle by dragging and dropping. The rebar control then changes the display to accommodate the new arrangement of bands. If all the bands are removed from a strip, the height of the rebar will be reduced, enlarging the viewing area.

An application can add or remove bands as needed. Typically, applications enable users to select which bands they want to have displayed through the View menu or a shortcut menu.

If the combined width of the bands on a strip exceeds the width of the window, the rebar control will adjust their widths as needed. Some of the tools might be covered by the adjacent band.

[Version 5.80](common-control-versions.md) of the common controls provides a way to make tools that have been covered by another band accessible to the user. If you set the RBBS\_USECHEVRON flag in the **fStyle** member of the band's [**REBARBANDINFO**](/windows/desktop/api/Commctrl/ns-commctrl-tagrebarbandinfoa) structure, a *chevron* will be displayed for toolbars that have been covered. When a user clicks the chevron, a menu is displayed that allows him or her to use the hidden tools. The following screen shot from Microsoft Internet Explorer 6 shows the menu that is displayed when part of the standard toolbar is covered.

![screen shot that shows the menu displayed by clicking the chevron](images/howto2.jpg)

Since each band contains a control, you can provide additional flexibility through the control's API. For example, you can implement toolbar customization to enable the user to add, move, or delete buttons on a toolbar.

### Implementing the Rebar Control

Most of the features of the Internet Explorer toolbar are actually implemented in the individual bands. The implementation of the rebar control itself is straightforward and is listed below.

1.  Create the rebar control with [**CreateWindowEx**](https://msdn.microsoft.com/library/windows/desktop/ms632680). Set *dwExStyle* to [**WS\_EX\_TOOLWINDOW**](https://msdn.microsoft.com/library/windows/desktop/ff700543#ws-ex-toolwindow) and *lpClassName* to [**REBARCLASSNAME**](https://www.bing.com/search?q=**REBARCLASSNAME**). Internet Explorer uses the following window styles:

    -   [**RBS\_BANDBORDERS**](https://www.bing.com/search?q=**RBS\_BANDBORDERS**)
    -   [**RBS\_DBLCLKTOGGLE**](https://www.bing.com/search?q=**RBS\_DBLCLKTOGGLE**)
    -   [**RBS\_REGISTERDROP**](https://www.bing.com/search?q=**RBS\_REGISTERDROP**)
    -   [**RBS\_VARHEIGHT**](https://www.bing.com/search?q=**RBS\_VARHEIGHT**)
    -   [**CCS\_NODIVIDER**](https://www.bing.com/search?q=**CCS\_NODIVIDER**)
    -   [**CCS\_NOPARENTALIGN**](https://www.bing.com/search?q=**CCS\_NOPARENTALIGN**)
    -   [**WS\_BORDER**](https://msdn.microsoft.com/library/windows/desktop/ms632600#ws-border)
    -   [**WS\_CHILD**](https://msdn.microsoft.com/library/windows/desktop/ms632600#ws-child)
    -   [**WS\_CLIPCHILDREN**](https://msdn.microsoft.com/library/windows/desktop/ms632600#ws-clipchildren)
    -   [**WS\_CLIPSIBLINGS**](https://msdn.microsoft.com/library/windows/desktop/ms632600#ws-clipsiblings)
    -   [**WS\_VISIBLE**](https://msdn.microsoft.com/library/windows/desktop/ms632600#ws-visible)

    Set the other parameters as appropriate for your application.

2.  Create a control with [**CreateWindowEx**](https://msdn.microsoft.com/library/windows/desktop/ms632680) or a specialized control creation function such as [**CreateToolbarEx**](/windows/desktop/api/Commctrl/nf-commctrl-createtoolbarex).
3.  Initialize a band for the control by filling in the members of [**REBARBANDINFO**](/windows/desktop/api/Commctrl/ns-commctrl-tagrebarbandinfoa). Include the RBBS\_USECHEVRON style with the **fStyle** member to enable chevrons.
4.  Add the band to the rebar control with an [**RB\_INSERTBAND**](rb-insertband.md) message.
5.  Repeat steps 2-4 for the remaining bands.
6.  Implement handlers for the rebar notifications. In particular, you will need to handle [RBN\_CHEVRONPUSHED](rbn-chevronpushed.md) to display a drop-down menu when a chevron is clicked. For further information, see [Handling Chevrons](#handling-chevrons).

The grippers are included by default. To omit the gripper for a band, set the RBBS\_NOGRIPPER flag in the **fStyle** member of the band's [**REBARBANDINFO**](/windows/desktop/api/Commctrl/ns-commctrl-tagrebarbandinfoa) structure. For further information on implementing rebar controls, see [About Rebar Controls](rebar-controls.md).

### Handling Chevrons

When a user clicks a chevron, the rebar control sends your application an [RBN\_CHEVRONPUSHED](rbn-chevronpushed.md) notification. The [**NMREBARCHEVRON**](/windows/desktop/api/Commctrl/ns-commctrl-tagnmrebarchevron) structure that is passed with the notification contains the band's identifier and a [**RECT**](https://msdn.microsoft.com/library/windows/desktop/dd162897) structure with the rectangle that is occupied by the chevron. Your handler must determine which buttons are hidden and display the associated commands on a pop-up menu.

The following procedure outlines how to handle an [RBN\_CHEVRONPUSHED](rbn-chevronpushed.md) notification:

1.  Retrieve the current bounding rectangle for the selected band by sending the rebar control an [**RB\_GETRECT**](rb-getrect.md) message.
2.  Retrieve the total number of buttons by sending the band's toolbar control a [**TB\_BUTTONCOUNT**](tb-buttoncount.md) message.
3.  Starting from the leftmost button, retrieve the button's bounding rectangle by sending the toolbar control a [**TB\_GETITEMRECT**](tb-getitemrect.md) message.
4.  Pass the band and button rectangles to the [**IntersectRect**](https://msdn.microsoft.com/library/windows/desktop/dd145001) function. This function will return a [**RECT**](https://msdn.microsoft.com/library/windows/desktop/dd162897) structure that corresponds to the visible portion of the button.
5.  Pass the button rectangle and the rectangle for the visible portion of the button to the [**EqualRect**](https://msdn.microsoft.com/library/windows/desktop/dd162699) function.
6.  If [**EqualRect**](https://msdn.microsoft.com/library/windows/desktop/dd162699) returns **TRUE**, the entire button is visible. Repeat steps 3-5 for the next button on the toolbar. If **EqualRect** returns **FALSE**, the button is at least partially hidden and all remaining buttons will be completely hidden. Continue to the next step.
7.  Create a pop-up menu with items for each of the hidden buttons.
8.  Display the pop-up menu by using the [**TrackPopupMenu**](https://msdn.microsoft.com/library/windows/desktop/ms648002) function. Use the chevron rectangle that was passed with the [RBN\_CHEVRONPUSHED](rbn-chevronpushed.md) notification to position the menu. The menu should be immediately below the chevron, with the left edges aligned.
9.  Handle the menu commands.

## The Toolbars

Most of the complexity of the Internet Explorer toolbar lies in the implementation of controls that make up the rebar bands. Internet Explorer commonly displays four bands:

-   The menu bar
-   The standard toolbar
-   The links toolbar
-   The address toolbar

All of these bands, including the menu bar, actually hold toolbar controls. This section discusses the implementation of the standard and links toolbars. The menu bar is somewhat more complicated and is discussed separately in [How to Create an Internet Explorer-Style Menu Bar](cc-faq-iemenubar.md).

The basic procedures for implementing toolbar controls are discussed in [About Toolbar Controls](toolbar-controls-overview.md). This section focuses on some of the newer toolbar features that are used by Internet Explorer to increase the usability of the control.

-   [Drop-down Buttons](#drop-down-buttons)
-   [List-Style Buttons](#list-style-buttons)
-   [Chevrons](#handling-chevrons)
-   [Hot-Tracking](#hot-tracking)

### Drop-down Buttons

Drop-down buttons support multiple commands. When the user clicks a drop-down button, the button displays a pop-up menu instead of launching a command. The user launches a command by selecting it from the menu. The following screen shot shows a drop-down button and menu from the Internet Explorer standard toolbar.

![screen shot showing the mail drop-down menu](images/howto3.jpg)

Drop-down functionality can be added to any button style by adding a style flag to the **fStyle** member of the button's [**TBBUTTON**](/windows/desktop/api/Commctrl/ns-commctrl-_tbbutton) structure. There are three styles of drop-down button, all of which are used by Internet Explorer:

-   Plain drop-down buttons have the [**BTNS\_DROPDOWN**](https://www.bing.com/search?q=**BTNS\_DROPDOWN**) style. They look like normal buttons, but they display a menu when clicked instead of launching a command.
-   Simple drop-down arrow buttons have the [**BTNS\_WHOLEDROPDOWN**](https://www.bing.com/search?q=**BTNS\_WHOLEDROPDOWN**) style. They have an arrow displayed next to the button image or text. Other than the difference in appearance, they are identical to plain drop-down buttons. The Mail button used as the example in the preceding illustration is a drop-down arrow button.
-   Drop-down arrow buttons that add the [**TBSTYLE\_EX\_DRAWDDARROWS**](https://www.bing.com/search?q=**TBSTYLE\_EX\_DRAWDDARROWS**) extended style to [**BTNS\_DROPDOWN**](https://www.bing.com/search?q=**BTNS\_DROPDOWN**) have an arrow that is separated from the text or image. This button style combines the functionality of drop-down and standard buttons. If the user clicks the arrow, a menu is displayed and the user can choose from several commands. If the user clicks the adjacent button, it launches a default command. The following screen shot shows the Internet Explorer **Back** button, which uses a separated arrow.

    ![screen shot that shows the menu of the back split button](images/howto4.jpg)

When the user clicks a drop-down button with either the plain or simple arrow styles, the toolbar control sends your application a [TBN\_DROPDOWN](tbn-dropdown.md) notification. When your application receives this message, it is responsible for creating and displaying the menu, and for handling the selected command.

When the user clicks a separated arrow, the toolbar control sends your application a [TBN\_DROPDOWN](tbn-dropdown.md) notification. Your application should handle it the same way as it handles the other two types of drop-down buttons. If the user clicks the main button, your application receives a [**WM\_COMMAND**](https://msdn.microsoft.com/library/windows/desktop/ms647591) message with the button's command ID, just as if it were a standard button. Applications typically respond by launching the top command in the drop-down menu, but you are free to respond in any suitable way.

### List-Style Buttons

With standard buttons, if you add text, it is displayed below the bitmap. The following screen shot shows the Internet Explorer **Search**and **Favorites** buttons with standard button text.

![screen shot showing the search and favorites toolbar with standard buttons](images/howto6.jpg)

Microsoft Internet Explorer 5 and later versions use the [**TBSTYLE\_LIST**](https://www.bing.com/search?q=**TBSTYLE\_LIST**) style. The text is to the right of the bitmap, reducing the height of the button and enlarging the viewing region. The following illustration shows the Internet Explorer 6 **Search** and **Favorites** buttons with the **TBSTYLE\_LIST** style.

![screen shot showing the search and favorites toolbar using list-style buttons](images/howto5.jpg)

### Chevrons

When the user rearranges the bands in the rebar control, part of a toolbar might be covered up. If the band was created with the RBBS\_USECHEVRON style, the rebar control will display a chevron at the right edge of the toolbar. The user clicks the chevron to display a menu with the hidden tools.

### Hot-Tracking

When hot-tracking is enabled, a button becomes *hot* when the cursor is over it. The hot button is normally distinguished from the other buttons on the toolbar by a distinctive image. By default, a hot button appears to be raised above the rest of the toolbar. When a new button becomes hot, your application receives a [TBN\_HOTITEMCHANGE](tbn-hotitemchange.md) notification. The following illustration shows the Internet Explorer 5 **Search** and **Favorites** buttons, with a hot **Search** button. In addition to having a raised appearance, the button's gray bitmap has been replaced with a colored one.

![screen shot showing the search and favorites buttons, with a hot search button](images/howto7.jpg)

To enable hot-tracking, create a toolbar control with either the [**TBSTYLE\_FLAT**](https://www.bing.com/search?q=**TBSTYLE\_FLAT**) or [**TBSTYLE\_LIST**](https://www.bing.com/search?q=**TBSTYLE\_LIST**) style. These are referred to as *flat* toolbars because the individual buttons are not ordinarily highlighted in any way. The bitmaps are simply displayed next to each other. They take on a button-like appearance only when they are hot. These two styles are also transparent, which means that the background of the icons will be the color of the underlying client window.

To have a different bitmap displayed when the button is hot, create a second [image list](image-lists.md) that contains hot images for all the buttons on the toolbar. The size and order of these images should be the same as in the default image list. Send the toolbar control a [**TB\_SETHOTIMAGELIST**](tb-sethotimagelist.md) message to set the hot image list.

## Related topics

<dl> <dt>

[How to Create an Internet Explorer-Style Menu Bar](cc-faq-iemenubar.md)
</dt> </dl>

 

 



