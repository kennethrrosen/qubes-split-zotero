# qubes-split-zotero
THIS REPO IS A WORK IN PROGRESS. USE AT YOUR OWN RISK.
Split Zotero configuration between offline research/writing qube and online browser.


### Prereqs
- Qubes OS installed and set up.
- Zotero installed in writ-standalonevm.
- Zotero Connector plugin for Firefox installed in browser-appvm.
- Flask installed in browser-appvm:
```
    sudo apt install python3-flask  # For Debian/Ubuntu-based systems
```
### Setup
    In writ-standalonevm, open a terminal and create a qrexec service using the following commands:
```
[browser0user ~]$ echo '#!/bin/sh' > /etc/qubes-rpc/custom.ZoteroService
[browser0user ~]$echo 'zotero "$@"' >> /etc/qubes-rpc/custom.ZoteroService
[browser0user ~]$chmod +x /etc/qubes-rpc/custom.ZoteroService
[dom0user ~]$ echo 'browser-appvm writ-standalonevm allow' >> /etc/qubes-rpc/policy/custom.ZoteroService
```
This will allow the browser-appvm to send requests to writ-standalonevm without any prompt.

### Mock Server in browser-appvm

In browser-appvm, create a Python script called `mock_zotero_server.py` for the mock server:

```
    from flask import Flask, request
    import subprocess

    app = Flask(__name__)

    @app.route('/', methods=['POST'])
    def zotero_mock():
        data = request.data
        subprocess.run(["qrexec-client-vm", "writ-standalonevm", "custom.ZoteroService"], input=data)
        return "OK"

    if __name__ == '__main__':
        app.run(port=23119)
```

### Run the Mock Server

Still in browser-appvm, run:
```
[browser0user ~]$ python3 mock_zotero_server.py
[browser0user ~]$ sudo nano /rw/config/rc.local
[browser0user ~]$ echo '/path/to/your/script/mock_zotero_server.py'
[browser0user ~]$ sudo chmod +x /rw/config/rc.local
```

This will start the mock server on each vm reboot, and it will listen for incoming requests from Zotero Connector.

### Usage

With the mock server operational:
- Launch your browser in browser-appvm.
- As you browse and find citations, use the Zotero Connector as you usually would.
- Data will be captured by the mock server in browser-appvm and then forwarded to the writ-standalonevm's Zotero instance.

### Required Security Disclaimer

Bypassing Qubes' inherent isolation introduces potential risks:
- Always be wary of the data you're forwarding between VMs.
- Ensure the mock server only listens to requests from the Zotero Connector.
- Periodically review the configuration for potential vulnerabilities.
