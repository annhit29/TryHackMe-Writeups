This is the [TryHackMe Compiled challenge](https://tryhackme.com/room/compiled).

We download the task files.

We know the `.compiled` files in CTF are binary files, often an ELF (Executable and Linkable Format) files on Linux systems, which we do reverse engineering to find the flag.

Thus, in the terminal, we do:
```strings Compiled-1688545393558.Compiled | less```
where 
- `strings` scans the file for any sequences of printable characters and prints them out
- `| less`: use it such that instead of dumping hundreds or thousands of lines to the terminal all at once, `less` lets us scroll through page by page (space to go forward, b to go back, `/` to search, q to quit).

Here, since there are not many lines in this file, we can do:
```strings Compiled-1688545393558.Compiled```

We see a hint:
```
Password: 
DoYouEven%sCTF

Correct!
Try again!
```

We know the `%s` format specifier is used in functions like `printf()` and `scanf()` to handle strings.
- With `scanf()`, `%s` is used to read a string from the input. It reads characters until it encounters whitespace, and stores them in the provided character array.
- With `printf()`, `%s` is replaced by the string value provided.

Thus, we make this binary file executable and run it:
```
chmod +x Compiled-1688545393558.Compiled
./Compiled-1688545393558.Compiled
```

We run it with different inputs:
```
~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven%
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: %p%p%p
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: %x
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven%p%p%pCTF
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: 
Try again!
```

So we kind of smell that we need to use Ghidra to get the correct answer!
- [Install Ghidra on Ubuntu 24.04 Linux](https://youtu.be/wbTxAKkGSk0?si=rZsfmgKx6AUeGCF8)
- [Ghidra quickstart & tutorial: watch the Project manager and the Code browser sections](https://youtu.be/fTGTnrgjuGA?t=35&si=Uw0KBETjZOJ6bb17)

\
We decompile the binary file using Ghidra and we see:
```c

undefined8 main(void)

{
  int iVar1;
  char local_28 [32];
  
  fwrite("Password: ",1,10,stdout);
  __isoc99_scanf("DoYouEven%sCTF",local_28);
  iVar1 = strcmp(local_28,"__dso_handle");
  if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
    printf("Try again!");
    return 0;
  }
  iVar1 = strcmp(local_28,"_init");
  if (iVar1 == 0) {
    printf("Correct!");
  }
  else {
    printf("Try again!");
  }
  return 0;
}
```

From the main function, we see:
the password consists of
```c
  __isoc99_scanf("DoYouEven%sCTF",local_28);
```
and
```c
  iVar1 = strcmp(local_28,"_init");
  if (iVar1 == 0) {
    printf("Correct!");
  }
```
so the only “correct” secret string is `_init` (for the `%s`).\
If we give `__dso_handle` or other strings (for the `%s`), they will always output us "Try again!"

\
Therefore, we try again on our terminal:
```
~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven_initCTF
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven_initCTF
Try again!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven_init   
Correct!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven
_init
Correct!~/try_hack_me_stuff/challenges/compiled$ ./Compiled-1688545393558.Compiled
Password: DoYouEven




_init
Correct!
```

So we've found the flag: `DoYouEven_init`