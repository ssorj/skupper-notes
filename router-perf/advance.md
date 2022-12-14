# Advance

The function `advance` in message.c is prominent in profiling.

## Current code

~~~ c
static bool can_advance(unsigned char **cursor, qd_buffer_t **buffer)
{
    if (qd_buffer_cursor(*buffer) > *cursor)
        return true;

    if (DEQ_NEXT(*buffer)) {
        *buffer = DEQ_NEXT(*buffer);
        *cursor = qd_buffer_base(*buffer);
    }

    return qd_buffer_cursor(*buffer) > *cursor;
}

static bool advance(unsigned char **cursor, qd_buffer_t **buffer, int consume)
{
    if (!can_advance(cursor, buffer))
        return false;

    unsigned char *local_cursor = *cursor;
    qd_buffer_t   *local_buffer = *buffer;

    int remaining = qd_buffer_cursor(local_buffer) - local_cursor;
    while (consume > 0) {
        if (consume <= remaining) {
            local_cursor += consume;
            consume = 0;
        } else {
            if (!local_buffer->next)
                return false;

            consume -= remaining;
            local_buffer = local_buffer->next;
            local_cursor = qd_buffer_base(local_buffer);
            remaining = qd_buffer_size(local_buffer);
        }
    }

    *cursor = local_cursor;
    *buffer = local_buffer;

    return true;
}
~~~

### Disassembly

~~~
0000000000459ba0 <advance>:
  459ba0:	             48 8b 06             	mov    rax,QWORD PTR [rsi]
  459ba3:	             4c 8b 07             	mov    r8,QWORD PTR [rdi]
  459ba6:	             8b 48 10             	mov    ecx,DWORD PTR [rax+0x10]
  459ba9:	             48 8d 4c 08 18       	lea    rcx,[rax+rcx*1+0x18]
  459bae:	             49 39 c8             	cmp    r8,rcx
  459bb1:	         /-- 72 2d                	jb     459be0 <advance+0x40>
  459bb3:	         |   48 8b 40 08          	mov    rax,QWORD PTR [rax+0x8]
  459bb7:	         |   48 85 c0             	test   rax,rax
  459bba:	/--------|-- 74 17                	je     459bd3 <advance+0x33>
  459bbc:	|        |   8b 48 10             	mov    ecx,DWORD PTR [rax+0x10]
  459bbf:	|        |   4c 8d 40 18          	lea    r8,[rax+0x18]
  459bc3:	|        |   48 89 06             	mov    QWORD PTR [rsi],rax
  459bc6:	|        |   4c 89 07             	mov    QWORD PTR [rdi],r8
  459bc9:	|        |   48 8d 4c 08 18       	lea    rcx,[rax+rcx*1+0x18]
  459bce:	|        |   49 39 c8             	cmp    r8,rcx
  459bd1:	|        +-- 72 0d                	jb     459be0 <advance+0x40>
  459bd3:	>--------|-> 31 c0                	xor    eax,eax
  459bd5:	|        |   c3                   	ret
  459bd6:	|        |   66 2e 0f 1f 84 00 00 00 00 00 	cs nop WORD PTR [rax+rax*1+0x0]
  459be0:	|        \-> 44 29 c1             	sub    ecx,r8d
  459be3:	|            85 d2                	test   edx,edx
  459be5:	|        /-- 7e 13                	jle    459bfa <advance+0x5a>
  459be7:	|        |   66 0f 1f 84 00 00 00 00 00 	nop    WORD PTR [rax+rax*1+0x0]
  459bf0:	|  /-----|-> 39 ca                	cmp    edx,ecx
  459bf2:	|  |  /--|-- 7f 1c                	jg     459c10 <advance+0x70>
  459bf4:	|  |  |  |   48 63 d2             	movsxd rdx,edx
  459bf7:	|  |  |  |   49 01 d0             	add    r8,rdx
  459bfa:	|  |  |  \-> 4c 89 07             	mov    QWORD PTR [rdi],r8
  459bfd:	|  |  |      48 89 06             	mov    QWORD PTR [rsi],rax
  459c00:	|  |  |      b8 01 00 00 00       	mov    eax,0x1
  459c05:	|  |  |      c3                   	ret
  459c06:	|  |  |      66 2e 0f 1f 84 00 00 00 00 00 	cs nop WORD PTR [rax+rax*1+0x0]
  459c10:	|  |  \----> 48 8b 40 08          	mov    rax,QWORD PTR [rax+0x8]
  459c14:	|  |         48 85 c0             	test   rax,rax
  459c17:	\--|-------- 74 ba                	je     459bd3 <advance+0x33>
  459c19:	   |         29 ca                	sub    edx,ecx
  459c1b:	   |         4c 8d 40 18          	lea    r8,[rax+0x18]
  459c1f:	   |         8b 48 10             	mov    ecx,DWORD PTR [rax+0x10]
  459c22:	   \-------- eb cc                	jmp    459bf0 <advance+0x50>
