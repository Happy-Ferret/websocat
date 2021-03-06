
# Websocat Reference (in progress)

Websocat has many command-line options and special format for positional arguments.

There are three main modes of websocat invocation:

* Simple client mode: `websocat wss://your.server/url`
* Simple server mode: `websocat -s 127.0.0.1:8080`
* Advanced [socat][1]-like mode: `websocat -t ws-l:127.0.0.1:8080 mirror:`

Ultimately in any of those modes websocat creates two connections and exchanges data between them.
If one of the connections is bytestream-oriented (for example the terminal stdin/stdout or a TCP connection), but the other is message-oriented (for example, a WebSocket or UDP) then websocat operates in lines: each line correspond to a message. Details of this are configurable by various options.

`ws-l:` or `mirror:` above are examples of address types. With the exception of special cases like WebSocket URL `ws://1.2.3.4/` or stdio `-`, websocat's positional argument is defined by this rule:

```
<specifier> ::= ( <overlay> ":" )* <addrtype> ":" [address]
```

Some address types may be "aliases" to other address types or combinations of overlays and address types.

[1]:http://www.dest-unreach.org/socat/doc/socat.html

# `--help=long`

"Advanced" options and flags are denoted by `[A]` marker.


```

websocat 1.1.0
Vitaly "_Vi" Shukela <vi0oss@gmail.com>
Command-line client for web sockets, like netcat/curl/socat for ws://.

USAGE:
    websocat ws://URL | wss://URL               (simple client)
    websocat -s port                            (simple server)
    websocat [FLAGS] [OPTIONS] <addr1> <addr2>  (advanced mode)

FLAGS:
        --dump-spec                             [A] Instead of running, dump the specifiers representation to stdout
    -e, --set-environment                       Set WEBSOCAT_* environment variables when doing exec:/cmd:/sh-c:
                                                Currently it's WEBSOCAT_URI and WEBSOCAT_CLIENT for
                                                request URI and client address (if TCP)
                                                Beware of ShellShock or similar security problems.
    -E, --exit-on-eof                           Close a data transfer direction if the other one reached EOF
        --jsonrpc                               Format messages you type as JSON RPC 2.0 method calls. First word
                                                becomes method name, the rest becomes parameters, possibly automatically
                                                wrapped in [].
        --linemode-strip-newlines               [A] Don't include trailing \n or \r\n coming from streams in WebSocket
                                                messages
    -0, --null-terminated                       Use \0 instead of \n for linemode
        --no-line                               [A] Don't automatically insert line-to-message transformation
        --no-fixups                             [A] Don't perform automatic command-line fixups. May destabilize
                                                websocat operation. Use --dump-spec without --no-fixups to discover what
                                                is being inserted automatically and read the full manual about Websocat
                                                internal workings.
    -1, --one-message                           Send and/or receive only one message. Use with --no-close and/or -u/-U.
        --oneshot                               Serve only once. Not to be confused with -1 (--one-message)
        --exec-sighup-on-stdin-close            [A] Make exec: or sh-c: or cmd: send SIGHUP on UNIX when input is
                                                closed.
        --exec-sighup-on-zero-msg               [A] Make exec: or sh-c: or cmd: send SIGHUP on UNIX when facing incoming
                                                zero-length message.
    -q                                          Suppress all diagnostic messages, except of startup errors
        --reuser-send-zero-msg-on-disconnect    [A] Make reuse-raw: send a zero-length message to the peer when some
                                                clients disconnects.
    -s, --server-mode                           Simple server mode: specify TCP port or addr:port as single argument
    -S, --strict                                strict line/message mode: drop too long messages instead of splitting
                                                them, drop incomplete lines.
        --udp-oneshot                           [A] udp-listen: replies only one packet per client
    -u, --unidirectional                        Inhibit copying data in one direction
    -U, --unidirectional-reverse                Inhibit copying data in the other direction (or maybe in both directions
                                                if combined with -u)
        --unlink                                [A] Unlink listening UNIX socket before binding to it
    -V, --version                               Prints version information
    -v                                          Increase verbosity level to info or further
    -b, --binary                                Send message to WebSockets as binary messages
    -n, --no-close                              Don't send Close message to websocket on EOF
    -t, --text                                  Send message to WebSockets as text messages

OPTIONS:
        --queue-len <broadcast_queue_len>
            [A] Number of pending queued messages for broadcast reuser [default: 16]

    -B, --buffer-size <buffer_size>                Maximum message size, in bytes [default: 65536]
    -H, --header <custom_headers>...
            Add custom HTTP header to websocket client request. Separate header name and value with a colon and
            optionally a single space. Can be used multiple times.
        --exec-args <exec_args>...
            [A] Arguments for the `exec:` specifier. Must be the last option, everything after it gets into the exec
            args list.
    -h, --help <help>
            See the help.
            --help=short is the list of easy options and address types
            --help=long lists all options and types (see [A] markers)
            --help=doc also shows longer description and examples.
        --origin <origin>                          Add Origin HTTP header to websocket client request
        --restrict-uri <restrict_uri>
            When serving a websocket, only accept the given URI, like `/ws`
            This liberates other URIs for things like serving static files or proxying.
    -F, --static-file <serve_static_files>...
            Serve a named static file for non-websocket connections.
            Argument syntax: <URI>:<Content-Type>:<file-path>
            Argument example: /index.html:text/html:index.html
            Directories are not and will not be supported for security reasons.
            Can be specified multiple times.
        --protocol <websocket_protocol>            Specify Sec-WebSocket-Protocol: header
        --websocket-version <websocket_version>    Override the Sec-WebSocket-Version value
        --ws-c-uri <ws_c_uri>                      [A] URI to use for ws-c: overlay [default: ws://0.0.0.0/]

ARGS:
    <addr1>    In simple mode, WebSocket URL to connect. In advanced mode first address (there are many kinds of
               addresses) to use. See --help=types for info about address types. If this is an address for
               listening, it will try serving multiple connections.
    <addr2>    In advanced mode, second address to connect. If this is an address for listening, it will accept only
               one connection.


Basic examples:
  Command-line websocket client:
    websocat ws://echo.websocket.org/
    
  WebSocket server
    websocat -s 8080
    
  WebSocket-to-TCP proxy:
    websocat --binary ws-l:127.0.0.1:8080 tcp:127.0.0.1:5678
    

```

