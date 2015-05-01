# Web Page Media Control API specification

*Version 0.1.*

This is API specification for web pages to be controlled by system-wide media controls. Media controls are special buttons on the keyboard, hardware buttond on headset, remote control or any other hardware or software way to give media related commands (Media Actions) to media source (player). Web Page Media Control API connects web application and Browser-Side Control System. As browsers currently have no common API to send media actions to web pages, special browser extensions must be used.

Media action listeners
----------------------

Browser-Side Control System will trigger following document-level events (Media Events) on media actions:

- MediaPlayPause
- MediaStop
- MediaPrev
- MediaNext

To receive this events web page must be registered.

Register page to receive Media Events
-------------------------------------

If some web page wants to be controlled by system-wide media controls, it should be registered for Media Events using one of this ways:

1. Add `<meta name="media-controlled">` on the page (this tag must be present on the page before page onLoad event is triggered).
2. Trigger `MediaControlled` event on document when page ready to be controlled, optionally trigger `MediaUncontrolled` event on document to remove control over current page.

If page was registered using `<meta name="media-controlled">`, `MediaUncontrolled` event can be used to unregister it.

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
document.dispatchEvent(new Event("MediaControlled"));
```

Browser-Side Control System
---------------------------

Browser (or browser extension) must:

1. Search for `<meta name="media-controlled">` tag in every tab on every pageload and register page to receive Media Events if the tag was found.
2. Register a listen for `MediaControlled` and `MediaUncontrolled` events on document tag in every tab on on every pageload. The first event must register page to receive Media Events, the second event must cancel the registration.
3. Trigger apprioritate Media Event when a Media Action received. Which pages will receive Media Event is a busines of Browser-Side Control System. It can be single page selected using [Media Focus Stack](http://smus.com/remote-controls-web-media/) approach, or all opened pages, or something else.