~~~

### Profile

~~~
Samples: 30K of event 'cpu_core/cycles/', Event count (approx.): 126019144859
  Overhead  Command       Shared Object                     Symbol
+    3.30%  wrkr_0        [kernel.kallsyms]                 [k] copy_user_enhan
+    3.22%  client        [kernel.kallsyms]                 [k] copy_user_enhan
+    3.17%  wrkr_2        [kernel.kallsyms]                 [k] copy_user_enhan
+    3.17%  server        [kernel.kallsyms]                 [k] copy_user_enhan
+    3.11%  wrkr_1        [kernel.kallsyms]                 [k] copy_user_enhan
+    2.90%  wrkr_3        [kernel.kallsyms]                 [k] copy_user_enhan
+    2.63%  wrkr_2        libc.so.6                         [.] __memmove_avx_u
+    2.59%  wrkr_3        libc.so.6                         [.] __memmove_avx_u
+    2.52%  wrkr_1        libc.so.6                         [.] __memmove_avx_u
+    2.50%  wrkr_0        libc.so.6                         [.] __memmove_avx_u
+    1.41%  wrkr_0        libc.so.6                         [.] pthread_mutex_l
+    1.34%  wrkr_2        libc.so.6                         [.] pthread_mutex_l
+    1.29%  wrkr_1        libc.so.6                         [.] pthread_mutex_l
+    1.23%  wrkr_3        libc.so.6                         [.] pthread_mutex_l
+    0.89%  wrkr_3        skrouterd                         [.] advance
+    0.83%  wrkr_1        skrouterd                         [.] advance
+    0.81%  wrkr_2        skrouterd                         [.] advance
+    0.80%  wrkr_0        skrouterd                         [.] advance
+    0.63%  wrkr_0        [nf_tables]                       [k] nft_do_chain
+    0.60%  wrkr_1        [nf_tables]                       [k] nft_do_chain
+    0.58%  wrkr_3        [nf_tables]                       [k] nft_do_chain
+    0.52%  wrkr_2        [nf_tables]                       [k] nft_do_chain
+    0.51%  wrkr_0        [kernel.kallsyms]                 [k] tcp_sendmsg_loc
~~~

## Proposed code

~~~ c
static bool advance(unsigned char **cursor, qd_buffer_t **buffer, int consume)
{
    int remaining = qd_buffer_cursor(*buffer) - *cursor;

    while (consume > 0) {
        if (consume <= remaining) {
            *cursor += consume;
            consume = 0;
        } else if (DEQ_NEXT(*buffer)) {
            consume -= remaining;
            *buffer = DEQ_NEXT(*buffer);
            *cursor = qd_buffer_base(*buffer);
            remaining = qd_buffer_size(*buffer);
        } else {
            return false;
        }
    }

    return true;
}
~~~

Things to note:

* This proposed impl doesn't copy the cursor and buffer.  I'm not sure
  why the original does.  The original *does* mutate buffer and
  cursor, inside can_advance.
* It doesn't check the can_advance preconditions, on the idea that the
  loop is already doing it.
* It passes the tests.
* There may be some small benefit in inlining this.

### Disassembly

