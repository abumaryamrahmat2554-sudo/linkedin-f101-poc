# F101 PoC — CSRF on `POST /psettings/profile-visibility`

Proof of concept for a cross site request forgery on LinkedIn's legacy settings
endpoint. A plain form POST from an unrelated origin changes the victim's
**Profile viewing options** with no CSRF token of any kind.

## How to run

The page must be served from a real origin, not opened as a `file://` URL.

```
python3 -m http.server 8101
```

Then open `http://127.0.0.1:8101/` in a browser that is signed in to LinkedIn.

1. Open the verify link on the page and note the current value of
   `discloseAsProfileViewer`.
2. Click a **different** value. Sending the value the account already has
   returns `202` and changes nothing, which looks like the bug failing.
3. Reload the verify link. The value has changed. Restore by clicking the
   original value.

## Browser note

Use **Chrome or Chromium**. Firefox's Total Cookie Protection partitions the
cookie jar for cross site requests by default, so the session cookie is not
attached and the endpoint answers `401`. That `401` is the absence of a
session, not the endpoint refusing the request.

Opening the file as `file://` fails for the same reason: the origin becomes
`null` and the cookie jar is partitioned away.

## Warning

`DISCLOSE_ANONYMOUS` and `HIDE` put the account into private mode, which
disables Who Viewed Your Profile and erases its viewer history. That is the
product's own documented behaviour, not an effect of this page. Use a test
account, and prefer the `DISCLOSE_FULL` direction, which destroys nothing.
