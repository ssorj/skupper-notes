# Clarity

~~~ diff
index 3eb157de..1003d53d 100644
--- a/src/server.c
+++ b/src/server.c
@@ -1506,7 +1506,7 @@ void qd_server_run(qd_dispatch_t *qd)
     sys_thread_t **threads = (sys_thread_t **)qd_calloc(n, sizeof(sys_thread_t*));
     for (i = 0; i < n; i++) {
         char thread_name[16];
-        snprintf(thread_name, sizeof(thread_name), "wrkr_%d", i);
+        snprintf(thread_name, sizeof(thread_name), "worker_thread");
         threads[i] = sys_thread(thread_name, thread_run, qd_server);
     }
~~~

## Before

~~~

~~~

## After

~~~
Samples: 3K of event 'cpu_core/cycles/', Event count (approx.): 66053343294
  Overhead  Command        Shared Object                     Symbol
+   12.33%  worker_thread  [kernel.kallsyms]                 [k] copy_user_enhanced_fast_string
+    3.72%  worker_thread  libc.so.6                         [.] __memmove_avx_unaligned_erms
     2.18%  worker_thread  [kernel.kallsyms]                 [k] tcp_sendmsg_locked
+    2.15%  worker_thread  libc.so.6                         [.] pthread_mutex_lock@@GLIBC_2.2.5
     1.90%  worker_thread  [nf_tables]                       [k] nft_do_chain
+    1.73%  worker_thread  skrouterd                         [.] message_section_check_LH
     1.43%  worker_thread  skrouterd                         [.] qd_message_send
     1.12%  client         [kernel.kallsyms]                 [k] copy_user_enhanced_fast_string
     1.08%  worker_thread  [kernel.kallsyms]                 [k] native_write_msr
     1.00%  worker_thread  [unknown]                         [.] 0x00007f73dae90100
~~~