~~~
0000000000459670 <advance>:
  459670:	             48 8b 06             	mov    rax,QWORD PTR [rsi]
  459673:	             4c 8b 07             	mov    r8,QWORD PTR [rdi]
  459676:	             8b 48 10             	mov    ecx,DWORD PTR [rax+0x10]
  459679:	             4c 8d 4c 08 18       	lea    r9,[rax+rcx*1+0x18]
  45967e:	             4d 29 c1             	sub    r9,r8
  459681:	             44 89 c9             	mov    ecx,r9d
  459684:	             85 d2                	test   edx,edx
  459686:	/----------- 7e 31                	jle    4596b9 <advance+0x49>
  459688:	|            41 39 d1             	cmp    r9d,edx
  45968b:	|        /-- 7c 16                	jl     4596a3 <advance+0x33>
  45968d:	|  /-----|-- eb 21                	jmp    4596b0 <advance+0x40>
  45968f:	|  |     |   90                   	nop
  459690:	|  |  /--|-> 29 ca                	sub    edx,ecx
  459692:	|  |  |  |   8b 48 10             	mov    ecx,DWORD PTR [rax+0x10]
  459695:	|  |  |  |   4c 8d 40 18          	lea    r8,[rax+0x18]
  459699:	|  |  |  |   48 89 06             	mov    QWORD PTR [rsi],rax
  45969c:	|  |  |  |   4c 89 07             	mov    QWORD PTR [rdi],r8
  45969f:	|  |  |  |   39 ca                	cmp    edx,ecx
  4596a1:	|  +--|--|-- 7e 0d                	jle    4596b0 <advance+0x40>
  4596a3:	|  |  |  \-> 48 8b 40 08          	mov    rax,QWORD PTR [rax+0x8]
  4596a7:	|  |  |      48 85 c0             	test   rax,rax
  4596aa:	|  |  \----- 75 e4                	jne    459690 <advance+0x20>
  4596ac:	|  |         31 c0                	xor    eax,eax
  4596ae:	|  |         c3                   	ret
  4596af:	|  |         90                   	nop
  4596b0:	|  \-------> 48 63 d2             	movsxd rdx,edx
  4596b3:	|            49 01 d0             	add    r8,rdx
  4596b6:	|            4c 89 07             	mov    QWORD PTR [rdi],r8
  4596b9:	\----------> b8 01 00 00 00       	mov    eax,0x1
  4596be:	             c3                   	ret
~~~

### Profile

`advance()` disappears because the compiler inlines the now simpler
implementation into `message_section_check_LH()`.

~~~
Samples: 31K of event 'cpu_core/cycles/', Event count (approx.): 127843274818
  Overhead  Command       Shared Object                     Symbol
+    3.21%  server        [kernel.kallsyms]                 [k] copy_user_enha???
+    3.17%  wrkr_3        [kernel.kallsyms]                 [k] copy_user_enha???
+    3.10%  client        [kernel.kallsyms]                 [k] copy_user_enha???
+    3.09%  wrkr_1        [kernel.kallsyms]                 [k] copy_user_enha???
+    3.06%  wrkr_2        [kernel.kallsyms]                 [k] copy_user_enha???
+    2.97%  wrkr_0        [kernel.kallsyms]                 [k] copy_user_enha???
+    2.43%  wrkr_1        libc.so.6                         [.] __memmove_avx_???
+    2.43%  wrkr_0        libc.so.6                         [.] __memmove_avx_???
+    2.40%  wrkr_3        libc.so.6                         [.] __memmove_avx_???
+    2.31%  wrkr_2        libc.so.6                         [.] __memmove_avx_???
+    1.37%  wrkr_2        libc.so.6                         [.] pthread_mutex_???
+    1.37%  wrkr_1        libc.so.6                         [.] pthread_mutex_???
+    1.36%  wrkr_3        libc.so.6                         [.] pthread_mutex_???
+    1.30%  wrkr_0        libc.so.6                         [.] pthread_mutex_???
+    0.89%  wrkr_0        skrouterd                         [.] message_sectio???
+    0.87%  wrkr_1        skrouterd                         [.] message_sectio???
+    0.81%  wrkr_2        skrouterd                         [.] message_sectio???
+    0.79%  wrkr_3        skrouterd                         [.] message_sectio???
+    0.64%  wrkr_3        [nf_tables]                       [k] nft_do_chain  ???
+    0.59%  wrkr_0        [nf_tables]                       [k] nft_do_chain  ???
+    0.58%  wrkr_1        [nf_tables]                       [k] nft_do_chain  ???
+    0.57%  wrkr_3        [kernel.kallsyms]                 [k] tcp_sendmsg_lo???
+    0.55%  wrkr_1        [kernel.kallsyms]                 [k] tcp_sendmsg_lo???
+    0.54%  wrkr_2        [nf_tables]                       [k] nft_do_chain  ???
~~~

## Related

Note in passing: `advance_guarded()` has this:

~~~ c
    while (consume > 0) {
        if (consume < remaining) {
~~~

Is that a bug?  Should that be `<=`?

A new `next_octet()` impl:

~~~ c
static inline bool next_octet(unsigned char **cursor, qd_buffer_t **buffer, unsigned char *octet)
{
    if (qd_buffer_cursor(*buffer) > *cursor) {
        *octet = **cursor;
    } else if (DEQ_NEXT(*buffer)) {
        *octet = *qd_buffer_base(DEQ_NEXT(*buffer));
    } else {
        return false;
    }

    advance(cursor, buffer, 1);

    return true;
}
~~~
