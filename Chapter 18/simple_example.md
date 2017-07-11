Joe Armstrong's **ezwebrame** code no longer works.  I asked for help on the erlang-questions forum with some of the errors I was getting, and Joe himself answered and essentially said, "It no longer works.  Tough luck."  Hmmm...I thought the whole point of posting the code on github was to keep it up to date.  Oh, well.

I decided to try and use [gun](https://github.com/ninenines/gun) for the http client, which has websocket support and is maintained by the same person who maintains cowboy, to try to interact with a cowboy server.

To setup cowboy, I followed the cowboy User Guide's [Getting Started](https://ninenines.eu/docs/en/cowboy/2.0/guide/getting_started/) section. Once that was setup correctly, I was at the following prompt:

`(hello_erlang@127.0.0.1)1>`

To setup gun, I opened up another terminal window, and the [gun docs](https://github.com/ninenines/gun/blob/master/doc/src/guide/start.asciidoc) say you can use something called `erlang.mk` and add gun as a dependency to your application.  According to the [Erlang.mk docs](https://erlang.mk/guide/getting_started.html#_getting_started_with_otp_releases), if you create something called _a release_, then you can use the command `make run` (or `gmake run`) to compile and execute your code. After I read the Erlang.mk docs, it wasn't clear to me how to run _an application_ or a _library_, so I opted to create _a release_.  First, I created a directory for my release, then I downloaded erlang.mk:

```
~/erlang_programs$ mkdir my_gun && cd $_

~/erlang_programs/my_gun$ curl -O "https:/erlang.mk/erlang.mk"   
```

Then I used an erlang.mk [command](https://erlang.mk/guide/getting_started.html#_getting_started_with_otp_releases) to create a release:

```
~/erlang_programs/my_gun$ gmake -f erlang.mk bootstrap-lib bootstrap-rel
git clone https://github.com/ninenines/erlang.mk .erlang.mk.build
Cloning into '.erlang.mk.build'...
remote: Counting objects: 7116, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 7116 (delta 2), reused 6 (delta 1), pack-reused 7105
Receiving objects: 100% (7116/7116), 3.29 MiB | 2.02 MiB/s, done.
Resolving deltas: 100% (4504/4504), done.
if [ -f build.config ]; then cp build.config .erlang.mk.build; fi
cd .erlang.mk.build && gmake
gmake[1]: Entering directory '/Users/7stud/erlang_programs/my_gun/.erlang.mk.build'
export LC_COLLATE=C; \
awk 'FNR==1 && NR!=1{print ""}1' core/core.mk index/*.mk core/index.mk core/deps.mk plugins/protobuffs.mk core/erlc.mk core/docs.mk core/rel.mk core/test.mk core/compat.mk plugins/asciidoc.mk plugins/bootstrap.mk plugins/c_src.mk plugins/ci.mk plugins/ct.mk plugins/dialyzer.mk plugins/edoc.mk plugins/erlydtl.mk plugins/escript.mk plugins/eunit.mk plugins/proper.mk plugins/relx.mk plugins/shell.mk plugins/syntastic.mk plugins/triq.mk plugins/xref.mk plugins/cover.mk plugins/sfx.mk core/plugins.mk core/deps-tools.mk \
	| sed 's/^ERLANG_MK_VERSION =.*/ERLANG_MK_VERSION = 2017.07.06-1-gff27159/' \
	| sed 's:^ERLANG_MK_WITHOUT =.*:ERLANG_MK_WITHOUT = :' > erlang.mk
gmake[1]: Leaving directory '/Users/7stud/erlang_programs/my_gun/.erlang.mk.build'
cp .erlang.mk.build/erlang.mk ./erlang.mk
rm -rf .erlang.mk.build

~/erlang_programs/my_gun$ ls
Makefile	erlang.mk	rel		relx.config	src
```
As you can see, the erlang.mk command that I used created some files and directories.  Next, I edited the Makefile to add gun as a dependency:
```make
PROJECT = my_gun
PROJECT_DESCRIPTION = New project
PROJECT_VERSION = 0.1.0

DEPS = gun

include erlang.mk
```

Then to compile and execute the code:

```
~/erlang_programs/my_gun$ gmake run
 DEP    gun
gmake[1]: Entering directory '/Users/7stud/erlang_programs/my_gun/deps/gun'
 DEP    cowlib
 DEP    ranch
gmake[2]: Entering directory '/Users/7stud/erlang_programs/my_gun/deps/cowlib'
 DEPEND cowlib.d
 ERLC   cow_base64url.erl cow_cookie.erl cow_date.erl cow_hpack.erl cow_http.erl cow_http2.erl cow_http_hd.erl cow_http_te.erl cow_mimetypes.erl cow_multipart.erl cow_qs.erl cow_spdy.erl cow_sse.erl cow_uri.erl cow_ws.erl
 APP    cowlib
gmake[2]: Leaving directory '/Users/7stud/erlang_programs/my_gun/deps/cowlib'
gmake[2]: Entering directory '/Users/7stud/erlang_programs/my_gun/deps/ranch'
 DEPEND ranch.d
 ERLC   ranch.erl ranch_acceptor.erl ranch_acceptors_sup.erl ranch_app.erl ranch_conns_sup.erl ranch_listener_sup.erl ranch_protocol.erl ranch_server.erl ranch_ssl.erl ranch_sup.erl ranch_tcp.erl ranch_transport.erl
 APP    ranch
gmake[2]: Leaving directory '/Users/7stud/erlang_programs/my_gun/deps/ranch'
 DEPEND gun.d
 ERLC   gun.erl gun_app.erl gun_content_handler.erl gun_data.erl gun_http.erl gun_http2.erl gun_spdy.erl gun_sse.erl gun_sup.erl gun_ws.erl gun_ws_handler.erl
 APP    gun
 GEN    rebar.config
gmake[1]: Leaving directory '/Users/7stud/erlang_programs/my_gun/deps/gun'
 DEPEND my_gun.d
 APP    my_gun
 GEN    /Users/7stud/erlang_programs/my_gun/.erlang.mk/relx
===> Starting relx build process ...
===> Resolving OTP Applications from directories:
          /Users/7stud/erlang_programs/my_gun/ebin
          /Users/7stud/erlang_programs/my_gun/deps
          /Users/7stud/.evm/erlang_versions/otp_src_19.2/lib/erlang/lib
          /Users/7stud/erlang_programs/my_gun/apps
===> Resolved my_gun_release-1
===> rendering builtin_hook_status hook to "/Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/bin/hooks/builtin/status"
===> Including Erts from /Users/7stud/.evm/erlang_versions/otp_src_19.2/lib/erlang
===> release successfully created!
===> tarball /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/my_gun_release-1.tar.gz successfully created!
Exec: /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/erts-8.2/bin/erlexec -boot /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/releases/1/my_gun_release -mode embedded -boot_var ERTS_LIB_DIR /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/lib -config /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/releases/1/sys.config -args_file /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release/releases/1/vm.args -pa -- console
Root: /Users/7stud/erlang_programs/my_gun/_rel/my_gun_release
/Users/7stud/erlang_programs/my_gun/_rel/my_gun_release
heart_beat_kill_pid = 41598
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]


=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
          supervisor: {local,sasl_safe_sup}
             started: [{pid,<0.352.0>},
                       {id,alarm_handler},
                       {mfargs,{alarm_handler,start_link,[]}},
                       {restart_type,permanent},
                       {shutdown,2000},
                       {child_type,worker}]

=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
          supervisor: {local,sasl_sup}
             started: [{pid,<0.351.0>},
                       {id,sasl_safe_sup},
                       {mfargs,
                           {supervisor,start_link,
                               [{local,sasl_safe_sup},sasl,safe]}},
                       {restart_type,permanent},
                       {shutdown,infinity},
                       {child_type,supervisor}]

=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
          supervisor: {local,sasl_sup}
             started: [{pid,<0.353.0>},
                       {id,release_handler},
                       {mfargs,{release_handler,start_link,[]}},
                       {restart_type,permanent},
                       {shutdown,2000},
                       {child_type,worker}]

=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
         application: sasl
          started_at: 'my_gun@127.0.0.1'

=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
          supervisor: {local,runtime_tools_sup}
             started: [{pid,<0.359.0>},
                       {id,ttb_autostart},
                       {mfargs,{ttb_autostart,start_link,[]}},
                       {restart_type,temporary},
                       {shutdown,3000},
                       {child_type,worker}]

=PROGRESS REPORT==== 10-Jul-2017::21:30:00 ===
         application: runtime_tools
          started_at: 'my_gun@127.0.0.1'
Eshell V8.2  (abort with ^G)

(my_gun@127.0.0.1)1> 
```
Now, at the erlang shell prompt you can call gun functions to send requests to cowboy.  However, typing a lot of code in the shell is a pain, which highlights another advantage of using a release: you can create a function containing the gun commands that you want to execute, and call that function from the erlang shell.   When you issue the command `gmake run`, all the code in the `src` directory of your release will be compiled.  So, I you can create the following file in the src directory of hyour release:

```erlang
-module(my).
-compile(export_all).

get() ->
    {ok, _} = application:ensure_all_started(gun),
    {ok, ConnPid} = gun:open("localhost", 8080),
    {ok, _Protocol} = gun:await_up(ConnPid),

    StreamRef = gun:get(ConnPid, "/"),

    case gun:await(ConnPid, StreamRef) of
        {response, fin, _Status, _Headers} ->
            no_data;
        {response, nofin, _Status, _Headers} ->
            {ok, Body} = gun:await_body(ConnPid, StreamRef),
            io:format("~s~n", [Body])
    end.
```





