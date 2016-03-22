# teacup_nats

A [Teacup](https://github.com/yuce/teacup.git) based [NATS](http://nats.io/) client for Erlang.

## Install

**teacup_nats** requires Erlang/OTP 18.0+. It uses [rebar3](http://www.rebar3.org/)
as the build tool and is available on [hex.pm](https://hex.pm/). Just include the following
in your `rebar.config`:

```erlang
{deps, [teacup_nats]}.
```

## Usage

**teacup_nats** depends on the `teacup` app to be started. Include it in your `.app.src` file:

```erlang
...
  {applications,
   [kernel,
    stdlib,
    teacup
   ]},
...
```

Or, start it manually:

```erlang
ok = application:start(teacup).
```

**rebar3** has a nice way of starting apps in the shell, you can try:

```
$ rebar3 shell --apps teacup
```

### Aysnchronous Connection

* Connection functions:
    * `teacup_nats:connect()`: Connect to the NATS server at address `127.0.0.1`, port `4222`,
    * `teacup_nats:connect(Host :: binary(), Port :: integer())`: Connect to the NATS server
    at `Host` and port `PORT`,
    * `teacup_nats:connect(Host :: binary(), Port :: integer(), Opts :: map())`: Similar to
    above, but also takes an `Opts` map. Currently usable keys:
        * `user => User :: binary()`,
        * `pass => Password :: binary()`
* Publish functions:
    * `teacup_nats:pub(Conn :: teacup_ref(), Subject :: binary())`: Publish message with only
    the subject,
    * `teacup_nats:pub(Conn :: teacup_ref(), Subject :: binary()), Opts :: map()`: Publish message
    the subject with `Options`. Valid options:
        * `payload => Payload :: binary()`,
        * `reply_to => Subject :: binary()`
* Subscribe functions:
    * `teacup_nats:sub(Conn :: teacup_ref(), Subject :: binary())`: Subscribe to the `Subject`,
    * `teacup_nats:sub(Conn :: teacup_ref(), Subject :: binary(), Opts :: map())`: Subscribe to the `Subject`, with
    `Options`. Valid options:
        * `queue_group => QGroup :: binary()`
* Unsubscribe functions:
    * `teacup_nats:unsub(Conn :: teacup_ref(), Subject :: binary())`: Unsubscribe from `Subject`,
    * `teacup_nats:unsub(Conn :: teacup_ref(), Subject :: binary(), Opts :: map())`: Unsubscribe from `Subject`, with
    `Options`. Valid options:
        * `max_messages => MaxMessages :: integer()`: Automatically unsubscribe after receiving `MaxMessages`.


#### Sample

```erlang
main() ->
    % Connect to the NATS server
    {ok, Conn} = teacup_nats:connect(<<"demo.nats.io">>, 4222),
    % When the connection is OK to use, a `ready` message is sent, wait for it
    ready_loop(Conn).

ready_loop(Conn) ->
    receive
        {Conn, ready} ->
            % It's OK to use the connection now
            % Publish some message
            teacup_nats:pub(Conn, <<"teacup.control">>, #{payload => <<"start">>}),
            % subscribe to some subject
            teacup_nats:sub(Conn, <<"foo.*">>),
            loop(Conn)
    end.

loop(Conn) ->
    receive
        {Conn, {msg, Subject, _ReplyTo, Payload}} ->
            % Do something with the received message
            io:format("~p: ~p~n", [Subject, Payload]),
            loop(Conn)
    end.
```


### Synchronous Connection

Synchronous functions use the same signature as the corresponding asynchronous funcitons,
but their namespace is `teacup_nats@sync` instead of `teacup_nats`.

#### Sample

```erlang
main() ->
    % Connect to the NATS server
    {ok, Conn} = teacup_nats@sync:connect(<<"demo.nats.io">>, 4222),
    % The connection is OK to use
    % Publish some message
    ok = teacup_nats@sync:pub(Conn, <<"teacup.control">>, #{payload => <<"start">>}),
    % subscribe to some subject
    ok = teacup_nats@sync:sub(Conn, <<"foo.*">>),
    loop(Conn).

loop(Conn) ->
    receive
        {Conn, {msg, Subject, _ReplyTo, Payload}} ->
            % Do something with the received message
            io:format("~p: ~p~n", [Subject, Payload]),
            loop(Conn)
    end.

```

## License

```
Copyright (c) 2016, Yuce Tekol <yucetekol@gmail.com>.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

* Redistributions of source code must retain the above copyright
  notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright
  notice, this list of conditions and the following disclaimer in the
  documentation and/or other materials provided with the distribution.

* The names of its contributors may not be used to endorse or promote
  products derived from this software without specific prior written
  permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```