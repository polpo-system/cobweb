# cobweb

cobweb is an HTTP and HTTPS server for [polpo](https://github.com/polpo-system/polpo), written in
Oberon; HTTPS is polpo's own TLS 1.3. It serves the files below a directory:

```
loksh cobweb.Serve site                              # http://, port 8080
loksh cobweb.Serve site 8443 /tls cert.pem key.pem   # https://
```

It answers GET and HEAD, one request per connection, with Date, Last-Modified, Content-Type and
Content-Length. A directory gives its `index.html`, or a list of its files when it has none; a
directory named without a slash at the end is redirected (301) to the name with it. Names
starting with a dot, and `..` in a path, are not served (404). Each request is logged: the
client, the status and the request.

```
cobweb.Serve dir [port] [/tls cert.pem key.pem] [/host name] [/n count] [/queue n] [/timeout s]
cobweb.SetQueueLen n
cobweb.SetTimeout s
cobweb.Help
```

`/host name` answers only requests for that host (421 for others), `/n count` ends after count
connections. cobweb serves one client at a time; clients that arrive meanwhile wait in the queue
of the listening socket (16), and each client has ten seconds. `/queue` and `/timeout` change that
for one run; `cobweb.SetQueueLen` and `cobweb.SetTimeout` for the next `cobweb.Serve` of the same
loksh (its shell, or the desktop), since a module stays loaded. Ctrl+C stops the server and closes
its socket. Programs can call `cobweb.Start(dir, cert, key, host, port, count, queue, secs)`.

## The certificate

For HTTPS the certificate must have an ECDSA P-256 key (the PEM forms `EC PRIVATE KEY` and
`PRIVATE KEY` are read): from Let's Encrypt with `certbot --key-type ecdsa`, or self-signed:

```
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -nodes \
  -keyout key.pem -out cert.pem -days 3650 -subj /CN=example.org
```

(The certificate in `test/` is for the tests only: its key is public.)

## Installing

With portia: `loksh portia.Install cobweb` (and `portia.Install /t cobweb` runs the tests). By hand,
in the polpo directory:

```
bin/x86/loksh compiler.Compile /s ../cobweb/cobweb.Mod
```

(acompiler, a7compiler, rvcompiler or mcompiler for the other ports).

## Tests

`test/CobwebTest.Mod` starts cobweb in two child processes, over HTTP and over HTTPS, on the files
of `test/` and asks for pages as a client: `CobwebTest.Run`, in a directory with the files of
`test/`.
