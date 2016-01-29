# The YUDU HTML Reader public API

## Contents

1. Table of Contents (You are here)
1. [Definition and Initialisation](#definition-and-initialisation)
1. [Using the API](#using-the-api)
1. [Functions](#functions)
    1. [Page turning](#page-turning)
        1. [Next page](#next-page)
        1. [Previous page](#previous-page)
        1. [Skip to arbitrary page](#skip-to-arbitrary-page)
    1. [Overlay control](#overlay-control)
        1. [Triggering launchable overlays](#triggering-launchable-overlays)

## Definition and Initialisation

The public API is defined as a Javascript hash of functions. It is defined and added to the window during the initialisation of the reader, but prior to any content being made available. In this way, it should always be available to on-page HTML content through the global variable.

```
window.yudu_readerApi = {
    nextPage: nextPage,
    previousPage: previousPage,
    goToPage: goToPage,
    triggerLightboxOnCurrentPage: triggerLightboxOnCurrentPage,
    triggerLightboxOnPage: triggerLightboxOnPage
};
```

A breakdown of the individual functions and their use is included below.

## Using the API

To use the functions defined by the above API, one need simply refer to the functions defined as its properties. For example, from the reader, calling `yudu_readerApi.goToPage(1)` will ask the reader to return to the first page of the edition. In particular, the `yudu_readerApi` has been added to the window's global namespace.

From on-page content, this is only complicated by the fact that, for security reasons amongst others, your content will be loaded in an [`iframe`](http://www.w3schools.com/tags/tag_iframe.asp). This means that the pages in your HTML package must refer first to `window.parent` in order to ensure the script knows to look for the reader's window and refer to its namespace. Example usages of each funtion will be included in their documentation below, assuming that the call is being made in a script on the main HTML page in your HTML package.

If your content has multiple layers of inner frames, you may need to recurse this call appropriately. The principle is that the reader's window should always be the parent of your content's effective "top" window. While possible, the use of `window.top` is not advised as this may have unexpected consequences should the reader itself be embedded in another page.

## Functions

### Page turning

Three page turning functions are currently made available to on-page HTML content. These are:

- [Next page](#next-page): `nextPage`
- [Previous page](#previous-page): `previousPage`
- [Skip to arbitrary page](#skip-to-arbitrary-page): `goToPage`

#### Next Page

Function reference: `nextPage`

Usage example: `window.parent.yudu_readerApi.nextPage()`

> Requests the reader turn to the next page (or double page in two-page mode) if there is one to turn to

This function emits an event to the reader, requesting it deal with a "next page" event. This will then be handled identically to a user clicking on the page-turn buttons on the toolbar, or swiping across a page without on-page HTML content.

#### Previous Page

Function reference: `previousPage`

Usage example: `window.parent.yudu_readerApi.previousPage()`

> Requests the reader turn to the previous page (or double page in two-page mode) if there is one to turn to

Similarly to the "next page" function, this function emits an event to the reader, requesting it deal with a "previous page" event. This will then be handled identically to a user clicking on the page-turn buttons on the toolbar, or swiping across a page without on-page HTML content.

#### Skip to arbitrary page

Function reference: `goToPage`

Usage example: `window.parent.yudu_readerApi.goToPage(pageNumber)`

> Requests the reader turn to the page number specified, starting from page 1.
> If the page given is not in the current range of pages, nothing will happen.

Parameters:

1. `pageNumber`: the page number to turn to, expects an integer

This function emits a special event to the reader, requesting it skip to an arbitrary page in the edition. Unlike changing the page's location, the reader then handles this on-page without requiring a reload.

Nothing will happen if any of the following conditions are met:

    - the page number specified is currently visible
    - the page number specified is less than 0
    - the page number specified is 0 but the edition contains no introduction page
    - the page number specified is greater than that of the last page in the edition

Any introduction page will always be available at page number 0. The first page of the edition (not including the introduction page) will always be available at page number 1. Page numbers required by this function match those added to the query string when the reader changes page - so if you are uncertain what page number you need to skip to, it may help to preview the edition without your HTML content.

### Overlay control

The following functions enable certain overlays to be triggered programmatically. This may be useful if, for example, you have a full-page HTML overlay that requires interaction, and hence would otherwise block interaction with other overlays on that page.

Note that due to current technical limitations, the reader cannot uniquely determine between two overlays of the same type on any given page programmatically. As such, when searching a page for an overlay, it will prefer caution, and will skip the page if it finds no overlays of that type, or if it finds more than one overlay of that type. This is expected behaviour, but may cause unexpected results if not planned for appropriately.

#### Triggering launchable overlays

Function reference: `triggerLightboxOnCurrentPage`

Usage example: `window.parent.yudu_readerApi.triggerLightboxOnCurrentPage()`

> Requests the reader open the lightbox associated with a lightbox overlay on the reader's current page
> This will only work if there is exactly one lightbox overlay on the page
> In two-page mode, this scans all currently visible pages starting with the left-hand page, triggering the
>  lightbox of the first page with exactly one lightbox that it encounters. For precise control of which
>  lightbox is opened, use `triggerLightboxOnPage`

This function attempts to open the lightbox associated with a launchable overlay on the currently visible page(s).

As noted above, a page will only be recognised as having an overlay available if it has exactly one. This is particularly pertinant when the reader is in two-page or "two-up" mode. In this case, the reader starts with the left-most page of the current spread, and continues to check all pages working left-to-right until it finds a page with exactly one overlay. It does not distinguish between pages with no overlays and pages with multiple overlays. This is most useful for editions with only one launchable overlay per double-page spread. For more precise control the following function may be required.

Function reference: `triggerLightboxOnPage`

Usage example: `window.parent.yudu_readerApi.triggerLightboxOnPage(pageNumber)`

> Requests the reader open the lightbox associated with a lightbox overlay on the specified page
> This will only work if there is exactly one lightbox overlay on the page

Parameters:

1. `pageNumber`: the page to check for a launchable overlay

This function, like the previous, attempts to open the lightbox associated with a launchable overlay. The difference this function provides is that the overlay may be located on any page in the edition, provided it is the only launchable overlay on that page. By specifying the page number, this function requests page information from the reader, and if it finds exactly one overlay on the specified page, it sends a request to the reader to skip to that page, and additionally attempts to open the lightbox.

The page numbering follows the same rules as that specified in the "[skip to arbitrary page](#skip-to-arbitrary-page)" function description.
