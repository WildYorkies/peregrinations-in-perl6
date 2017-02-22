# How to Web

## Sockets

### role IO::Socket

To get a better idea of what a socket is, let's walk through the documentation on the [IO::Socket role](https://docs.perl6.org/type/IO$COLON$COLONSocket).
The docs will tell us what a socket _is_ by telling us what a socket _does_.

```perl6
method recv(IO::Socket:D: Cool $elems = Inf, :$bin)
```
```
Receive a package and return it, either as a Blob if :bin was passed, or a Str if not. If $elems is supplied, only that many bytes or characters are returned.

Fails if the socket is not connected.
```

The `recv` method tell us that a socket can receive, or read, data.

```perl6
method print(IO::Socket:D: Str(Cool) $string)
```
```
Writes the supplied string to the socket, thus sending it to other end of the connection. The binary version is method write.

Fails if the socket is not connected.
```

The `print` method tells us that a socket can send, or write, data.

```perl6
method close(IO::Socket:D)
```
```
Closes the socket.

Fails if the socket is not connected.
```

And finally, `close` tells us that a socket can be closed (and implies that it can be opened!).

What these methods don't tell us is from and to where these sockets receive and send data.

The answer is _other sockets_.

### class IO::Socket::INET does IO::Socket

The [IO::Socket::INET](https://docs.perl6.org/type/IO$COLON$COLONSocket$COLON$COLONINET) type `does` the
IO::Socket role. So it implements all the methods of the role above, but it also adds some extra stuff that
is specific to TCP sockets.

To continue on with the idea of sockets sending and recieving data from other sockets, let's look at some example code.

```perl6
#!/usr/bin/env perl6
 
my $listen = IO::Socket::INET.new(:listen, :localhost<localhost>, :localport(3333));
loop {
    my $conn = $listen.accept;
    while my $buf = $conn.recv(:bin) {
        $conn.write: $buf;
    }
    $conn.close;
}
```

The `$listen` variable holds a TCP Socket object that is currently listening on your localhost (your own computer).

So we have an open socket. What should we do with it? Well, plug it into another socket so that it can send and
receive data!

Until we force quit the process, this loop will keep going, non-stop. On every iteration of the loop it does
the following:

1. Finds and accepts a connection (another socket). Assign the connection to `$conn`.
2. While the other socket has information to send, receive the info, and write it back to the socket.
3. Close the connection with the other socket.

Now to test our socket above, we must create a socket that "plugs into" or connects with it.

```perl6
#!/usr/bin/env perl6

my $conn = IO::Socket::INET.new(:host<localhost>, :port(3333));
$conn.print: 'Hello, Perl 6';
say $conn.recv;
$conn.close;
```

`$conn` sends our listening socket a message of `'Hello, Perl 6'` and the listening socket sends the same message
back to `$conn`.

Let us rewrite these two code examples in a more metaphorical way. 

Imagine a soda machine where you put quarters in and it just gives your quarters back through the change dispenser.
In this metaphor, you are `$conn`, your quarters are `$buf`, and the soda machine is `$listen`.

See our metaphorical listener/server code below:

```perl6
#!/usr/bin/env perl6
 
my $soda-machine = IO::Socket::INET.new(:listen, :localhost<localhost>, :localport(3333));
loop {
    my $human = $soda-machine.accept;
    while my $money = $human.recv(:bin) {
        $human.write: $money;
    }
    $human.close;
}
```

Now see our metaphorical connector code below:

```perl6
#!/usr/bin/env perl6

# Connect to the soda-machine by creating a new socket that connects to its port
my $human = IO::Socket::INET.new(:host<localhost>, :port(3333));
$human.print: '4 quarters';
say $human.recv; # OUTPUT: '4 quarters'
$human.close;
```

You and the soda machine are both sockets. The difference is that you can create sockets that are meant to primarily
_listen_ and you can create sockets that are meant to connect to listening sockets. See the docs on the [IO::Socket::INET.new method for more info on that](https://docs.perl6.org/type/IO$COLON$COLONSocket$COLON$COLONINET#method_new).

## What is a server?

Using our previous socket metaphor with `$soda-machine` and `$human`, a **server** is an easier way to create and work with
`$soda-machine`'s. In the world of servers, our `$human` from before would be called a **client**.

So think of a server as a listening socket, and think of a client as a connecting socket.

Similar to how Perl6 has a socket role and types that "do" that socket role, the community has
made a server role and types that "do" that role.

### role HTTP::Server

The [HTTP::Server role](https://github.com/perl6/perl6-http-server) is technically three roles in one. A _server_, a
_request_, and a _response_.

Take some time to read the readme file on the github repository. Here are some things I want to point out that relate
to sockets.

The HTTP::Server role says that servers should, among other things, have a "listen" method that starts up the server and
tells it to listen for incoming connections. This is just like our `$soda-machine` that accepts connections from humans.

Here comes the interesting part. Instead of connecting right away, HTTP (HyperText Transfer Protocol) says that we need
to follow some standards.

If a human wants to connect to a soda machine, the human needs to send a *request* to the machine. Then the soda machine
should send a *response* back to the human. These requests and responses have a certain standard that they follow. Please
read the github repository code for [HTTP::Request](https://github.com/perl6/perl6-http-server/blob/master/lib/HTTP/Request.pm6) and [HTTP::Response](https://github.com/perl6/perl6-http-server/blob/master/lib/HTTP/Response.pm6).

Notice that the response that the server sends back has `write` and `close` methods, just like our `$soda-machine` had
send and close methods.

### class HTTP::Server::Threaded does HTTP::Server

An example of a type that does the roles described above is [HTTP::Server::Threaded](https://github.com/perl6/perl6-http-server-threaded). Please take some time to skim the code in that github repository.

There will be a lot of code that you don't understand in those files. I want you to try to find the code that you can
understand, though.

In the [Threaded.pm6](https://github.com/perl6/perl6-http-server-threaded/blob/master/lib/HTTP/Server/Threaded.pm6) file, find the code for the `listen` method.

In the [Response.pm6](https://github.com/perl6/perl6-http-server-threaded/blob/master/lib/HTTP/Server/Threaded/Response.pm6) file, find the code for the `write` and `close` methods.

And notice that the [Request.pm6](https://github.com/perl6/perl6-http-server-threaded/blob/master/lib/HTTP/Server/Threaded/Request.pm6) file relies entirely on the HTTP::Request role and does not customize anything else.

### Tell the Server What to Do

Let's use this example on the HTTP::Server::Threaded readme page.

```perl6
use HTTP::Server::Threaded;

my $s = HTTP::Server::Threaded.new;

$s.handler(sub ($request, $response) {
  $response.headers<Content-Type> = 'text/plain';
  $response.status = 200;
  $response.write("Hello ");
  $response.close("world!");
});

$s.listen;
```

1. Server is created
2. A handler is attached to the [array of handler Callables/subs](https://github.com/perl6/perl6-http-server-threaded/blob/master/lib/HTTP/Server/Threaded.pm6#L20)
3. The server starts listening on its [default port](https://github.com/perl6/perl6-http-server-threaded/blob/master/lib/HTTP/Server/Threaded.pm6#L7)

Once the server starts listening, it can accept connections from clients. With the code above, no matter what the
client "requests" the server will send the same response of "Hello world!", among other things.

You might be wondering, how do we make the server send different things depending on different requests?

### Routing

If you want a server to do different things depending on which URL the client tries to request, you want
to use a _router_. [HTTP::Server::Router](https://github.com/tony-o/perl6-http-server-router) uses everything you've seen thus far and adds this extra functionality.

See some example code below:

```perl6
use HTTP::Server::Threaded;
use HTTP::Server::Router;

my HTTP::Server::Threaded $server .=new;

serve $server;

route '/', sub($req, $res) {
  $res.close('Hello world!');
}

route '/:whatever', sub($req, $res) {
  $res.close($req.params<whatever>);
}

route / .+ /, sub($req, $res) {
  $res.status = 404;
  $res.close('Not found.');
}

$server.listen;
```

In the example above, if you create a client that goes to the `$server`'s port and adds an extra `/`, the server
will respond with "Hello world!". If the client goes to `/something-else` (the second route), the server will
respond with "something-else" echoing it back. Finally, the third route is a catch all that responds with 404.

To use our soda machine metaphor from before, let's look at what it would look like with a server+routes and a client!

Soda machine code:
```perl6
#!/usr/bin/env perl6

use HTTP::Server::Threaded;
use HTTP::Server::Router;

my HTTP::Server::Threaded $soda-machine .=new;

serve $soda-machine;

route '/:money', sub($req, $res) {
  $res.close($req.params<money>);
}

$soda-machine.listen;
```

Human code:
```perl6
#!/usr/bin/env perl6

use Net::HTTP::GET;
my %header   = :Connection<keep-alive>;
my $response = Net::HTTP::GET("http://localhost:8091/4-quarters", :%header);
say $response.content; # OUTPUT: "4-quarters"
```

I won't cover the details of clients yet (our code for the human), but if you want to read more, you can do so [here](https://github.com/ugexe/Perl6-Net--HTTP). 

In the code above, you'll see that our soda machine is still broken! It takes quarters from a human and just gives those
quarters right back to the human. Let's try to build a _working_ soda machine in the next chapter.

Remember that a client is just a socket that is meant for connecting. A server is a socket that is meant for
listening and accepting connections.

## Return of the Sockets: Async Edition

Now that you have a semi-okay grasp of what is going on with servers and clients, let's back up all the way
to the beginning.

In the computer science field, you may have heard of words like parallelism, concurrency, or asynchrony. These
are fancy ways of figuring out how to make things go faster and work for more people at one time.

All of the code I've shown you so far has been synchronous. This means that it starts at the top and runs to
the bottom of the code instructions (essentially). For example, our client code in our human/soda-machine example:

```perl6
# Connect to the soda-machine by creating a new socket that connects to its port
my $human = IO::Socket::INET.new(:host<localhost>, :port(3333));
$human.print: '4 quarters';
say $human.recv; # OUTPUT: '4 quarters'
$human.close;
```

What if we don't know how long it will take for the soda machine to give our money back? What if the human code runs
before the soda machine is even turned on?

There are a lot of what-ifs with synchronous code. I'm going to show you a better way. And in some ways, a more readable
way to program our soda machine sockets.

### class IO::Socket::Async

```perl6
# soda-machine code
react {
    whenever IO::Socket::Async.listen('localhost', 3333) -> $human {
        whenever $human.Supply(:bin) -> $money {
            await $human.write: $money
        }
    }
}
```
```perl6
# human code
await IO::Socket::Async.connect('localhost', 3333).then( -> $promise {
    if $promise.status {
        given $promise.result -> $soda-machine {
            $soda-machine.print('4 quarters');
            react {
                whenever $soda-machine.Supply() -> $response {
                    $response.say;
                    done;
                }
            }
            $soda-machine.close;
        }
    }
});
```



## You Knew It Had To Be Easier or; Frameworks

Clients are clients. But when it comes to servers, they come in many shapes and sizes. The varying kinds of
servers is why the HTTP::Server/Request/Response role system is so handy. It tells us that no matter how weird a
server is, we can always expect it to do some basic things.

In order to hide some of the complexity of working with Servers, Routing, and a host of other things, people
write modules that abstract over it. Sometimes these modules are called a framework because it provides
a frame for us to work with.
