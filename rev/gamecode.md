# GameCode (rev, 300 points)

> I wanted to play this game, but I can't seem to unlock my copy. Can you find a key for me?

## Files:

- Elixir.Gamecode.beam

```
❯ file Elixir.Gamecode.beam
Elixir.Gamecode.beam: Erlang BEAM file
```

- `nc 0.cloud.chals.io 26717`

## Solution:

We're given a compiled Elixir program, which compiles to a [BEAM](https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine)) file.

I know nothing about Elixir. So let's look a bunch of stuff up!

First, we want to try to disassemble the BEAM file. I found a VSCode extension BEAMdasm which can disassemble the file.

Now, let's take a look at the disassembly:

```
Module:  Elixir.Gamecode

Attributes: [{vsn, [15E8CAC54E6D76FA2D297B5C9E7D728C]}]

Compilation Info: [{version, 7.4.5}, {options, [no_spawn_compiler_process, from_core, no_auto_import]}, {source, /app/Gamecode.ex}]


//Function  Elixir.Gamecode:__info__/1
label01:  func_info            Elixir.Gamecode __info__ 1
label02:  select_val           X[0] label01 [compile, label06, md5, label06, attributes, label06, functions, label05, macros, label04, deprecated, label04, module, label03]
label03:  move                 Elixir.Gamecode X[0]
          return              
label04:  move                 nil X[0]
          return              
label05:  move                 [{process_serial, 1}] X[0]
          return              
label06:  move                 X[0] X[1]
          move                 Elixir.Gamecode X[0]
          call_ext_only        2 erlang:get_module_info/2

//Function  Elixir.Gamecode:process_serial/1
label07:  func_info            Elixir.Gamecode process_serial 1 //line Gamecode.ex, 2
label08:  allocate_zero        1 1
          move                 - X[1]
          // split(param1, '-')
          call_ext             2 Elixir.String:split/2 //line Gamecode.ex, 3
          // X[1] = length(X[0])
          gc_bif1              label00 1 erlang:length/1 X[0] X[1] //line Gamecode.ex, 4
          // if length != 6 fail
          is_eq_exact          label09 X[1] 6
          // X[0] = enumerate(X[0])
          call_ext             1 Elixir.Enum:with_index/1 //line Gamecode.ex, 5
          // Y[0] = X[0]
          move                 X[0] Y[0]
          make_fun2            0
          move                 X[0] X[1]
          move                 Y[0] X[0]
          // X[1] = lambda
          // X[0] = enumerate(X[0])
          call_ext_last        2 Elixir.Enum:each/2 1 //line Gamecode.ex, 6
          // false
label09:  move                 false X[0]
          deallocate           1
          return              

//Function  Elixir.Gamecode:module_info/0
label10:  func_info            Elixir.Gamecode module_info 0
label11:  move                 Elixir.Gamecode X[0]
          call_ext_only        1 erlang:get_module_info/1

//Function  Elixir.Gamecode:module_info/1
label12:  func_info            Elixir.Gamecode module_info 1
label13:  move                 X[0] X[1]
          move                 Elixir.Gamecode X[0]
          call_ext_only        2 erlang:get_module_info/2

//Function  Elixir.Gamecode:-process_serial/1-fun-1-/1
label14:  func_info            Elixir.Gamecode -process_serial/1-fun-1- 1 //line Gamecode.ex, 6
label15:  is_tuple             label14 X[0]
          test_arity           label14 X[0] 2
          allocate_zero        2 1
          // X[0] = (e, i)
          // Y[1] = saved value
          move                 X[0] Y[1]
          get_tuple_element    X[0] 0 X[0]
          // X[0] = e
          call_ext             1 Elixir.String:graphemes/1 //line Gamecode.ex, 7
          // X[1] = length of graphemes(e)
          gc_bif1              label00 1 erlang:length/1 X[0] X[1] //line Gamecode.ex, 9
          // Y[0] = degraphemed
          move                 X[0] Y[0]
          // check length is 6
          is_eq_exact          label16 X[1] 6
          make_fun2            1
          move                 X[0] X[1]
          move                 Y[0] X[0]
          init                 Y[0]
          call_ext             2 Elixir.Enum:map/2 //line Gamecode.ex, 10
          call_ext             1 Elixir.Enum:sum/1 //line Gamecode.ex, 14
          // X[1] = i
          get_tuple_element    Y[1] 1 X[1]
          gc_bif2              label00 2 erlang:+/2 X[1] 1 X[2] //line Gamecode.ex, 15
          gc_bif2              label00 3 erlang:rem/2 X[0] X[2] X[2] //line Gamecode.ex, 15
          is_eq_exact          label16 X[2] 0
          gc_bif2              label00 2 erlang:*/2 X[1] 100 X[1] //line Gamecode.ex, 16
          is_lt                label16 X[1] X[0]
          move                 true X[0]
          call_ext_last        1 Elixir.IO:puts/1 2 //line Gamecode.ex, 17
label16:  move                 nil X[0]
          deallocate           2
          return              

//Function  Elixir.Gamecode:-process_serial/1-fun-0-/1
label17:  func_info            Elixir.Gamecode -process_serial/1-fun-0- 1 //line Gamecode.ex, 10
label18:  bs_start_match3      label20 X[0] 1 X[1]
          bs_get_position      X[1] X[0] 2
          bs_get_utf8          label19 X[1] 2 0 X[2]
          bs_test_tail2        label19 X[1] 0
          move                 X[2] X[0]
          return              
label19:  bs_set_position      X[1] X[0]
          bs_get_tail          X[1] X[0] 2
          badmatch             X[0] //line Gamecode.ex, 11
label20:  badmatch             X[0] //line Gamecode.ex, 11
          // no
          int_code_end        
```

