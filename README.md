Web Page Media Control API Specification
==========================================

*Version 0.4.*

The *Web Page Media Control API* was created to provide generic way of to control web page embedded media content (audio or video). *Media Content Source* is a web page with any king of media source (sound or video). *Media Control Source* is one of the folowing: media buttons on the keyboard, hardware buttons on headset, remote control tool or any other hardware or software way to issue media related commands -- Media Actions.

*Media Action* is an intent to control a media source (to start/stop/pause the playback, or to choose next or previous track, or to change sound volume). *Web Page Media Control API* described here can be used to deliver Media Actions from *Browser-Side Component* to a web application. *Browser-Side Component* is a browser itself or an extension that directly binded to Media Control Source. It provides Media Actions to Media Content Source.

*Media Events* are web page document-level events triggered by Browser-Side Component in response to Media Actions received from Media Control Source. Media Event is a carrier, it holds information about Media Action (just a message type currently).

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

The Browser-Side Component triggers the following document-level events (Media Events) on Media Actions:

- `MediaPlayPause`
- `MediaStop`
- `MediaPrev`
- `MediaNext`

To receive Media Events, the web page must register itself.

Web Page Registration to Receive Media Events
---------------------------------------------

Web pages with embedded Media Content Sources must be registered to be controlled by one or more Browser-Side Components. The web pages must register itselfs by adding `<meta name="media-controllable">` tag. This tag must have no `content` property, or have `content` property value not equal to `no`. If the tag is present into the page in time of Browser-Side Components initialization, the page gets registered automatically. Usualy Browser-Side Components initialization occurs after all page scripts are loaded.

Web page can be registered/unregistered dynamically. Browser-Side Components listen to `MediaControlStateChanged` document-level event on every opened page. After Browser-Side Component receives `MediaControlStateChanged` event, it looks for `<meta name="media-controllable">` tag on the page. If the tag has been found and it's `content` attribute is not equal to `no`, the page gets registered. To unregister the page, `<meta name="media-controllable">` meta tag must be removed, or it's `content` attribure must be set to `no`: `<meta name="media-controllable" content="no">`.

Userland (web page) code example
--------------------------------

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

// Register the page to receive user Media Events
addMetaTagToDocumentHead(); // <meta name="media-controllable"> added to html>head inside
document.dispatchEvent(new Event("MediaControlStateChanged"));
```

Browser-Side Component
----------------------

Browser-Side Component -- browser itself or it's extension -- must:

1. Register a listener for `MediaControlStateChanged` event on `document` tag into every opened tab on every page load. When the event have been caught, Browser-Side Component must look for `<meta name="media-controllable">` tags into the page. `content` attribute of all tags must not be equal to `no`. If the tag was found and the page is not already registered, it must become registered. If the tag was not found (or it had `content="no"` attribute) and the page is currently registered, this registration must be canceled.

2. Search for `<meta name="media-controllable">` tag (without `content="no"` attribute) in every tab after every page load.

3. Trigger apprioritate Media Event when a Media Action is received. Browser-Side Component use it's own rules to make a decision which pages will receive Media Event. Single page can be selected using [Media Focus Stack](http://smus.com/remote-controls-web-media/) approach, or all opened pages can receive event, or another logic can be applied.
