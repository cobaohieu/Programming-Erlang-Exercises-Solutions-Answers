I found the ring exercise extremely difficult.  I wrote a solution program five or six times, and every time it would hang somewhere.  Finally, I got something to work, but it seems overly complex.  I had to create a separate receive loop for the "start" process in order to decrement the loop count.  I also passed the pid of the next process as an argument to the receive loop to keep the pid from going out of scope.

```erlang
-module(ring2).
-export([ring/2]).
-include_lib("eunit/include/eunit.hrl"). 
                   
ring(NumProcs, NumLoops) ->
    NextPid = spawn(fun() -> create_ring(NumProcs-1, self()) end),
    NextPid ! {NumLoops, "hello"},
    start(NextPid).  %receive loop for the "start" process.

create_ring(1, StartPid) ->  %...then stop spawning processes.
    loop(StartPid);  %receive loop for the other processes.
create_ring(NumProcs, StartPid) ->
    NextPid = spawn(fun() -> create_ring(NumProcs-1, StartPid) end),
    loop(NextPid).

%receive loop for the "start" process:
start(NextPid) ->
    receive 
        {1, Msg} ->  %...then stop looping.
            io:format("*start* received message: ~s (~w)~n", [Msg, 1]),
            NextPid ! stop;  %kill other processes; this processs will die because it stops looping.
        {NumLoops, Msg} ->
            io:format("*start* received message: ~s (~w)~n", [Msg, NumLoops]),
            NextPid ! {NumLoops-1, Msg},
            start(NextPid)
    end.
   
%receive loop for the other processes:
loop(NextPid) ->
    receive
        {NumLoops, Msg} ->
            io:format("Process ~w received message: ~s (~w)~n", [self(), Msg, NumLoops]),
            NextPid ! {NumLoops, Msg},
            loop(NextPid);
        stop -> 
            NextPid ! stop  % When sent to non-existent "start" process, this still returns stop.
    end.
    
```