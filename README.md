# MKControl Custom Cast Receiver

This is a custom Google Cast receiver: a small webpage that runs *on the
Chromecast itself* instead of the built-in Default Media Receiver. It plays
audio natively (instant start, real seeking - identical mechanism to the
Default Media Receiver) while also showing a custom "now playing" screen
with album art and live lyrics, driven by MKControl over a separate message
channel.

This lets MKControl do **instant audio + live lyrics at the same time** -
something not possible with the Default Media Receiver, since it can only
show one loaded media session at once.

## One-time setup

### 1. Register a Google Cast Developer account

- Go to https://cast.google.com/publish
- Sign in with the Google account you want tied to this
- Pay the one-time, non-refundable **$5** registration fee
- This is a real Google requirement for *any* Styled or Custom receiver -
  the Default Media Receiver is the only option that skips this

### 2. Register your Chromecast devices for testing

Still in the Cast SDK Developer Console:

- Find your TV's serial number: on the TV, go to
  **Settings → Device → About** (varies slightly by device), or check the
  Google Home app under the device's settings
- Add each device (Living Room TV, Master Bdrm TV) under **Devices** in the
  console, with its serial number
- It can take a few minutes to a few hours for a newly registered device to
  actually start seeing unpublished receivers - this is a Google-side
  propagation delay, not something wrong with the setup

### 3. Host `index.html` somewhere with real HTTPS

The Chromecast's embedded browser requires the receiver page to be served
over HTTPS with a trusted certificate - a self-signed cert on your LAN will
not work for the *receiver page itself*. The simplest free option:

**GitHub Pages:**
1. Create a new GitHub repo (can be private or public)
2. Put `index.html` (in this folder) in the repo root
3. Repo Settings → Pages → set source to the main branch
4. GitHub gives you a URL like `https://yourname.github.io/reponame/`
5. Confirm it loads in a normal browser first

Note: the receiver page loading over HTTPS is a different question from
where the *audio/art files themselves* come from - those still come from
MKControl's local server on your LAN over plain HTTP, same as your existing
working casts. Audio/video/image elements loading HTTP content from an
HTTPS page is normally allowed by browsers (this is "passive" mixed
content, treated less strictly than scripts) - but since I can't test this
against your actual Chromecast hardware, **test this first** before
building anything further on top of it. If it doesn't work, the fallback is
tunneling your local audio server through something like Cloudflare Tunnel
or Tailscale Funnel to get it a real HTTPS URL too - let me know if it
comes to that.

### 4. Register the Custom Receiver application

Back in the Cast SDK Developer Console:

- Click **Add New Application → Custom Receiver**
- Enter the HTTPS URL from step 3
- Save, and copy the **Application ID** it gives you (an 8-character code)

### 5. Tell MKControl about it

In `MKControl-v6_0_2.py`, near the top:

```python
CUSTOM_RECEIVER_APP_ID="XXXXXXXX"   # paste your real App ID here
```

That's it - once this is set, checking **"Enable Live Lyrics on TV"** on the
Audio tab automatically uses this custom receiver instead of the old
ffmpeg-based engine. Leave it blank and everything works exactly as it does
today (falls back to the old slow engine when the checkbox is on, fast
native casting when it's off).

## Testing tips

- **Remote debugging**: with the receiver actively casting, open Chrome on
  your computer and go to `chrome://inspect/#devices` - you should see your
  Chromecast listed with an "Inspect" link, giving you real DevTools
  (console, network tab) for what's happening on the TV itself. This is the
  best way to see if audio/art requests are succeeding or failing.
- Unpublished/testing receivers only work on devices you've explicitly
  registered in step 2 - if it doesn't show up at all, double check the
  device is registered and enough time has passed.
- If lyrics don't appear but audio plays fine, check the console (via
  remote debugging) for `Lyrics message handler failed` - that would point
  at a namespace mismatch between `LYRICS_NAMESPACE` in MKControl and in
  `index.html` (they must match exactly).
