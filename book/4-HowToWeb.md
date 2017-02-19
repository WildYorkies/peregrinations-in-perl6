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

The `recv` method tell us that a socket can recieve, or read, data.

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

What these methods don't tell us is from and to where these sockets recieve and send data.

The answer is _other sockets_.

### class IO::Socket::INET does IO::Socket

The [IO::Socket::INET](https://docs.perl6.org/type/IO$COLON$COLONSocket$COLON$COLONINET) type `does` the
IO::Socket role. So it implements all the methods that the role requires, but it also adds some extra stuff that
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
recieve data!

Until we force quit the process, this loop will keep going, non-stop. On every iteration of the loop it does
the following:

1. Finds and accepts a connection (another socket). Assign the connection to `$conn`.
2. While the other socket has information to send, recieve the info, and write it back to the socket.
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

See our listener/server code below:

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

Now see our connector code below:

```perl6
#!/usr/bin/env perl6

# Connect to the soda-machine by creating a new socket that connects to its port
my $human = IO::Socket::INET.new(:host<localhost>, :port(3333));
$human.print: '4 quarters';
say $human.recv; # OUTPUT: '4 quarters'
$human.close;
```

You and the soda machine are both sockets. The difference is that you can create sockets that are meant to primarily
_listen_ and you can create sockets that are meant to connect to listening sockets.

## What is a server?

In our previous socket metaphor with `$soda-machine` and `$human`, a Server is an easier way to create and work with
`$soda-machine`'s. Similar to how Perl6 has a socket role and types that "do" that socket role, the community has
made a server role and types that "do" that role.

## role HTTP::Server

Walk throuh this code https://github.com/perl6/perl6-http-server

### A minimal implementation of the role

write a minimal version. maybe use this as a baseline? 
https://github.com/supernovus/perl6-http-easy

## HTTP::Server Implementations or; The work is already done

Walk through server async code and show how to use it 
https://github.com/perl6/perl6-http-server-async

or perhaps use https://github.com/perl6/perl6-http-server-threaded ?

## Callable App or; Give the Server Something To Do 

Writing an app that plugs in to HTTP::Server::Async 

## Routing

Telling things where to go with HTTP::Server::Router

https://github.com/tony-o/perl6-http-server-router

## You Knew It Had To Be Easier or; Frameworks

https://github.com/tony-o/perl6-hiker

