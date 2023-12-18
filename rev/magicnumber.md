# magicnumber (rev, 200 points)

> numbers are hard, but sometimes they are predictable

## Files:

- magic_number

```
❯ file magic_number
magic_number: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=882ec9903a01a40600e9f7e79e5fa44e44f967f4, stripped
```

- `nc 0.cloud.chals.io 32732`

## Solution:

Since we have an ELF, the first step is to decompile it. Let's run it through Ghidra. I've already renamed the functions here.

```c
long add_0x44(int param_1)

{
  int result;
  ulong i;
  
  result = param_1;
  for (i = 0; i < 0x11; i = i + 1) {
    result = result + 4;
  }
  return (long)result;
}

long shl(long param,ulong amt)

{
  long result;
  ulong i;
  
  result = param;
  for (i = 0; i < amt; i = i + 1) {
    result = result << 1;
  }
  return result;
}

ulong prng(ulong num,long iters)

{
  long tmp;
  
  if ((num != 1) && (iters != 0)) {
                    /* if even: n / 2 */
    if ((num & 1) == 0) {
      num = prng(num >> 1,iters + -1);
    }
    else {
                    /* else: multiply by 8 and add 1 */
      tmp = shl(num,3);
      num = prng(tmp + 1,iters + -1);
    }
  }
  return num;
}

undefined8 main(void)

{
  undefined8 x;
  ulong option;
  
  puts("Select your option: ");
  option = read_ul();
  if (option == 300) {
    x = add_0x44(300);
    x = prng(x,2);
    x = add_0x44(x);
    x = prng(x,8);
    x = add_0x44(x);
    x = prng(x,5);
    add_0x44(x);
  }
  else if (option < 0x12d) {
    if (option == 1) {
      x = prng(1,4);
      x = prng(x,4);
      x = prng(x,4);
      x = prng(x,4);
      prng(x,4);
    }
    else {
      if (option != 0x14) {
fail:
        fwrite("You messed up somewhere ...\n",1,0x1c,stdout);
                    /* WARNING: Subroutine does not return */
        exit(-1);
      }
      x = add_0x44(0x14);
      x = add_0x44(x);
      x = add_0x44(x);
      x = add_0x44(x);
      x = add_0x44(x);
      add_0x44(x);
    }
  }
  else if (option == 4000) {
    x = read_ul();
    x = add_0x44(x);
    x = prng(x,3);
    x = add_0x44(x);
    x = prng(x,3);
    x = add_0x44(x);
    x = prng(x,6);
    add_0x44(x);
  }
  else {
                    /* correct */
    if (option != 50000) goto fail;
    puts("Alright Guess The Magic Number ... : ");
    x = read_ul();
    x = add_0x44(x);
    x = prng(x,4);
    x = add_0x44(x);
    option = prng(x,8);
  }
  printf("guess is %lu \n",option);
  if (option == 0x96a878d249249) {
    win();
  }
  else {
    puts("Not quite! Keep Trying :)");
  }
  return 0;
}
```

Let's take a look at what each of these functions do:

- `add_0x44`: unsurprisingly, adds `0x44` to the value.
- `prng`: what seems to be some form of transformation. If we look at the code, it performs a specified number of iterations, where each iteration:
  - halves the number if it is even
  - multiplies it by 8 and adds one

  This is very similar to the hailstone sequence / 3x+1 sequence! There's one key difference though: if the number is odd, it will always continue to be odd. This might come in handy...
- `main`: the driver of the program. After examining a bit, we realize we have to enter `50000`, and then enter another number such that after a specific set of operations are performed, we will get `0x96a878d249249`.

We can work on reversing the operations one at a time. We'll write out everything in binary. Our target number is:

```
1001011010101000011110001101001001001001001001001001
```

The very last operation is `prng(x, 8)`. Thus, we'll reverse 8 iterations. Notice that when the last bits are `001`, we can just remove those, since we know that shifting this new value left by 3 and adding 1 will result in our current value.

```
# 1 iteration
1001011010101000011110001101001001001001001001001
# 2 iterations
1001011010101000011110001101001001001001001001
# 3 iterations
1001011010101000011110001101001001001001001
# 4 iterations
1001011010101000011110001101001001001001
# 5 iterations
1001011010101000011110001101001001001
# 6 iterations
1001011010101000011110001101001001
# 7 iterations
1001011010101000011110001101001
# 8 iterations
1001011010101000011110001101
```

Next, `main` adds `0x44`, so let's subtract it.

```
1001011010101000011101001001
```

After this is `prng(x, 4)`. We can continue the same process as before for two iterations:

```
# 1 iteration
1001011010101000011101001
# 2 iterations
1001011010101000011101
```

However, our bit string no longer ends in `001`. Thus, we have to take another approach: multiplying the number by two, so that the new value is even and would be divided by two to give our current number. This allows us to finish off the remaining iterations:

```
# 3 iterations
10010110101010000111010
# 4 iterations
100101101010100001110100
```

Finally, `main` adds `0x44` again, so we subtract it.

```
100101101010100000110000
```

Converting this to decimal gives us `9873456` (cool arrangement of digits!). Let's enter it into the server.

```
❯ nc 0.cloud.chals.io 32732
*************************************************************************
                       _                                _
 _ __ ___   __ _  __ _(_) ___     _ __  _   _ _ __ ___ | |__   ___ _ __
| '_ ` _ \ / _` |/ _` | |/ __|   | '_ \| | | | '_ ` _ \| '_ \ / _ \ '__|
| | | | | | (_| | (_| | | (__    | | | | |_| | | | | | | |_) |  __/ |
|_| |_| |_|\__,_|\__, |_|\___|___|_| |_|\__,_|_| |_| |_|_.__/ \___|_|
                 |___/      |_____|
**************************************************************************
Select your option:
50000
50000
Alright Guess The Magic Number ... :
9873456
9873456
guess is 2650405211509321
flag{y444ay_u_f0und_t43_m4gic_nUmb3r}
```
