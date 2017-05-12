Web Page Media Control API Specification
==========================================

**Version 1.0.**

The **Web Page Media Control API** was created to provide generic way to control web page embedded media content (audio or video). **Media Content Source** is a web page with any king of media source (sound or video). **Media Control Source** is one of the folowing: media buttons on the keyboard, hardware buttons on headset, remote control tool or any other hardware or software way to issue media related commands -- Media Actions.

**Media Action** is an intent to control a media source (to start/stop/pause the playback, or to choose next or previous track, or to change sound volume). Web Page Media Control API described here is used to deliver Media Actions from **Browser-Side Component** to a web application. **Browser-Side Component** is a browser itself or an extension that directly binded to Media Control Source. It provides Media Actions to Media Content Source.

**Media Events** are web page document-level events triggered by Browser-Side Component in response to Media Actions received from Media Control Source. Media Event is a carrier, it holds information about Media Action.

```
                                               |<-- Web Page Media Control API -->|
___________                       _____________                                    ___________
| Media   |                       | Browser-  |                                    | Web     |
| Control |  --(Media Action)-->  | Side      |        --(Media Events)-->         | page    |
| Source  |                       | Component |                                    | scripts |
-----------                       -------------                                    -----------
```

*Example*. Let Media Control Source be keyboard hardware media keys, Browser-Side Component be [Keysocket](https://github.com/borismus/keysocket) Google Chrome extension (provides support for keyboard media keys). Keysocket uses Web Page Media Control API to send Media Action to web pages by Media Events. Web pages use Web Page Media Control API to receive Media Events and perform Media Action.

Media Action Listeners
----------------------

The Browser-Side Component triggers following document-level events (Media Events) on Media Actions:

- `MediaPlayPause`
- `MediaStop`
- `MediaPrev`
- `MediaNext`

To receive Media Events, the web page must register itself.

Web Page Registration to Receive Media Events
---------------------------------------------

Web pages with embedded Media Content Sources must be registered to be controlled by one or more Browser-Side Components. Web page can be registered/unregistered in two ways: static and dynamic.

Web page can be registered/unregistered *statically*. Web page must register itselfs by adding `<meta name="media-controllable">` tag. This tag must have no `content` property, or have `content` property value *not* equal to `no`. If the tag is present at the time of Browser-Side Components initialization, page gets registered automatically. Usualy Browser-Side Components initialization occurs after all page scripts are loaded.

Alternatively, web page can be registered/unregistered *dynamically*. Browser-Side Components listen to `MediaControlStateChanged` document-level event on every opened page. When `MediaControlStateChanged` event is thrown, Browser-Side Component determines what it should do: register or unregister the web-page.

The agrument of event can be instance of [`CustomEvent`][custom-event] or [`Event`][event]. For [`CustomEvent`][custom-event] details property must tell whether to register or unregister the page. If `event.details == "enabled"` the page will be registered. If `event.details == "disabled"` the page will be unregistered. Other values are ignored.

For [`Event`][event] Browser-Side Component looks for `<meta name="media-controllable">` tag on the page. If the tag is found and it's `content` attribute is not equal to `no`, the page will be registered. To unregister the page, `<meta name="media-controllable">` meta tag must be removed, or it's `content` attribure must be set to `no`: `<meta name="media-controllable" content="no">`.

Userland (web page) code example
--------------------------------

Common part: events handling.

``` js
// Wrapper for your plaing source
var player = new MyCustomPlayerImplementation();

// Register for Media Events
document.addEventListener("MediaPlayPause", function () {
    player.toggle();
});
document.addEventListener("MediaStop", function () {
    player.stop();
});
document.addEventListener("MediaPrev", function () {
    player.prev();
});
document.addEventListener("MediaNext", function () {
    player.next();
});
```

**Registration**. Case A -- Event.

``` js
// Register the page to receive user Media Events
addMetaTagToDocumentHead(); // <meta name="media-controllable"> added to html>head inside
document.dispatchEvent(new Event("MediaControlStateChanged"));
```

Case B -- CustomEvent.

``` js
// Register the page to receive user Media Events
document.dispatchEvent(new CustomEvent("MediaControlStateChanged", { "detail": "enabled" }));
```

**Unregistration**. Case A -- Event.

``` js
// Register the page to receive user Media Events
addMetaTagToDocumentHead(); // <meta name="media-controllable"> added to html>head inside
document.dispatchEvent(new Event("MediaControlStateChanged"));
```

Case B -- CustomEvent.

``` js
// Register the page to receive user Media Events
document.dispatchEvent(new CustomEvent("MediaControlStateChanged", { "detail": "disabled" }));
```

Browser-Side Component
----------------------

Browser-Side Component -- browser itself or it's extension -- must:

1. Register a listener for `MediaControlStateChanged` event on `document` tag into every opened tab on every page load. When the event have been caught, Browser-Side Component determines what it should do: register or unregister the web-page. It acts depending on event argument type.

    1. If the type is [`CustomEvent`][custom-event], Browser-Side Component must check it's details property. If `event.details == "enabled"`, the page must be registered. If `event.details == "disabled"`, the page must be unregistered. Other values are ignored.
    
    2. If the event agrument is *not* an instance of [`CustomEvent`][custom-event] (so it's an instance of [`Event`][event]), Browser-Side Component must search `<meta name="media-controllable">` tags into the page. If one or more tags with `content` attribute is not equal to `no` has been found, the page must be registered. In other case (no tags have been found, or `content` of at least one tag equal to `no`) the page must be unregistered.

2. Search for `<meta name="media-controllable">` tag (without `content="no"` attribute) in every tab after every page load.

3. Trigger apprioritate Media Event when a Media Action is received. Browser-Side Component use it's own rules to select the page that will receive Media Event. Single page can be selected using [Media Focus Stack](http://smus.com/remote-controls-web-media/) approach, or all opened pages can receive event, or another logic can be applied.

Browser-Side Component code example
-----------------------------------

```js
// Tab registration/unregistration by MediaControlStateChanged event
document.addEventListener("MediaControlStateChanged", function (event) {
    if (event instanceof CustomEvent && event.detail) {
        if (event.detail === 'enabled') {
            registerOrUnregisterPage(true);
        } else if (event.detail === 'disabled') {
            registerOrUnregisterPage(false);
        }
    } else {
        registerOrUnregisterPage(isPageMediaControllableByTag());
    }
});

function isPageMediaControllableByTag() {
    var tags = document.getElementsByName("media-controllable");
    if (tags.length > 0) {
        for (var i = 0; i < tags.length; i++) {
            if (tags[i].getAttribute("content") === "no") {
                return false;
            }
        }
        return true;
    }
    return false;
}

function registerOrUnregisterPage(register) {
    // operate
}

// Initial tab registration by meta tag
registerOrUnregisterPage(isPageMediaControllableByTag());
```

Changelog
---------

* 1.0. May 12, 2017. `CustomEvent` rules added. More examples.
* 0.4. May 6, 2015. Meta-tag changed.
* 0.3. May 5, 2015. `MediaControlStateChanged` event is introduced, some terms explained, some new terms introduced.
* 0.2. May 1, 2015. Add `MediaControlApiInit` event, add some description.
* 0.1. May 1, 2015. Initial version.

[custom-event]: https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent
[event]: https://developer.mozilla.org/en-US/docs/Web/API/Event