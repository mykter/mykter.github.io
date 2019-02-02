---
layout: post
title:  "Streaming from a USB record player to Sonos speakers via a Raspberry Pi"
date:   2019-02-02
categories: linux
---

I was very happy to find [two][coreyk] [guides][basdp] to setting up a Raspberry Pi to stream music from a turntable with a USB output, so we could listen to it on our existing Sonos speakers and via a network-connected receiver in a different room.

I adapted the two guides and learned a few things along the way -- this is mostly a record for my own reference, but perhaps it will be of help to someone else too.

The changes from [coreyk][coreyk]'s and [basdp][basdp]'s guides are roughly:
 * Used Raspbian Stretch (no changes needed)
 * Use stock darkice binaries from the repository
 * Marginally simpler darkice init script fixes
 * Stream from port 80 so you don't have to specify a port
 * Power control from a mobile via a URL

1. Will be replaced with the ToC
{:toc}

Stock darkice
-------------
coreyk compiles darkice from source, to add AAC+ support. I don't need this, as on a local network the suggestion of doing fixed 320kbps MP3 encoding is fine.

basdp provides their own .deb. This is fragile (what versions of raspbian will it work with?) and there's no way to tell if it's trustworthy.

I had no issues with the stock darkice. This meant the only packages I needed to install were darkice and icecast2 (and vim!).

Simpler darkice init script fix
-------------------------------
I followed [coreyk's ][coreyk] guide, but instead of deleting the pidfile with `rm`, you can pass `--remove-pidfile` to `start-stop-daemon` when stopping.

Default port
------------
Both guides suggest setting up icecast to listen on port 8000. This is fine, but makes the URL slightly ugly.

With port 80, you can stream from `http://vinyl.local/listen`.

The port can easily be changed in `/etc/darkice.cfg` and `/etc/icecast2/icecast.xml`, the trick is to give icecast the [capability][caps] to bind to port 80:
```bash
sudo setcap 'cap_net_bind_service=+ep' `which icecast2`
```

Reboot and shutdown from a mobile
------------------------------
To provide a clean shutdown option, I wrote this little script to listen for web requests. A shortcut to the URL alongside the Sonos app on my phone's launcher then provides a convenient way to cleanly shut it down. I don't know what the risk of SD card corruption is from hard power-offs, but this eases my mind.

It's called from /etc/rc.local so it should always be running when the Pi is. This also means it runs as root - not generally a good idea for a web server, to say the least, but in this case it's quite convenient as that gives it permission to shutdown and reboot.

```python
#!/usr/bin/env python3

from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib import parse
import subprocess

# HTTPRequestHandler class
class handler(BaseHTTPRequestHandler):
    def do_GET(self):
        parsed_path = parse.urlparse(self.path).path
        if parsed_path in ['/reboot','/restart']:
            code = 200
            message = "Rebooting..."
            subprocess.Popen(["shutdown", "-r", "now"]) # will run concurrently
        elif parsed_path in ['/halt','/shutdown','/stop']:
            code = 200
            message = "Shutting down..."
            subprocess.Popen(["shutdown", "-h", "now"]) # will run concurrently
        else:
            code = 404
            message = "Not found. Available paths: /reboot, /halt\n"

        self.send_response(code)
        self.send_header('Content-type','text/html')
        self.end_headers()

        # Send response to client as utf8
        self.wfile.write(bytes(message, "utf8"))
        return

def run():
    # bind to all addresses, unprivileged port
    server_address = ('', 8000)
    httpd = HTTPServer(server_address, handler)
    print('running server...')
    httpd.serve_forever()

run()
```


[caps]:http://man7.org/linux/man-pages/man7/capabilities.7.html
[coreyk]:https://github.com/coreyk/darkice-libaacplus-rpi-guide
[basdp]:https://github.com/basdp/USB-Turntables-to-Sonos-with-RPi
