Here is a naive implementation:

```erlang
time_func(F) -> 
    Start = now(),
    F(),
    End = now(),
    [
     element(I, End) - element(I, Start) || I <- lists:seq(1, size(Start) )
    ].
    %list_to_tuple(
      %fix_timestamp(Timestamp)
    %).
```