As we look through this, we can slowly start to understand what is going on. The `process_serial` function is called, which splits the code by the string `"-"`. It then checks if there are 6 groups, and calls the first lambda on all of them, passing in the group and its index (0 - 5).

The first lambda takes in the group and index, and applies the second lambda to all of the characters in the group, then takes the sum of the results. Then, it computes the following:

```
          // X[1] = i
          get_tuple_element    Y[1] 1 X[1]
          // X[2] = i + 1
          gc_bif2              label00 2 erlang:+/2 X[1] 1 X[2] //line Gamecode.ex, 15
          // X[2] = (previous sum) % (i + 1)
          gc_bif2              label00 3 erlang:rem/2 X[0] X[2] X[2] //line Gamecode.ex, 15
          // assert X[2] is 0, else fail
          is_eq_exact          label16 X[2] 0
          // X[1] = i * 100
          gc_bif2              label00 2 erlang:*/2 X[1] 100 X[1] //line Gamecode.ex, 16
          // assert i * 100 < previous sum, else fail
          is_lt                label16 X[1] X[0]
```

What does the second lambda do? I tried to find documentation about the opcodes here, but I couldn't find anything. After a day of searching and pivoting to other challenges, I resorted to the best strategy of all.

# guess

It's not a very long function, and it only takes a single character in. Additionally, the `bs_get_utf8` opcode seems suggestive of getting the value of the character. Thus, let's guess that this function is getting the UTF-8 value of a character.

From here, we can set up a payload that will pass the check. If we ensure that each of the characters is divisible by 60, we don't have to worry about the divisibility check. Then, we just need to pick a large enough character in UTF-8 such that we don't have to worry about the `100 * i` check. Let's pick `U+4b0`, or `Ұ`. Now, we just have to copy paste this as a code: `ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ`:

```
❯ nc 0.cloud.chals.io 26717

  ________   __  ________________  ___  ____
 / ___/ _ | /  |/  / __/ ___/ __ \/ _ \/ __/
/ (_ / __ |/ /|_/ / _// /__/ /_/ / // / _/
\___/_/ |_/_/  /_/___/\___/\____/____/___/


         __                            __
   ___  / /__ ___ ____ ___   ___ ___  / /____ ____
  / _ \/ / -_) _ `(_-</ -_) / -_) _ \/ __/ -_) __/
 / .__/_/\__/\_,_/___/\__/__\__/_//_/\__/\__/_/  __
/_/_ _  _  _____ _/ (_)__/ / ___ ___ ____(_)__ _/ /
/ _ `/ | |/ / _ `/ / / _  / (_-</ -_) __/ / _ `/ /
\_,_/  |___/\_,_/_/_/\_,_/ /___/\__/_/ /_/\_,_/_/


serial: ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ-ҰҰҰҰҰҰ

                                             _          _         _               _                    _
  __ _  __ _ _ __ ___   ___     ___ ___   __| | ___    (_)___    | |__   ___  ___| |_     ___ ___   __| | ___
 / _` |/ _` | '_ ` _ \ / _ \   / __/ _ \ / _` |/ _ \   | / __|   | '_ \ / _ \/ __| __|   / __/ _ \ / _` |/ _ \
| (_| | (_| | | | | | |  __/  | (_| (_) | (_| |  __/   | \__ \   | |_) |  __/\__ \ |_   | (_| (_) | (_| |  __/
 \__, |\__,_|_| |_| |_|\___|___\___\___/ \__,_|\___|___|_|___/___|_.__/ \___||___/\__|___\___\___/ \__,_|\___|
 |___/                    |_____|                 |_____|   |_____|                 |_____|

 flag{game_code_is_best_code}
 ```
