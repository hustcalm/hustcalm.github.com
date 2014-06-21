---
layout: post
title: "Look inside mongoose - an easy to use web server implementation"
date: 2014-04-09 20:22:54 +0800
comments: true
categories: Web HTTP Network
---

I come across with Network Programming recently and find it really charming. Afer reading the book "Advanced Linux Programming", I'm more curious about how to implement a basic http server or web server.

The source code of chapter 11 of "ALP" does tell the key idea, thus communicating through sockets between client and server, for HTTP compliance, just implement the HTTP protocal([RFC
2616](https://tools.ietf.org/html/rfc2616)). To know the things better, I'd like to know how to implement a core and basic HTTP server. You may know 'Apache', 'NginX' and 'Lighttpd' for a while, and they do lead the main tread. Basically, they are heavy enough to begin to know a basic HTTP server implementation.

<!--more-->

After googling around, I find [this link on stackoverflow](http://stackoverflow.com/questions/176409/how-to-build-a-simple-http-server-in-c) is valuable and a good starting point to follow. From there, I decide to dig into [mongoose](https://code.google.com/p/mongoose/). To walk through what mongoose does and provides, I will present some comments of core source code.

## Be aware of the Protocal Stack

Many people get confused by `HTTP`, `FTP`, `TCP/IP`, `SMTP`, etc. To get all these stuff clear of your head, do please spend several miniutes to learn the `Internet communication model`, like the `OSI 7-layer model` or the `TCP/IP 4-layer model`.

Once you know the `phsical`, `data-link`, `network`, `transportation`, `application` what's all about, trust me, you will find all the confused words gone:-)

## HTTP Basics

A web server, basicly implements the `HTTP` protocol to define how `the client interacts with the server`, thus enabling the communication of varies of devices. But aware that `HTTP` is an application protocol, built upon `TCP/IP` which handles the real transportation of data, in the perspective of programming, that is `Socket`.

Give the references a shot:

*   [HTTP(HyperText Transfer Protocol) Basics](http://www.ntu.edu.sg/home/ehchua/programming/webprogramming/HTTP_Basics.html)
*   [HTTP: The Protocol Every Web Developer Must Know](http://code.tutsplus.com/tutorials/http-the-protocol-every-web-developer-must-know-part-1--net-31177)
*   [HTTP Made Really Easy](http://www.jmarshall.com/easy/http/)
*   [HTTP basics - slideshare](http://www.slideshare.net/sanjoysanyal/http-basics)

## Socket Basics

In a word, `socket` is the `API` of `TCP/IP` provided to programmers.
See my blog [What Is Socket Anyway?](http://hustcalm.me/blog/2014/04/17/what-is-socket-anyway/).

## Cross Platform development concerns

Wow, `Mongoose` works on Windows, Mac, UNIX/Linux, iPhone, Android eCos, QNX and many other platforms, what an amazing software!

Well, the basic idea behind `cross platform development` is to abstract another layer to adapt to the different implementations to the same task on different `OS and platforms`, my personal understanding, sorry:-)

When you see the source code of `Mongoose`, be prepared to come across with lots of `#ifdef` and `#ifndef`, etc. Things got really complicated when dealing with `cross platform development`, there are even specifed books and literatures talking about it.

OK I'm also newbie to this, so please `RTFSC`.

## HTTP features kept in mind

As an `embedded` web server, `Mongoose` needs not to implement everything, just the core functionalities, however really powerful enough. Like `CGI, SSI, SSL, Digest auth, Websocket, WEbDAV, Resumed download, URL rewrite, file blacklist, Custom error pages, Virtual hosts, IP-based ACL, Windows service`, even `Lua Server Pages`.

As `HTTP` is an application protocol, so every feature is really related to the description in the `RFC`s along with many more extensions like [HTTPS - HTTP over TLS](http://datatracker.ietf.org/doc/rfc2818/).

Though many heavy web servers like `Apache` or `Nginx` does not implement everything of `HTTP`, so `keep it simple and work`.

## The programming paradigm

There is not any `class` defined in `Mongoose`, though it can be compiled using `C++`, so the source code is `pure C` actually. The underlying dirty work will be handled by [net_skeleton](https://github.com/cesanta/net_skeleton), like the `management of socket connections` and `sending and receiving data packets`.

To be clear, `object` is implemented through the use of `struct`, like `mg_server`, `mg_connection`. Get a good knowledge of `C programming language` before you decide to dig into `Mongoose`, period.

## I/O Models to know

`Blocking I/O` or `Asynchronous I/O`? 

See the following references for good:

*   [NetworkingCPP - I/O Models](http://www.madwizard.org/programming/tutorials/netcpp/5)
*   [The C10K problem](http://www.kegel.com/c10k.html)
*   [Boost application performance using asynchronous I/O](http://www.ibm.com/developerworks/linux/library/l-async/)

`Mongoose` is using the traditional `select()`, serving many clients with each thread, and using nonblocking I/O as illustrated in the function `ns_server_poll` implemented in `net_skeleton`.

## The core of Mongoose

Let the journey begin!

### Net Skeleton

Quote from [here](https://github.com/cesanta/net_skeleton):

> Net Skeleton is a networking library written in C. It provides easy to use event-driven interface that allows to implement network protocols or scalable network applications with little effort. Net Skeleton releives developers from the burden of network programming complexity and let them concentrate on the logic. Net Skeleton saves time and money.

``` c Core interfaces provided by net_skeleton
void ns_server_free(struct ns_server *);
int ns_server_poll(struct ns_server *, int milli);
void ns_server_wakeup(struct ns_server *);
void ns_server_wakeup_ex(struct ns_server *, ns_callback_t, void *, size_t);
void ns_iterate(struct ns_server *, ns_callback_t cb, void *param);
struct ns_connection *ns_add_sock(struct ns_server *, sock_t sock, void *p);

int ns_bind(struct ns_server *, const char *addr);
int ns_set_ssl_cert(struct ns_server *, const char *ssl_cert);
int ns_set_ssl_ca_cert(struct ns_server *, const char *ssl_ca_cert);
struct ns_connection *ns_connect(struct ns_server *, const char *host,
                                 int port, int ssl, void *connection_param);

int ns_send(struct ns_connection *, const void *buf, int len);
int ns_printf(struct ns_connection *, const char *fmt, ...);
int ns_vprintf(struct ns_connection *, const char *fmt, va_list ap);

// Utility functions
void *ns_start_thread(void *(*f)(void *), void *p);
int ns_socketpair(sock_t [2]);
int ns_socketpair2(sock_t [2], int sock_type);  // SOCK_STREAM or SOCK_DGRAM
void ns_set_close_on_exec(sock_t);
void ns_sock_to_str(sock_t sock, char *buf, size_t len, int flags);
int ns_hexdump(const void *buf, int len, char *dst, int dst_len);
```

``` c Core data structures used in net_skeleton
struct ns_server {
  void *server_data;
  sock_t listening_sock;
  struct ns_connection *active_connections;
  ns_callback_t callback;
  SSL_CTX *ssl_ctx;
  SSL_CTX *client_ssl_ctx;
  sock_t ctl[2];
};

struct ns_connection {
  struct ns_connection *prev, *next;
  struct ns_server *server;
  sock_t sock;
  union socket_address sa;
  struct iobuf recv_iobuf;
  struct iobuf send_iobuf;
  SSL *ssl;
  void *connection_data;
  time_t last_io_time;
  unsigned int flags;
#define NSF_FINISHED_SENDING_DATA   (1 << 0)
#define NSF_BUFFER_BUT_DONT_SEND    (1 << 1)
#define NSF_SSL_HANDSHAKE_DONE      (1 << 2)
#define NSF_CONNECTING              (1 << 3)
#define NSF_CLOSE_IMMEDIATELY       (1 << 4)
#define NSF_ACCEPTED                (1 << 5)
#define NSF_WANT_READ               (1 << 6)
#define NSF_WANT_WRITE              (1 << 7)

#define NSF_USER_1                  (1 << 26)
#define NSF_USER_2                  (1 << 27)
#define NSF_USER_3                  (1 << 28)
#define NSF_USER_4                  (1 << 29)
#define NSF_USER_5                  (1 << 30)
#define NSF_USER_6                  (1 << 31)
};
```

``` c Data structures and interfaces for I/O used for sending and receiving data
struct iobuf {
  char *buf;
  size_t len;
  size_t size;
};

void iobuf_init(struct iobuf *, size_t initial_size);
void iobuf_free(struct iobuf *);
size_t iobuf_append(struct iobuf *, const void *data, size_t data_size);
void iobuf_remove(struct iobuf *, size_t data_size);
```

``` c Event handling in net_skeleton
// Net skeleton interface
// Events. Meaning of event parameter (evp) is given in the comment.
enum ns_event {
  NS_POLL,     // Sent to each connection on each call to ns_server_poll()
  NS_ACCEPT,   // New connection accept()-ed. union socket_address *remote_addr
  NS_CONNECT,  // connect() succeeded or failed. int *success_status
  NS_RECV,     // Data has benn received. int *num_bytes
  NS_SEND,     // Data has been written to a socket. int *num_bytes
  NS_CLOSE     // Connection is closed. NULL
};

// Callback function (event handler) prototype, must be defined by user.
// Net skeleton will call event handler, passing events defined above.
struct ns_connection;
typedef void (*ns_callback_t)(struct ns_connection *, enum ns_event, void *evp);
```

Besides the interfaces provieded, there are lots of `private` functions to make everything possible, these are all implemented in `net_skeleton.c` as `static` functions.

### Mongoose

Well, here finally comes the main character! As the dirty work has been done by `net_skeleton`, `mongoose` can focus on the real job.

``` c Core interfaces of mongoose
// Server management functions
struct mg_server *mg_create_server(void *server_param, mg_handler_t handler);
void mg_destroy_server(struct mg_server **);
const char *mg_set_option(struct mg_server *, const char *opt, const char *val);
int mg_poll_server(struct mg_server *, int milliseconds);
const char **mg_get_valid_option_names(void);
const char *mg_get_option(const struct mg_server *server, const char *name);
void mg_set_listening_socket(struct mg_server *, int sock);
int mg_get_listening_socket(struct mg_server *);
void mg_iterate_over_connections(struct mg_server *, mg_handler_t, void *);
void mg_wakeup_server(struct mg_server *);
struct mg_connection *mg_connect(struct mg_server *, const char *, int, int);

// Connection management functions
void mg_send_status(struct mg_connection *, int status_code);
void mg_send_header(struct mg_connection *, const char *name, const char *val);
void mg_send_data(struct mg_connection *, const void *data, int data_len);
void mg_printf_data(struct mg_connection *, const char *format, ...);

int mg_websocket_write(struct mg_connection *, int opcode,
                       const char *data, size_t data_len);

// Deprecated in favor of mg_send_* interface
int mg_write(struct mg_connection *, const void *buf, int len);
int mg_printf(struct mg_connection *conn, const char *fmt, ...);

const char *mg_get_header(const struct mg_connection *, const char *name);
const char *mg_get_mime_type(const char *name, const char *default_mime_type);
int mg_get_var(const struct mg_connection *conn, const char *var_name,
               char *buf, size_t buf_len);
int mg_parse_header(const char *hdr, const char *var_name, char *buf, size_t);
int mg_parse_multipart(const char *buf, int buf_len,
                       char *var_name, int var_name_len,
                       char *file_name, int file_name_len,
                       const char **data, int *data_len);

// Utility functions
void *mg_start_thread(void *(*func)(void *), void *param);
char *mg_md5(char buf[33], ...);
int mg_authorize_digest(struct mg_connection *c, FILE *fp);

struct mg_expansion {
  const char *keyword;
  void (*handler)(struct mg_connection *);
};
void mg_template(struct mg_connection *, const char *text,
                 struct mg_expansion *expansions);
```

``` c Core data structures
// This structure contains information about HTTP request.
struct mg_connection {
  const char *request_method; // "GET", "POST", etc
  const char *uri;            // URL-decoded URI
  const char *http_version;   // E.g. "1.0", "1.1"
  const char *query_string;   // URL part after '?', not including '?', or NULL

  char remote_ip[48];         // Max IPv6 string length is 45 characters
  char local_ip[48];          // Local IP address
  unsigned short remote_port; // Client's port
  unsigned short local_port;  // Local port number

  int num_headers;            // Number of HTTP headers
  struct mg_header {
    const char *name;         // HTTP header name
    const char *value;        // HTTP header value
  } http_headers[30];

  char *content;              // POST (or websocket message) data, or NULL
  size_t content_len;         // Data length

  int is_websocket;           // Connection is a websocket connection
  int status_code;            // HTTP status code for HTTP error handler
  int wsbits;                 // First byte of the websocket frame
  void *server_param;         // Parameter passed to mg_add_uri_handler()
  void *connection_param;     // Placeholder for connection-specific data
  void *callback_param;       // Needed by mg_iterate_over_connections()
};

struct mg_server {
  struct ns_server ns_server;
  union socket_address lsa;   // Listening socket address
  mg_handler_t event_handler;
  char *config_options[NUM_OPTIONS];
};

// Local endpoint representation
union endpoint {
  int fd;                     // Opened regular local file
  struct ns_connection *nc;   // CGI or proxy->target connection
};
```

``` c Event handling
enum mg_event {
  MG_POLL = 100,  // Callback return value is ignored
  MG_CONNECT,     // If callback returns MG_FALSE, connect fails
  MG_AUTH,        // If callback returns MG_FALSE, authentication fails
  MG_REQUEST,     // If callback returns MG_FALSE, Mongoose continues with req
  MG_REPLY,       // If callback returns MG_FALSE, Mongoose closes connection
  MG_CLOSE,       // Connection is closed, callback return value is ignored
  MG_LUA,         // Called before LSP page invoked
  MG_HTTP_ERROR   // If callback returns MG_FALSE, Mongoose continues with err
};
typedef int (*mg_handler_t)(struct mg_connection *, enum mg_event);
```

Find something? Yes, `mongoose` is heavily using the underlying `net_skeleton`, as `ns_server` in ` mg_server` suggests.

Or look at this:

``` c Mongoose is just a wrapper for application built upon net_skeleton?
int mg_poll_server(struct mg_server *server, int milliseconds) {
   return ns_server_poll(&server->ns_server, milliseconds);
 }
```

To be really aware of what is heppening under the hood, the functions below is the key:

``` c The key functions to implement the basic web server
static void ns_add_to_set(sock_t sock, fd_set *set, sock_t *max_fd) {
   if (sock != INVALID_SOCKET) {
     FD_SET(sock, set);
     if (*max_fd == INVALID_SOCKET || sock > *max_fd) {
       *max_fd = sock;
     }
   }
 }

int ns_server_poll(struct ns_server *server, int milli) {
   struct ns_connection *conn, *tmp_conn;
   struct timeval tv;
   fd_set read_set, write_set;
   int num_active_connections = 0;
   sock_t max_fd = INVALID_SOCKET;
   time_t current_time = time(NULL);

   if (server->listening_sock == INVALID_SOCKET &&
       server->active_connections == NULL) return 0;

   FD_ZERO(&read_set);
   FD_ZERO(&write_set);
   ns_add_to_set(server->listening_sock, &read_set, &max_fd);
   ns_add_to_set(server->ctl[1], &read_set, &max_fd);

   for (conn = server->active_connections; conn != NULL; conn = tmp_conn) {
     tmp_conn = conn->next;
     ns_call(conn, NS_POLL, &current_time);
     ns_add_to_set(conn->sock, &read_set, &max_fd);
     if (conn->flags & NSF_CONNECTING) {
       ns_add_to_set(conn->sock, &write_set, &max_fd);
     }
     if (conn->send_iobuf.len > 0 && !(conn->flags & NSF_BUFFER_BUT_DONT_SEND)) {
       ns_add_to_set(conn->sock, &write_set, &max_fd);
     } else if (conn->flags & NSF_CLOSE_IMMEDIATELY) {
       ns_close_conn(conn);
     }
   }

tv.tv_sec = milli / 1000;
tv.tv_usec = (milli % 1000) * 1000;

if (select((int) max_fd + 1, &read_set, &write_set, NULL, &tv) > 0) {
 // Accept new connections
 if (server->listening_sock != INVALID_SOCKET &&
     FD_ISSET(server->listening_sock, &read_set)) {
   // We're not looping here, and accepting just one connection at
   // a time. The reason is that eCos does not respect non-blocking
   // flag on a listening socket and hangs in a loop.
   if ((conn = accept_conn(server)) != NULL) {
     conn->last_io_time = current_time;
   }
 }

 // Read possible wakeup calls
 if (server->ctl[1] != INVALID_SOCKET &&
     FD_ISSET(server->ctl[1], &read_set)) {
   unsigned char ch;
   recv(server->ctl[1], &ch, 1, 0);
   send(server->ctl[1], &ch, 1, 0);
 }

for (conn = server->active_connections; conn != NULL; conn = tmp_conn) {
       tmp_conn = conn->next;
       if (FD_ISSET(conn->sock, &read_set)) {
         conn->last_io_time = current_time;
         ns_read_from_socket(conn);
       }
       if (FD_ISSET(conn->sock, &write_set)) {
         if (conn->flags & NSF_CONNECTING) {
           ns_read_from_socket(conn);
         } else if (!(conn->flags & NSF_BUFFER_BUT_DONT_SEND)) {
           conn->last_io_time = current_time;
           ns_write_to_socket(conn);
         }
       }
     }
   }

   for (conn = server->active_connections; conn != NULL; conn = tmp_conn) {
     tmp_conn = conn->next;
     num_active_connections++;
     if (conn->flags & NSF_CLOSE_IMMEDIATELY) {
       ns_close_conn(conn);
     }
   }
   //DBG(("%d active connections", num_active_connections));

   return num_active_connections;
 }
```

Feel something? Go dig yourself for much more fun:-)

## Good References
*   [tinyhttpd](http://tinyhttpd.sourceforge.net/)
*   [nweb: a tiny, safe Web server (static pages only)](http://www.ibm.com/developerworks/systems/library/es-nweb/#icomments)
*   [Design and Implementation of an HTTP Server in Java](https://users.cs.jmu.edu/bernstdh/web/common/lectures/slides_http-server-example_java.php)
