# Window management

A PWA outside of the browser manages its own window. Learn about the APIs
and capabilities for managing a window within the operating system.

Running in your own window, managed by your PWA, has all the advantages and
responsibilities of any window in that operating system, such as:

- The ability to move and resize the window in multi-window operating systems, such as Windows or ChromeOS.
- Sharing the screen with other app windows, as in iPadOS split mode or Android split-screen mode.
- Appearing in docks, taskbars, and in the alt-tab menu on desktop, and multi-task window lists on mobile devices.
- The ability to minimize, move the window across screens and desktops, and close it at any time.

### Move and resize the window

Your PWA window can be of any size and positioned anywhere on the screen on
desktop operating systems. By default, when the user opens the PWA for the first
time after installation, the PWA gets a default window size of a percentage of
the current screen, with a maximum resolution of 1920x1080 positioned at the
top-left corner of the screen.

The user can move and resize the window, and the browser will remember the last
preference. The next time the user opens the app, the window will retain the
size and position from the previous usage.

There is no way to define your PWA's preferred size and position within the
manifest. You can only reposition and resize the window using the JavaScript
API. From your code, you can move and resize your own PWA window using the
[`moveTo(x, y)`](https://developer.mozilla.org/docs/Web/API/Window/moveTo) and
[`resizeTo(x, y)`](https://developer.mozilla.org/docs/Web/API/Window/resizeTo)
functions of the `window` object.

For example, you can resize and move your PWA window when the PWA loads using:  

    document.addEventListener("DOMContentLoaded", event => {
       // we can move only if we are not in a browser's tab
       isBrowser = matchMedia("(display-mode: browser)").matches;
       if (!isBrowser) {
          window.moveTo(16, 16);
          window.resizeTo(800, 600);
       }
    });

You can query the current screen size and position using the `window.screen`
object; you can detect when the window is resized using the `resize` event from
the `window` object. There is no event for capturing the window move, so your
option is to query the position frequently.

Instead of moving and resizing the window absolutely, you can move relatively
and resize using `moveBy()` and `resizeBy()`.
| **Note:** On mobile devices, moving or resizing your window won't make any changes on the screen.

### Browse other sites

If you want to send the user to an external site that is out-of-scope for your
PWA, you can do so with a standard `<a href>` HTML element. Use `location.href`
or open a new window on compatible platforms.

When you visit a URL that is out of the
[scope of your manifest](https://web.dev/learn/pwa/web-app-manifest#recommended_fields), the
browser engine of your PWA renders an in-app browser within the context of
your window.

![An in-app browser on a desktop PWA when browsing a URL that is out-of-scope.](https://web.dev/static/learn/pwa/windows/image/an-app-browser-a-deskto-0c18838026b82.png)

Some features of the in-app browsers are:

- They appear on top of your content.
- They have a static address bar showing the current origin, the window's title, and a menu. Typically, they're themed with the `theme_color` of your manifest.
- From the contextual menu, you can open that URL in the browser.
- Users can close the browser or go back.

While the in-app browser is on the screen, your PWA is waiting in the
background, as if another window is obscuring it.
| **Note:** On iOS and iPadOS, the in-app browser uses `SafariViewController`, a Safari-rendering engine isolated from your PWA storage. Therefore, if the user has a session in Safari, it won't appear in your PWA, but it does appear within an in-app browser in your PWA.
![](https://web.dev/static/learn/pwa/windows/image/an-app-browser-an-iphon-dddc327f709aa.png) An in-app browser on an iPhone when browsing a URL that is out-of-scope within a standalone PWA. ![](https://web.dev/static/learn/pwa/windows/image/an-app-browser-android-3d73f1c909f0b.png) An in-app browser on Android when browsing a URL that is out-of-scope within a standalone PWA.

#### Authorization flows

Many web authentication and authorization flows require redirecting the user to
a different URL on a different origin to acquire a token that returns to your
PWA's origin, such as with [OAuth 2.0](https://oauth.net/2/).

In these cases, the in-app browser follows this process:

1. The user opens your PWA and clicks login.
2. Your PWA redirects the user to a URL that is out of the scope of the PWA so that the rendering engine opens an in-app browser within your PWA.
3. The user can cancel the in-app browser and go back to your PWA at any time.
4. The user logs into the in-app browser. The authentication server redirects the user to your PWA origin, sending the token as an argument.
5. The in-app browser closes itself when it detects a URL that is part of the scope of the PWA.
6. The engine redirects the main PWA window navigation to the URL that the authentication server went to while in the in-app browser.
7. Your PWA gets the token, stores the token, and renders the PWA.

#### Force a browser's navigation

If you want to force open the browser with a URL and not an in-app browser, you
can use the `_blank` target of `<a href>` elements. This works only on desktop
PWAs. On mobile devices, there's no option to open a browser with a URL.  

    function openBrowser(url) {
        window.open("url", "_blank", "");
    }

### Open new windows

On desktop, users can open more than one window of the same PWA. Each window has
a different navigation for the same `start_url`, as if you were opening two
browser tabs of the same URL.
From the menu in the PWA, users can select **File** then **New window** . From
your PWA code, you can open a new window with the
[`open()`](https://developer.mozilla.org/docs/Web/API/Window/open) function.  

    function openNewWindow() {
        window.open("/", "new-window", "width=600,height=600");
    }

![The same installed PWA with several windows opened on a desktop operating system.](https://web.dev/static/learn/pwa/windows/image/the-same-installed-pwa-s-2e0275a3f5fce.png)

When you call `open()` within a PWA window on iOS or iPadOS, it returns `null`
and doesn't open a window. Opening new windows on Android creates a new in-app
browser for the URL, even if the URL is within the scope of your PWA, that
typically doesn't trigger an external browsing experience.
| **Caution:** The second argument of `open()` is the window's name. To always open a new window, you must use different string values for a name.
| **Tip:** The functions `open()`, `moveTo()`, and `resizeTo()` work without the `window.` prefix, because `window.` is a global JavaScript object. Instead, you can call `moveTo(0, 0)`, which may be easier to understand.

### Window title

The `<title>` element was primarily used for SEO purposes as the space within a browser tab is limited. When you move from the browser to your window in a PWA, all the title bar space is available to you.

You can define the contents of the title bar:

- Statically in your HTML `<title>` element.
- Dynamically changing the `document.title` string property at any time.

On desktop PWAs, the title is essential, and it's used in the title bar of the
window and sometimes in the task manager or multi-task selection. If you have a
single-page application, you may want to update your title on every route.
| **Note:** To reduce phishing, some desktop browsers may include additional measures on the window's title. For example, if you don't use your app's name in the title, the browser may add a prefix with your PWA's name on it. Other browsers may render the current origin in the title bar for a few seconds when you change the title, to highlight where the user navigated.

### Tabbed mode

*Tabbed mode* is an experimental capability that lets your PWA have a tab-based
design, similar to a web browser. In this case, the user can open several tabs
in the same PWA, but all tied together in the same operating system window.

You can read more about this experimental capability at
[Tabbed application mode for PWA](https://developer.chrome.com/docs/capabilities/tabbed-application-mode).
| **Note:** You'll learn more about experimental capabilities in the [Experimental chapter](https://web.dev/learn/pwa/experimental).

### Window controls overlay

We've mentioned that you can change the window's title by defining the value of
the `<title>` element or the `document.title` property. But it's always a string
value. What if we could design the title bar with HTML, CSS, and images?
Window Controls Overlay, an experimental capability in Microsoft Edge and Google
Chrome for desktop PWAs, may help.
| **Note:** Tabbed mode and window controls overlay capabilities define new values for the manifest's `display` member. To ensure compatibility with all devices, the manifest group lets you replace the [fallback `display` chain](https://web.dev/learn/pwa/app-design#display_modes), so you can specify what `display` value to use if your first option is unavailable. Read more about [display modes](https://developer.chrome.com/docs/capabilities/display-override).

You can read more about this capability at [Customize the window controls overlay of your PWA's title bar](https://web.dev/articles/window-controls-overlay).

![With window controls overlay, you can render content in the title bar.](https://web.dev/static/learn/pwa/windows/image/with-window-controls-over-870f687efe854.png)

## Window management

With multiple screens, users want to use all the space available to them. For
example:

- Multi-window graphics editors, such as Gimp, can place various editing tools in accurately positioned windows.
- Virtual trading desks can show market trends in multiple windows any of which can be viewed in fullscreen mode.
- Slideshow apps can show speaker notes on the internal primary screen and the presentation on an external projector.

The [Window Management API](https://developer.chrome.com/docs/capabilities/web-apis/window-management)
allows PWAs to do just that and more.

### Getting screen details

The [Window Management API](https://developer.mozilla.org/docs/Web/API/Window_Management_API) adds a new method, `window.getScreenDetails()`, that returns an object with screens as an immutable array of attached screens. There's also a live object accessible from `ScreenDetails.currentScreen`, corresponding to the current `window.screen`.
| **Note:** On some browsers, calling `window.getScreenDetails()` requires the user to grant permission to your PWAs.

The returned object also fires a [`screenschange` event](https://developer.chrome.com/articles/window-management#the-screenschange-event) when the `screens` array changes. (This does not happen when attributes on individual screens are changed.) Individual screens, either `window.screen` or a screen in the `screens` array, also fire a `change` event when their attributes change.  

    // Request an object with a screen objects
    const screenDetails = await window.getScreenDetails();
    screenDetails.screens[0].isPrimary;  // e.g. true
    screenDetails.screens[0].isInternal;  // e.g. true
    screenDetails.screens[0].label;  // e.g. 'Samsung Electric Company 28"'

    // Access the live object corresponding to the current `window.screen`.
    // The object is updated on cross-screen window placements or device changes.
    screenDetails.currentScreen;
    screenDetails.addEventListener('screenschange', function() {
     // NOTE: Does not fire on changes to attributes of individual screens.
      const screenCount = screenDetails.screens.length;
      const currentScreen screenDetails.currentScreen.id;
    });

If the user or the operating system moves your PWA's window from one screen to
another, a [`currentscreenchange` event](https://developer.chrome.com/articles/window-management#the-currentscreenchange-event) is also fired from the screen
details object.
| **Note:** With the [Presentation API](https://developer.mozilla.org/docs/Web/API/Presentation_API), you can display web content through devices such as projectors and network-connected televisions.

## Screen wake lock

Imagine this. You're in the kitchen following a recipe on your tablet. You've just finished prepping your ingredients. Your hands are a mess, and you turn back to your device to read the next step. Disaster! The screen's gone black! The [Screen Wake Lock API](https://developer.mozilla.org/docs/Web/API/Screen_Wake_Lock_API) is here for you and lets a PWA prevent screens from dimming, sleeping, or locking, allowing users to stop, start, leave, and return without worry.  

    // Request a screen wake lock
    const wakeLock = await navigator.wakeLock.request();

    // Listen for wake lock release
    wakeLock.addEventListener('release', () => {
     console.log(`Screen Wake Lock released: ${wakeLock.released}`);
    });
    // Manually release the wake lock
    wakeLock.release();

## Virtual keyboard

Touch-based devices, such as phones and tablets, offer a virtual on-screen
keyboard so the user can type when form elements of your PWA are in focus.

With the [VirtualKeyboard API](https://web.dev/virtualkeyboard), your PWA can have
more control of the keyboard on compatible platforms using the
`navigator.virtualKeyboard` interface.

- Show and hide the virtual keyboard with `navigator.virtualKeyboard.show()` and `navigator.virtualKeyboard.hide()`.
- Tell the browser that you are closing the virtual keyboard yourself by setting `navigator.virtualKeyboard.overlaysContent` equal to `true`.
- Know when the keyboard appears and disappears with the `geometrychange` event.
- Set the virtual keyboard policy on editing host elements by setting `contenteditable` to `auto` or `manual`, with the `virtualkeyboardpolicy` HTML attribute. A policy lets you decide if you want the virtual keyboard to be handled automatically by the browser (`auto`) or handled by your script (`manual`).
- Use CSS environmental variables to get information about the virtual keyboard appearance, such as `keyboard-inset-height` and `keyboard-inset-top`.

Read more about this API in [Full control with the VirtualKeyboard API](https://web.dev/virtualkeyboard).