Asyncio Example Client
======================

This client sends one HTTPS GET request using :mod:`asyncio` streams. It shows
how to drive h2 from a coroutine without adding networking or concurrency to
the protocol implementation itself. It disables server push and collects only
the requested stream.

Run it with a server that supports HTTP/2 over TLS:

.. code-block:: console

   $ python examples/asyncio/asyncio-client.py nghttp2.org /httpbin/get

The example uses the system trust store, verifies the server certificate and
hostname, and checks that ALPN negotiated ``h2`` before sending HTTP/2 bytes.
``--port`` selects a non-default HTTPS port. Headers and trailers go to standard
error; the response body goes to standard output as bytes.

The same coroutine reads network data, handles h2 events, and flushes generated
frames with ``write()`` and ``drain()``. Received data is acknowledged using its
``flow_controlled_length`` (which includes padding), so responses larger than
the initial receive window can finish. ``StreamEnded`` marks a complete
response, including responses that end in trailers or contain no body. A reset,
a premature EOF, or GOAWAY before completion fails the request. The transport
is closed on success, failure, cancellation, and the command's 15-second timeout.

This is a complete single-request example, not a general-purpose HTTP client.
It buffers the response body in memory and does not follow redirects, decode
content encodings, upload a body, retry requests, or multiplex streams. An
application handling large bodies should consume chunks as they arrive and
acknowledge them after consumption; one reusing a connection must manage the
lifetimes of its individual streams as well as the connection.

The current example server has an expired self-signed certificate which
the example client rejects during SSL verification. add argument ``-k``
or ``--insecure`` *a la* `curl <https://curl.se/docs/sslcerts.html>`__
to the client invocation to skip these checks:

.. code:: bash

   python3 asyncio-client.py --port=8443 --insecure 127.0.0.1

The certificate expiry can be refreshed:

.. code:: bash

   openssl req -newkey rsa:2048 -x509 -days 365 -keyout cert.key -out cert.crt -nodes -subj '/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd/CN=localhost'
   python3 asyncio-server.py

Add argument ``--cacert`` to use this certificate:

.. code:: bash

   python3 asyncio-client.py --port=8443 --cacert=cert.crt 127.0.0.1

.. literalinclude:: ../../examples/asyncio/asyncio-client.py
   :language: python
   :linenos:
   :encoding: utf-8
