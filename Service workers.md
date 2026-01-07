# Service workers

Users expect apps to start reliably on slow or flaky network connections, or
even offline. They expect the content they've most recently interacted with,
such as media tracks or tickets and itineraries, to be available and usable.
When a request isn't possible, they expect the app to tell them instead of
silently failing or crashing. And they want all of this to happen quickly. As
you can see in [Milliseconds make millions](https://web.dev/case-studies/milliseconds-make-millions),
even a 0.1 second improvement in load times can improve conversion by up to 10%.
Service workers are the tool that lets your Progressive Web App (PWA) live up to
your users' expectations.
![A service worker as a middleware proxy, running device-side, between your PWA and servers, which includes both your own servers and cross-domain servers.](https://web.dev/static/learn/pwa/service-workers/image/a-service-worker-a-middl-982e684894b75.png) A service worker acts as middleware between your PWA and the servers it interacts with.

When an app requests a resource covered by the service worker's scope, the
service worker intercepts the request and acts as a network proxy, even if the
user is offline. It can then decide if it should serve the resource from the
cache using the Cache Storage API, serve it from the network as if there were no
active service worker, or create it from a local algorithm. This lets you
provide a high-quality experience like that of a platform app, even when your
app is offline.
| **Tip:** Not all browsers support service workers. Even when they are supported, your service worker won't be available on first load or while it's waiting to activate. We recommend treating your service worker as optional and not requiring it for core features.

## Register a service worker

Before a service worker takes control of your page, it must be registered for
your PWA. That means the first time a user opens your PWA, all its network
requests go directly to your server because the service worker doesn't have
control of your pages yet.

After checking whether the browser supports the Service Worker API, your PWA can
register a service worker. After it loads, the service worker sets itself up
between your PWA and the network, intercepting requests and serving the
corresponding responses.  

    if ('serviceWorker' in navigator) {
       navigator.serviceWorker.register("/serviceworker.js");
    }

| **Note:** You can only use one service worker in each PWA, but that doesn't mean you need to place your service worker code in only one file. A service worker can include other files using [`importScripts`](https://developer.mozilla.org/docs/Web/API/WorkerGlobalScope/importScripts) in every browser or using [ECMAScript module imports](https://web.dev/articles/es-modules-in-sw) in some modern browsers.

### Verify whether a service worker is registered

To verify whether a service worker is registered, use developer tools in your
favorite browser.

In Firefox and Chromium-based browsers (Microsoft Edge, Google Chrome, or
Samsung Internet):

1. Open developer tools, then click the **Application** tab.
2. In the left pane, select **Service Workers**.
3. Check that the service worker's script URL appears with the status "Activated". (For more information, see [Lifecycle](https://web.dev/learn/pwa/service-workers#lifecycle)). On Firefox, the status can be "Running" or "Stopped".

In Safari:

1. Click **Develop** \> **Service Workers**.
2. Check this menu for an entry with the current origin. Clicking that entry opens an inspector over the service worker's context.

![Service worker developer tools on Chrome, Firefox and Safari.](https://web.dev/static/learn/pwa/service-workers/image/service-worker-developer-5ebe49234dc0f.png) Service worker developer tools on Chrome, Firefox and Safari. **Note:** To check service worker registration status on a mobile device, you can remotely inspect a phone or tablet from your desktop browser. For details, see [Tools and debug](https://web.dev/learn/pwa/tools-and-debug).

### Scope

The folder your service worker sits in determines its scope. A service worker
that lives at `example.com/my-pwa/sw.js` can control any navigation at or under
the *my-pwa* path, such as `example.com/my-pwa/demos/`. Service workers can
control only items (pages, workers, collectively "clients") in their scope.
This scope applies to browser tabs and PWA windows.

Only *one* service worker is allowed per scope. When a service worker is active
and running, only one instance is typically available no matter how many clients
(PWA windows or browser tabs) are in memory.
| **Warning:** Set the scope of your service worker as close to the root of your app as possible so it can intercept all requests related to your PWA. Don't put it inside, for example, a JavaScript folder, or have it loaded from a CDN.

Safari has more complex scope management, known as partitions, affecting how
scopes work with cross-domain iframes. To learn more about WebKit's
implementation, refer to [their blog post](https://webkit.org/blog/8090/workers-at-your-service/).

## Lifecycle

Service workers have a lifecycle that dictates how they're installed, separately
from your PWA installation.

The service worker lifecycle starts with registering the service worker. The
browser then tries to download and parse the service worker file. If parsing
succeeds, the service worker's `install` event is fired. The `install` event
only fires once.

Service worker installation happens silently, without requiring user permission,
even if the user doesn't install the PWA. The Service Worker API is available
even on platforms that don't support PWA installation, such as Safari and
Firefox on desktop devices.
| **Tip:** Service worker *registration* and *installation* are related but separate events. Registration happens when a page requests a service worker by calling `register()`. Installation happens when a registered service worker exists, can be parsed as JavaScript, and doesn't throw any errors during its first execution.

After the installation, the service worker needs to be activated before it can
control its clients, including your PWA. When the service worker is ready to
control its clients, the `activate` event fires. However, by default, an
activated service worker can't manage the page that registered it until the next
time you navigate to that page by reloading the page or reopening the PWA.

You can listen for events in the service worker's global scope using the `self`
object:

serviceworker.js  

    // This code executes in its own worker or thread
    self.addEventListener("install", event => {
       console.log("Service worker installed");
    });
    self.addEventListener("activate", event => {
       console.log("Service worker activated");
    });

### Update a service worker

Service workers get updated when the browser detects that the service worker
controlling the client and the new version of the service worker file from the
server are byte-different.
| **Warning:** When updating your service worker, keep the filename the same. If you change the name, even by adding file hashes, the browser will never get the new version.

After a successful installation, the new service worker waits to activate until
the old service worker no longer controls any clients. This state is called
"waiting", and it's how the browser ensures that only one version of your
service worker is running at a time.

Refreshing a page or reopening the PWA won't make the new service worker take
control. The user must close or navigate away from all tabs and windows using
the current service worker and then navigate back to give the new service worker
control. For more information, see [The service worker lifecycle](https://web.dev/articles/service-worker-lifecycle).

## Service worker lifespan

An installed and registered service worker can manage all network request
within its scope. It runs on its own thread, with activation and termination
controlled by the browser, which lets it work even before your PWA is open or
after it closes. Service workers run on their own thread, but in-memory state
might not persist between runs of a service worker, so make sure anything you
want to reuse for each run is available either in IndexedDB or some other
persistent storage.

If it's not already running, a service worker starts whenever a network request
is sent in its scope, or when it receives a triggering event like a periodic
background sync or a push message.

Service workers are terminated if they've been idle for a few seconds, or if
they've been busy for too long. Timings for this vary between browsers. If a
service worker has been terminated and an event occurs that would start it up,
it restarts.

## Capabilities

A registered and active service worker uses a thread with a completely different
execution lifecycle from your PWA's main thread. However, by default, the
service worker file itself has no behavior. It won't cache or serve any
resources; these are things your code needs to do. You'll find out how in the
following chapters.

Service worker's capabilities aren't just for proxy or serving HTTP requests.
Other features are available on top of it for other purposes, such as background
code execution, web push notifications, and process payments. We'll discuss
these additions in [Capabilities](https://web.dev/learn/pwa/capabilities).

## Resources

- [Service Worker API (MDN)](https://developer.mozilla.org/docs/Web/API/Service_Worker_API)
- [Service Worker mindset](https://web.dev/articles/service-worker-mindset)
- [WebKit Workers at your service](https://webkit.org/blog/8090/workers-at-your-service/)
- [ES Modules in Service Workers](https://web.dev/articles/es-modules-in-sw)
- [Service worker lifecycle](https://web.dev/articles/service-worker-lifecycle)