---
layout: post
title: Kernel Modules Debugging Tips (1)
date: 2018-05-28
---

Debugging kernel modules can be intimidating at first,
especially with the cryptic error messages that could result.
There are different methods that can help the debugging process,
and the easiest to start with is by figuring out the exact line of code that triggered a crash.

As an example, the kernel seemed unhappy
when I tried to access the debugfs data for the module I was working on:

<pre>
$ cat /sys/kernel/debug/dri/0/state
Killed.
</pre>

### Backtrace Report

To check out more details about the error, one way is to check the debug log in
<code>/var/log/kernel.log</code> or simply run <code>dmesg</code> in the console.

<pre>
$ dmesg
[   95.032733] BUG: unable to handle kernel NULL pointer dereference at 0000000000000000
[   95.032945] PGD 0 P4D 0
[   95.033031] Oops: 0000 [#1] SMP PTI
[   95.033136] Modules linked in: vkms drm_kms_helper drm fb_sys_fops syscopyarea sysfillrect sysimgblt snd_intel8x0 snd_ac97_codec ac97_bus snd_pcm intel_powerclamp crct10dif_pclmul crc32_pclmul ghash_clmulni_intel pcbc snd_seq_midi aesni_intel snd_seq_midi_event aes_x86_64 crypto_simd snd_rawmidi cryptd joydev glue_helper intel_rapl_perf input_leds snd_seq serio_raw snd_seq_device snd_timer snd mac_hid soundcore sch_fq_codel parport_pc ppdev lp parport ip_tables x_tables autofs4 hid_generic usbhid hid psmouse e1000 ahci video libahci i2c_piix4 pata_acpi
[   95.034221] CPU: 0 PID: 998 Comm: cat Tainted: G        W         4.17.0-rc5+ #3
[   95.034406] Hardware name: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
[   95.034620] RIP: 0010:drm_atomic_connector_print_state+0xe/0x70 [drm]
[   95.034788] RSP: 0018:ffffc0d580d63cb8 EFLAGS: 00010286
[   95.034929] RAX: ffff99ec09a56b50 RBX: ffffc0d580d63d40 RCX: 0000000000000001
[   95.035108] RDX: ffff99ec09a56b68 RSI: 0000000000000000 RDI: ffffc0d580d63d40
[   95.035305] RBP: ffffc0d580d63cd0 R08: 0000000000000002 R09: ffff99ec09a51174
[   95.035485] R10: 0000000000000000 R11: ffff99ec09a51191 R12: 0000000000000001
[   95.035719] R13: ffff99ec09a56368 R14: ffff99ec09a56000 R15: ffff99ec09a56358
[   95.035947] FS:  00007f4f0ff9b540(0000) GS:ffff99ec1fc00000(0000) knlGS:0000000000000000
[   95.036458] CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
[   95.036746] CR2: 0000000000000000 CR3: 000000010b0ce003 CR4: 00000000000606f0
[   95.036970] Call Trace:
[   95.037120]  __drm_state_dump.part.20+0x13c/0x1a0 [drm]
[   95.037301]  drm_state_info+0x59/0x80 [drm]
[   95.037460]  ? drm_get_color_range_name+0x30/0x30 [drm]
[   95.037657]  seq_read+0xe5/0x430
[   95.037801]  full_proxy_read+0x5c/0x90
[   95.037955]  __vfs_read+0x3a/0x170
[   95.038102]  ? security_file_permission+0xa0/0xb0
[   95.038307]  vfs_read+0x8e/0x130
[   95.038480]  ksys_read+0x55/0xc0
[   95.038680]  __x64_sys_read+0x1a/0x20
[   95.038875]  do_syscall_64+0x5a/0x120
[   95.039090]  entry_SYSCALL_64_after_hwframe+0x44/0xa9
[   95.039326] RIP: 0033:0x7f4f0faa9081
[   95.039486] RSP: 002b:00007fffe1d9c188 EFLAGS: 00000246 ORIG_RAX: 0000000000000000
[   95.039793] RAX: ffffffffffffffda RBX: 0000000000020000 RCX: 00007f4f0faa9081
[   95.040041] RDX: 0000000000020000 RSI: 00007f4f0ff79000 RDI: 0000000000000003
[   95.040301] RBP: 0000000000020000 R08: 00000000ffffffff R09: 0000000000000000
[   95.040529] R10: 0000000000000022 R11: 0000000000000246 R12: 00007f4f0ff79000
[   95.040756] R13: 0000000000000003 R14: 00007f4f0ff7900f R15: 0000000000020000
[   95.040987] Code: 85 c0 74 0b 48 89 de 4c 89 e7 e8 ee a5 c5 de 48 8d 65 e8 5b 41 5c 41 5d 5d c3 0f 1f 00 0f 1f 44 00 00 55 48 89 e5 41 55 41 54 53 <48> 8b 1e 49 89 f5 48 c7 c6 85 16 5d c0 49 89 fc 8b 53 28 48 8b
[   95.041587] RIP: drm_atomic_connector_print_state+0xe/0x70 [drm] RSP: ffffc0d580d63cb8
[   95.041874] CR2: 0000000000000000
[   95.042057] ---[ end trace 1d52a3ff608060d7 ]---
</pre>

From above, we can see the backtrace report that resulted from a kernel crash.

### Analysing Backtrace Report

The first line in the backtrace report tells us that the problem was caused by a NULL pointer dereference:
<div align="center"><code>BUG: unable to handle kernel NULL pointer dereference at 0000000000000000 </code></div>

The (R) in the following part refers to the state of the CPU registers (IP, SP, AX, DX, BP, CX, DI) 
[1](https://www.tutorialspoint.com/assembly_programming/assembly_registers.htm)
for 64 bits. If your machine is a 32 bits, then look for E prefix instead of R 
[2](https://wiki.ubuntu.com/DebuggingKernelOops).

<pre>
[   95.039326] RIP: 0033:0x7f4f0faa9081
[   95.039486] RSP: 002b:00007fffe1d9c188 EFLAGS: 00000246 ORIG_RAX: 0000000000000000
[   95.039793] RAX: ffffffffffffffda RBX: 0000000000020000 RCX: 00007f4f0faa9081
[   95.040041] RDX: 0000000000020000 RSI: 00007f4f0ff79000 RDI: 0000000000000003
[   95.040301] RBP: 0000000000020000 R08: 00000000ffffffff R09: 0000000000000000
[   95.040529] R10: 0000000000000022 R11: 0000000000000246 R12: 00007f4f0ff79000
[   95.040756] R13: 0000000000000003 R14: 00007f4f0ff7900f R15: 0000000000020000
</pre>

The most important one to us when debugging is RIP (Instruction Pointer),
which points to the instruction that caused the crash.

<div align="center"><code>RIP: drm_atomic_connector_print_state+0xe/0x70 [drm] RSP: ffffc0d580d63cb8</code></div>

From the value of RIP, we can see that
the error was triggered somewhere when running <code>drm_atomic_connector</code> in the drm module at <code>0xe</code> bytes into the function.

### Finding the Offensive Address Line

To find the exact line number and file where the NULL pointer derefrence happened,
we can use <code>eu-addr2line</code> tool from the <code>elfutils</code> package.

<pre>
$ sudo apt-get instal elfutils
$ eu-addr2line -e ~/kernelsource/drivers/gpu/drm/drm.o drm_atomic_connector_print_start+0xe
drivers/gpu/drm/drm_atomic.c:1300
</pre>

This will return the line number for the problematic statement and the exact file name.
Even better, we can use <code>gdb</code> and get a snippet of the code with the offensive line immediatly.
First, since the problem happened somehwere in the drm module as the <code>RIP</code> indicates, we have to load <code>drm.ko</code>
module when running the <code>gdb</code>: 

<pre>
$ gdb ~/kernelsource/drivers/gpu/drm/drm.ko
</pre>

This will start the <code>gdb</code> tool, and will load the symbols from the drm.ko module:
<div align="center"><code>Reading symbols from drivers/gpu/drm/drm.ko...done.</code></div>

In the <code>gdb</code>, we can check the line using the<code>list</code> command or <code>l</code> for short:

<pre>
(gdb) l *drm_atomic_connector_print_state+0xe
0x16a5e is in drm_atomic_connector_print_state (drivers/gpu/drm/drm_atomic.c:1300).
1295    }
1296    
1297    static void drm_atomic_connector_print_state(struct drm_printer *p,
1298            const struct drm_connector_state *state)
1299    {
1300        struct drm_connector *connector = state->connector;
1301    
1302        drm_printf(p, "connector[%u]: %s\n", connector->base.id, connector->name);
1303        drm_printf(p, "\tcrtc=%s\n", state->crtc ? state->crtc->name : "(null)");
1304    
</pre>

It seems derefrencing the variable <code>state</code> at line 1300 causes the problem, which gives us a clue that somewhere in the code, we've failed to allocate the memory for the <code>state</code>.

---

### Refrences:

[1] [Assembly - Registers](https://www.tutorialspoint.com/assembly_programming/assembly_registers.htm)

[2] [Debugging Kernel Ooops](https://wiki.ubuntu.com/DebuggingKernelOops)
