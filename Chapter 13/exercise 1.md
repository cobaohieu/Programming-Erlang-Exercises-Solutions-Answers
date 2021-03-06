My first attempt is below. 
```erlang
-module(e1).
-export([my_spawn/3, atomize/0, test/0]).

my_spawn(Mod, Func, Args) ->
    %%Create separate process to run Func:
    FuncPid = spawn(Mod, Func, Args),
    statistics(wall_clock),   %%Get the start time (and throw away the return value).
    
    %%****WHAT IF THE FuncPid PROCESS FAILS HERE???******
    
    %%Create separate process for the monitor:
    spawn(fun() ->
        Ref = monitor(process, FuncPid),  %%Ref is sent in the 'DOWN' message.
        receive
            {'DOWN', Ref, process, FuncPid, Why} -> %% Ref and FuncPid are bound!
                {_, WallTime} = statistics(wall_clock),   %%Get the elapsed since the last call to statistics(wall_clock).
                io:format("Process ~w lived for ~w milliseconds,~n", 
                          [FuncPid, WallTime]),
                io:format("then died due to: ~p~n", [Why]),
                io:format("*---------*~n")
        end 
    end),  %%Monitor process dies after receiving a 'DOWN' message.

    FuncPid. %%To mimic spawn(Mod, Func, Args), return the Pid of the
             %%process that is running Func.

atomize() ->
    receive
        List -> list_to_atom(List)
    end.

test() ->
    timer:sleep(500), %%Allow time for shell startup
                      %%so output appears after 1> prompt.
    io:format("testing...~n"),  

    Atomizer = my_spawn(e1, atomize, []),
    timer:sleep(2000), %%Let atomize() run for awhile.
    Atomizer ! hello,  %%Will cause an error in Atomizer.
    ok.

```

In the shell:

```
$ ./run
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.2  (abort with ^G)

1> testing...
Process <0.59.0> lived for 2001 milliseconds,
then died due to: {badarg,[{erlang,list_to_atom,[hello],[]},
                           {e1,atomize,0,[{file,"e1.erl"},{line,27}]}]}

=ERROR REPORT==== 7-May-2017::15:23:31 ===
Error in process <0.59.0> with exit value:
{badarg,[{erlang,list_to_atom,[hello],[]},
         {e1,atomize,0,[{file,"e1.erl"},{line,27}]}]}
*---------*

```

That solution has a serious problem though: if the FuncPid process fails immediately after being spawned, the monitor will not be hooked up yet, so the information about the process life will not be output. (**Edit:** Nope!  A monitor will still work if the process doesn't exist when the monitor is created: the monitor will receive a 'DOWN' message and `Why`will be `noproc`, meaning _no process_).  I think the following attempt covers the problems with my first solution:
```erlang
-module(e1).
-export([my_spawn/3, atomize/0, test/0]).

my_spawn(Mod, Func, Args) ->
    MySpawn = self(),
    Tag = make_ref(),  %%Tag needs to be accesible in the spawned func as well
                       %%as in the receive at the end of this func.
    %%Create separate process for the monitor:
    Monitor =
        spawn(fun() ->
            {FuncPid, Ref} = spawn_monitor(Mod, Func, Args),  %%Erlang will include Ref in the 'DOWN' message
            statistics(wall_clock),
            MySpawn ! {self(), Tag, FuncPid}, %%Use the Monitor pid and a reference to tag the message.  
            receive
                {'DOWN', Ref, process, FuncPid, Why} -> %% Ref and FuncPid are bound!
                    {_, WallTime} = statistics(wall_clock),   %%Elapsed time since last call.
                    io:format("Process ~w lived for ~w milliseconds,~n", 
                              [FuncPid, WallTime]),
                    io:format("then died due to: ~p~n", [Why]),
                    io:format("*---------*~n")
            end 
        end),  %%Monitor process dies after receiving a 'DOWN' message.
    
    receive %%Blocks until the line that sends the message in Monitor executes.
        {Monitor, Tag, FuncPid} -> FuncPid
    end.
 

atomize() ->
    receive
        List -> list_to_atom(List)
    end.

test() ->
    timer:sleep(500), %%Allow time for shell startup
                      %%so output appears after 1> prompt.
    io:format("testing...~n"),  

    Atomizer = my_spawn(e1, atomize, []),
    timer:sleep(2000), %%Let atomize() run for awhile.
    Atomizer ! hello,
    ok.

```

In the shell:
```
$ ./run
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.2  (abort with ^G)
1> testing...
Process <0.60.0> lived for 2001 milliseconds,
then died due to: {badarg,[{erlang,list_to_atom,[hello],[]},
                           {e1,atomize,0,[{file,"e1.erl"},{line,31}]}]}

=ERROR REPORT==== 8-May-2017::09:29:58 ===
Error in process <0.60.0> with exit value:
{badarg,[{erlang,list_to_atom,[hello],[]},
         {e1,atomize,0,[{file,"e1.erl"},{line,31}]}]}
*---------*

```
