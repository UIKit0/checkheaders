# Introduction #

This page will describe the usage of the tool.


# Walkthrough - checking a simple program #

Here is a simple example program:
```
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("hello world\n");
    return 0;
}
```

If you save that code in the file _main.c_ and then execute `checkheaders main.c` you will see this output:
```
daniel@daniel-desktop:~$ checkheaders/checkheaders main.c
Checking main.c...
[main.c:1] (style) Header not found 'stdio.h'. Use -I or --skip to fix this message.
[main.c:2] (style) Header not found 'stdlib.h'. Use -I or --skip to fix this message.
```

To fix `Header not found` messages you can use either `-I` or `--skip`. The recommendation is to use `-I` when the header is available. If you just want to suppress the messages then use `--skip stdio.h --skip stdlib.h`.

Note: you don't have to fix all `Header not found` if you don't want. The checking will continue for all the other headers.

On my computer the 'stdio.h' and 'stdlib.h' files are available in the /usr/include path so therefore I use the command `checkheaders -I /usr/include main.c`. The result is:
```
daniel@daniel-desktop:~$ checkheaders/checkheaders -I /usr/include main.c
Checking main.c...
[/usr/include/_G_config.h:14] (style) Header not found 'stddef.h'. Use -I or --skip to fix this message.
[/usr/include/stdlib.h:32] (style) Header not found 'stddef.h'. Use -I or --skip to fix this message.
progress: file main.c checking include stdio.h
progress: needed symbol 'printf'
progress: file main.c checking include stdlib.h
progress: bail out (header not found)
```

Now there are two error messages that 'stddef.h' isn't found. It is included indirectly by both stdio.h and stdlib.h on my system.

There are also 4 progress messages. These progress messages are written by the checking functionality.

The first two progress messages are:
```
progress: file main.c checking include stdio.h
progress: needed symbol 'printf'
```
It tells you that the 'stdio.h' was checked and that a needed symbol was found: 'printf'.

The last two progress messages are:
```
progress: file main.c checking include stdlib.h
progress: bail out (header not found)
```
It tells you that the 'stdlib.h' was checked. No needed symbol was found but because headers wasn't found the check bail out.

To fix the `progress: bail out (header not found)` message I can use either `--skip stddef.h` or `-I <path>`.

On my computer the 'stddef.h' is available in `/usr/include/linux` so I add another `-I`
```
checkheaders -I /usr/include -I /usr/include/linux main.c
Checking main.c...
progress: file main.c checking include stdio.h
progress: needed symbol 'printf'
progress: file main.c checking include stdlib.h
[main.c:2] (style): The included header 'stdlib.h' is not needed
```

The `[main.c:2] (style): The included header 'stdlib.h' is not needed` is written to stderr.


# System headers #

It is not a bad idea to use the normal system headers. But it can generate wrong results.
  * The standard headers are not always identical on different targets. This sometimes cause wrong reports that headers are not needed. Removing those headers would be ok on the current target but not necessarily on all other targets.
  * The standard headers contain many symbols that cause wrong matches. Checkheaders will think that the headers are needed. You might for example see `progress: needed symbol 'string'` when the sstream header is checked.

One possible solution is that you create your own system headers. By changing the given `-I` commands you can switch between the normal system headers and your own.

An example `list` header file:
```
class list { };
```

With that example file the `#include <list>` will only be necessary if the list class is used.

You can also define symbols in your system headers that are not available in the normal system headers. To get rid of false errors that system headers are not needed.