# Full list of address types

"Advanced" address types are denoted by `[A]` marker.


### `ws://`

Internal name for --dump-spec: WsClient


Insecure (ws://) WebSocket client. Argument is host and URL.

Example: connect to public WebSocket loopback and copy binary chunks from stdin to the websocket.

    websocat - ws://echo.websocket.org/


### `wss://`

Internal name for --dump-spec: WsClientSecure


Secure (wss://) WebSocket client. Argument is host and URL.

Example: forward TCP port 4554 to a websocket

    websocat tcp-l:127.0.0.1:4554 wss://127.0.0.1/some_websocket

### `ws-listen:`

Aliases: `ws-l:`, `l-ws:`, `listen-ws:`  
Internal name for --dump-spec: WsTcpServer


WebSocket server. Argument is host and port to listen.

Example: Dump all incoming websocket data to console

    websocat ws-l:127.0.0.1:8808 -

Example: the same, but more verbose:

    websocat ws-l:tcp-l:127.0.0.1:8808 reuse:-


### `inetd-ws:`

Aliases: `ws-inetd:`  
Internal name for --dump-spec: WsInetdServer


WebSocket inetd server. [A]

TODO: transfer the example here


### `l-ws-unix:`

Internal name for --dump-spec: WsUnixServer


WebSocket UNIX socket-based server. [A]


### `l-ws-abstract:`

Internal name for --dump-spec: WsAbstractUnixServer


WebSocket abstract-namespaced UNIX socket server. [A]


### `stdio:`

Aliases: `-`  
Internal name for --dump-spec: Stdio


Same as `-`. Read input from console, print to console.

This specifier can be specified only one time.
    
Example: simulate `cat(1)`. This is an exception from "only one time" rule above:

    websocat - -

Example: SSH transport

    ssh -c ProxyCommand='websocat - ws://myserver/mywebsocket' user@myserver


### `inetd:`

Internal name for --dump-spec: Inetd


Like `stdio:`, but intented for inetd(8) usage. [A]

Automatically enables `-q` (`--quiet`) mode.

`inetd-ws:` - is of `ws-l:inetd:`

Example of inetd.conf line that makes it listen for websocket
connections on port 1234 and redirect the data to local SSH server.

    1234 stream tcp nowait myuser  /opt/websocat websocat inetd-ws: tcp:127.0.0.1:22


### `tcp:`

Aliases: `tcp-connect:`, `connect-tcp:`, `tcp-c:`, `c-tcp:`  
Internal name for --dump-spec: TcpConnect


Connect to specified TCP host and port. Argument is a socket address.

Example: simulate netcat netcat

    websocat - tcp:127.0.0.1:22

Example: redirect websocket connections to local SSH server over IPv6

    websocat ws-l:0.0.0.0:8084 tcp:[::1]:22


### `tcp-listen:`

Aliases: `listen-tcp:`, `tcp-l:`, `l-tcp:`  
Internal name for --dump-spec: TcpListen


Listen TCP port on specified address.
    
Example: echo server

    websocat tcp-l:0.0.0.0:1441 mirror:
    
Example: redirect TCP to a websocket

    websocat tcp-l:0.0.0.0:8088 ws://echo.websocket.org


### `sh-c:`

Internal name for --dump-spec: ShC


Start specified command line using `sh -c` (even on Windows)
  
Example: serve a counter

    websocat -U ws-l:127.0.0.1:8008 sh-c:'for i in 0 1 2 3 4 5 6 7 8 9 10; do echo $i; sleep 1; done'
  
Example: unauthenticated shell

    websocat --exit-on-eof ws-l:127.0.0.1:5667 sh-c:'bash -i 2>&1'


### `cmd:`

Internal name for --dump-spec: Cmd


Start specified command line using `sh -c` or `cmd /C` (depending on platform)

Otherwise should be the the same as `sh-c:` (see examples from there).


### `exec:`

Internal name for --dump-spec: Exec


Execute a program directly (without a subshell), providing array of arguments on Unix [A]

Example: Serve current date

  websocat -U ws-l:127.0.0.1:5667 exec:date
  
Example: pinger

  websocat -U ws-l:127.0.0.1:5667 exec:ping --exec-args 127.0.0.1 -c 1
  


### `readfile:`

Internal name for --dump-spec: ReadFile


Synchronously read a file. Argument is a file path.

Blocking on operations with the file pauses the whole process

Example: Serve the file once per connection, ignore all replies.

    websocat ws-l:127.0.0.1:8000 readfile:hello.json



### `writefile:`

Internal name for --dump-spec: WriteFile



Synchronously truncate and write a file.

Blocking on operations with the file pauses the whole process

Example:

    websocat ws-l:127.0.0.1:8000 writefile:data.txt



### `appendfile:`

Internal name for --dump-spec: AppendFile



Synchronously append a file.

Blocking on operations with the file pauses the whole process

Example: Logging all incoming data from WebSocket clients to one file

    websocat -u ws-l:127.0.0.1:8000 reuse:appendfile:log.txt


### `udp:`

Aliases: `udp-connect:`, `connect-udp:`, `udp-c:`, `c-udp:`  
Internal name for --dump-spec: UdpConnect


Send and receive packets to specified UDP socket, from random UDP port  


### `udp-listen:`

Aliases: `listen-udp:`, `udp-l:`, `l-udp:`  
Internal name for --dump-spec: UdpListen


Bind an UDP socket to specified host:port, receive packet
from any remote UDP socket, send replies to recently observed
remote UDP socket.

Note that it is not a multiconnect specifier like e.g. `tcp-listen`:
entire lifecycle of the UDP socket is the same connection.

File a feature request on Github if you want proper DNS-like request-reply UDP mode here.


### `open-async:`

Internal name for --dump-spec: OpenAsync


Open file for read and write and use it like a socket. [A]
Not for regular files, see readfile/writefile instead.
  
Example: Serve big blobs of random data to clients

    websocat -U ws-l:127.0.0.1:8088 open-async:/dev/urandom



### `open-fd:`

Internal name for --dump-spec: OpenFdAsync


Use specified file descriptor like a socket. [A]

Example: Serve random data to clients v2

    websocat -U ws-l:127.0.0.1:8088 reuse:open-fd:55   55< /dev/urandom


### `threadedstdio:`

Internal name for --dump-spec: ThreadedStdio


Stdin/stdout, spawning a thread. [A]

Like `-`, but forces threaded mode instead of async mode

Use when standard input is not `epoll(7)`-able or you want to avoid setting it to nonblocking mode.


### `unix:`

Aliases: `unix-connect:`, `connect-unix:`, `unix-c:`, `c-unix:`  
Internal name for --dump-spec: UnixConnect


Connect to UNIX socket. Argument is filesystem path. [A]

Example: forward connections from websockets to a UNIX stream socket

    websocat ws-l:127.0.0.1:8088 unix:the_socket


### `unix-listen:`

Aliases: `listen-unix:`, `unix-l:`, `l-unix:`  
Internal name for --dump-spec: UnixListen


Listen for connections on a specified UNIX socket [A]

Example: forward connections from a UNIX socket to a WebSocket

    websocat --unlink unix-l:the_socket ws://127.0.0.1:8089
    
Example: Accept forwarded WebSocket connections from Nginx

    umask 0000
    websocat --unlink ws-l:unix-l:/tmp/wstest tcp:[::]:22
      
Nginx config:
    
    location /ws {
        proxy_read_timeout 7d;
        proxy_send_timeout 7d;
        #proxy_pass http://localhost:3012;
        proxy_pass http://unix:/tmp/wstest;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection \"upgrade\";
    }

This configuration allows to make Nginx responsible for
SSL and also it can choose which connections to forward
to websocat based on URLs.

Obviously, Nginx can also redirect to TCP-listening
websocat just as well - UNIX sockets are not a requirement for this feature.

TODO: --chmod option?


### `unix-dgram:`

Internal name for --dump-spec: UnixDgram


Send packets to one path, receive from the other. [A]
A socket for sending must be already openend.

I don't know if this mode has any use, it is here just for completeness.

Example:

    socat unix-recv:./sender -&
    websocat - unix-dgram:./receiver:./sender


### `abstract:`

Aliases: `abstract-connect:`, `connect-abstract:`, `abstract-c:`, `c-abstract:`  
Internal name for --dump-spec: AbstractConnect


Connect to UNIX abstract-namespaced socket. Argument is some string used as address. [A]

Too long addresses may be silently chopped off.

Example: forward connections from websockets to an abstract stream socket

    websocat ws-l:127.0.0.1:8088 abstract:the_socket

Note that abstract-namespaced Linux sockets may not be normally supported by Rust,
so non-prebuilt versions may have problems with them.


### `abstract-listen:`

Aliases: `listen-abstract:`, `abstract-l:`, `l-abstract:`  
Internal name for --dump-spec: AbstractListen


Listen for connections on a specified abstract UNIX socket [A]

Example: forward connections from an abstract UNIX socket to a WebSocket

    websocat abstract-l:the_socket ws://127.0.0.1:8089

Note that abstract-namespaced Linux sockets may not be normally supported by Rust,
so non-prebuilt versions may have problems with them.


### `abstract-dgram:`

Internal name for --dump-spec: AbstractDgram


Send packets to one address, receive from the other. [A]
A socket for sending must be already openend.

I don't know if this mode has any use, it is here just for completeness.

Example (untested):

    websocat - abstract-dgram:receiver_addr:sender_addr

Note that abstract-namespaced Linux sockets may not be normally supported by Rust,
so non-prebuilt versions may have problems with them. In particular, this mode
may fail to work without `workaround1` Cargo feature.


### `mirror:`

Internal name for --dump-spec: Mirror


Simply copy output to input. No arguments needed.

Example: emulate echo.websocket.org

    websocat -t ws-l:127.0.0.1:1234 mirror:


### `literalreply:`

Internal name for --dump-spec: LiteralReply


Reply with a specified string for each input packet.

Example:

    websocat ws-l:0.0.0.0:1234 literalreply:'{"status":"OK"}'


### `clogged:`

Internal name for --dump-spec: Clogged


Do nothing. Don't read or write any bytes. Keep connections in "hung" state. [A]


### `literal:`

Internal name for --dump-spec: Literal


Output a string, discard input.

Example:

    websocat ws-l:127.0.0.1:8080 literal:'{ "hello":"world"} '


### `assert:`

Internal name for --dump-spec: Assert


Check the input.  [A]

Read entire input and panic the program if the input is not equal
to the specified string. Used in tests.


### `assert2:`

Internal name for --dump-spec: Assert2


Check the input. [A]

Read entire input and emit an error if the input is not equal
to the specified string.


### `seqpacket:`

Aliases: `seqpacket-connect:`, `connect-seqpacket:`, `seqpacket-c:`, `c-seqpacket:`  
Internal name for --dump-spec: SeqpacketConnect


Connect to AF_UNIX SOCK_SEQPACKET socket. Argument is a filesystem path. [A]

Start the path with `@` character to make it connect to abstract-namespaced socket instead.

Too long paths are silently truncated.

Example: forward connections from websockets to a UNIX seqpacket abstract socket

    websocat ws-l:127.0.0.1:1234 seqpacket:@test


### `seqpacket-listen:`

Aliases: `listen-seqpacket:`, `seqpacket-l:`, `l-seqpacket:`  
Internal name for --dump-spec: SeqpacketListen


Listen for connections on a specified AF_UNIX SOCK_SEQPACKET socket [A]

Start the path with `@` character to make it connect to abstract-namespaced socket instead.

Too long (>=108 bytes) paths are silently truncated.

Example: forward connections from a UNIX seqpacket socket to a WebSocket

    websocat --unlink seqpacket-l:the_socket ws://127.0.0.1:8089




# Full list of overlays

"Advanced" overlays denoted by `[A]` marker.


### `ws-upgrade:`

Aliases: `upgrade-ws:`, `ws-u:`, `u-ws:`  
Internal name for --dump-spec: WsServer


WebSocket upgrader / raw server. Specify your own protocol instead of usual TCP. [A]

All other WebSocket server modes actually use this overlay under the hood.

Example: serve incoming connection from socat

    socat tcp-l:1234,fork,reuseaddr exec:'websocat -t ws-u\:stdio\: mirror\:'


### `reuse-raw:`

Aliases: `raw-reuse:`  
Internal name for --dump-spec: Reuser


Reuse subspecifier for serving multiple clients: unpredictable mode. [A]

Better used with --unidirectional, otherwise replies get directed to
random connected client.

Example: Forward multiple parallel WebSocket connections to a single persistent TCP connection

    websocat -u ws-l:0.0.0.0:8800 reuse:tcp:127.0.0.1:4567

Example (unreliable): don't disconnect SSH when websocket reconnects

    websocat ws-l:[::]:8088 reuse:tcp:127.0.0.1:22


### `broadcast:`

Aliases: `reuse:`, `reuse-broadcast:`, `broadcast-reuse:`  
Internal name for --dump-spec: BroadcastReuser


Reuse this connection for serving multiple clients, sending replies to all clients.

Messages from any connected client get directed to inner connection,
replies from the inner connection get duplicated across all connected
clients (and are dropped if there are none).

If WebSocket client is too slow for accepting incoming data,
messages get accumulated up to the configurable --broadcast-buffer, then dropped.

Example: Simple data exchange between connected WebSocket clients

    websocat -E ws-l:0.0.0.0:8800 reuse-broadcast:mirror:


### `autoreconnect:`

Internal name for --dump-spec: AutoReconnect


Re-establish underlying connection on any error or EOF

Example: keep connecting to the port or spin 100% CPU trying if it is closed.

    websocat - autoreconnect:tcp:127.0.0.1:5445
    
Example: keep remote logging connection open (or flood the host if port is closed):

    websocat -u ws-l:0.0.0.0:8080 reuse:autoreconnect:tcp:192.168.0.3:1025
  
TODO: implement delays between reconnect attempts


### `ws-c:`

Aliases: `c-ws:`, `ws-connect:`, `connect-ws:`  
Internal name for --dump-spec: WsConnect


Low-level WebSocket connector. Argument is a some another address. [A]

URL and Host: header being sent are independent from the underlying connection.

Example: connect to echo server in more explicit way

    websocat --ws-c-uri=ws://echo.websocket.org/ - ws-c:tcp:174.129.224.73:80

Example: connect to echo server, observing WebSocket TCP packet exchange

    websocat --ws-c-uri=ws://echo.websocket.org/ - ws-c:cmd:"socat -v -x - tcp:174.129.224.73:80"



### `msg2line:`

Internal name for --dump-spec: Message2Line


Line filter: Turns messages from packet stream into lines of byte stream. [A]

Ensure each message (a chunk from one read call from underlying connection)
contains no inner newlines (or zero bytes) and terminates with one newline.

Reverse of the `line2msg:`.

Unless --null-terminated, replaces both newlines (\x0A) and carrige returns (\x0D) with spaces (\x20) for each read.

Does not affect writing at all. Use this specifier on both ends to get bi-directional behaviour.

Automatically inserted by --line option on top of the stack containing a websocket.

Example: TODO


### `line2msg:`

Internal name for --dump-spec: Line2Message


Line filter: turn lines from byte stream into messages as delimited by '\\n' or '\\0' [A]

Ensure that each message (a successful read call) is obtained from a line [A]
coming from underlying specifier, buffering up or splitting content as needed.

Reverse of the `msg2line:`.

Does not affect writing at all. Use this specifier on both ends to get bi-directional behaviour.

Automatically inserted by --line option at the top of the stack opposite to websocket-containing stack.

Example: TODO


### `jsonrpc:`

Internal name for --dump-spec: JsonRpc


[A] Turns messages like `abc 1,2` into `{"jsonrpc":"2.0","id":412, "method":"abc", "params":[1,2]}`.

For simpler manual testing of websocket-based JSON-RPC services

Example: TODO



  
### Address types to be done:

`sctp:` and `ssl:`

### Final example

Final example just for fun: wacky mode

    websocat ws-c:ws-l:ws-c:- tcp:127.0.0.1:5678
    
Connect to a websocket using stdin/stdout as a transport,
then accept a websocket connection over the previous websocket used as a transport,
then connect to a websocket using previous step as a transport,
then forward resulting connection to the TCP port.

(Excercise to the reader: manage to make it actually connect to 5678).

