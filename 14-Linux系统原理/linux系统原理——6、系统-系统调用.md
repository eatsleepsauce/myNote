#### 系统调用

**glibc对系统调用的封装**，linux提供了glibc这个中介，将系统调用封装成更加友好的接口，供我们调用。

glibc中有个文件syscalls.list 里面列着所有glibc的函数对应的系统调用。

glibc还有一个脚本 make-syscall.sh，根据syscall.list里面的配置对于每个封装好的系统调用生成一个文件，这个文件里面定义了一些宏，比如 #define SYSCALL_NAME open，这个宏被glibc的另一个文件syscall-template.S使用，而宏里面定义了系统调用的调用方式，显示任何一个系统调用最终都会调用DO_CALL这个宏，对于32位和64位 DO_CALL 的定义是不一样的。

##### 一、32位系统调用

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145734.jpg" alt="32系统调用.jpg" style="zoom:67%;" />

```
/* Linux takes system call arguments in registers:
  syscall number  %eax       call-clobbered
  arg 1    %ebx       call-saved
  arg 2    %ecx       call-clobbered
  arg 3    %edx       call-clobbered
  arg 4    %esi       call-saved
  arg 5    %edi       call-saved
  arg 6    %ebp       call-saved
......
*/
#define DO_CALL(syscall_name, args)                           \
    PUSHARGS_##args                               \
    DOARGS_##args                                 \
    movl $SYS_ify (syscall_name), %eax;                          \
    ENTER_KERNEL                                  \
    POPARGS_##args
```

（1）将请求参数放到寄存器里面，根据系统调用的名称得到系统调用号，放在寄存器eax里面，然后执行ENTER_KERNEL。

（2）ENTER_KERNEL 是个宏，定义为 int  $0x80 ，int是interrupt中断的意思，int $0x80触发一个软中断，通过它可以陷入(trap)内核。

（3）内核启动的时候有个trap_init()，其中有这样的代码:

```
set_system_intr_gate(IA32_SYSCALL_VECTOR,entry_INT80_32);
```

这是一个软中断的陷入门，当接收到一个系统调用的时候，entry_INT80_32就被调用了，entry_INT80_32 ，通过push和SAVE_ALL将当前用户态的寄存器保存在pt_regs结构里面。保存所有寄存器后，然后调用 do_syscall_32_irqs_on。

```
ENTRY(entry_INT80_32)
        ASM_CLAC
        pushl   %eax                    /* pt_regs->orig_ax */
        SAVE_ALL pt_regs_ax=$-ENOSYS    /* save rest */
        movl    %esp, %eax
        call    do_syscall_32_irqs_on
.Lsyscall_32_done:
......
.Lirq_return:
  INTERRUPT_RETURN
```

（4）do_syscall_32_irqs_on 中将系统调用号从eax里面取出来，然后根据系统调用号，在系统调用表中找到相应的函数进行调用，并将寄存器中保存的参数取出来，作为函数参数。

```
static __always_inline void do_syscall_32_irqs_on(struct pt_regs *regs)
{
  struct thread_info *ti = current_thread_info();
  unsigned int nr = (unsigned int)regs->orig_ax;
......
  if (likely(nr < IA32_NR_syscalls)) {
    regs->ax = ia32_sys_call_table[nr](
      (unsigned int)regs->bx, (unsigned int)regs->cx,
      (unsigned int)regs->dx, (unsigned int)regs->si,
      (unsigned int)regs->di, (unsigned int)regs->bp);
  }
  syscall_return_slowpath(regs);
}
```

（5）系统调用结束后，回到 entry_INT80_32，在这后面紧接着调用 INTERRUPT_RETURN，也就是iret，iret指令将原来用户态保存的现场恢复回来，包括代码段、指令指针寄存器等。这时候用户态进程恢复执行。

```
#define INTERRUPT_RETURN                iret
```

##### 二、64位系统调用

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145735.jpg" alt="64系统调用.jpg" style="zoom:67%;" />

```
/* The Linux/x86-64 kernel expects the system call parameters in
   registers according to the following table:
    syscall number  rax
    arg 1    rdi
    arg 2    rsi
    arg 3    rdx
    arg 4    r10
    arg 5    r8
    arg 6    r9
......
*/
#define DO_CALL(syscall_name, args)                \
  lea SYS_ify (syscall_name), %rax;                \
  syscall
```

（1）和32位系统调用一样，还是将系统调用名称转换成系统调用号，放到寄存器rax中，不过后面进行了真正调用，该用syscall指令了，而不是使用中断了。传递参数的寄存器也变了。

（2）syscall指令还是使用了一种特殊的寄存器，**特殊模块寄存器(Model specific Register，简称 MSR)**，这种寄存器是CPU为 了完成某些特殊控制功能为目的的寄存器，其中就有系统调用。

（3）系统初始化的时候，trap_init() 除了中断模式，还有会调用 cpu_init->sys call_init。这里面有代码：

```
wrmsrl(MSR_LSTAR, (unsigned long)entry_SYSCALL_64);
```

rdmsr 和 wrmsr 是用来读写特殊模块寄存器的。MSR_LSTAR这个特殊的寄存器，当syscall指令调用的时候，会从这个寄存器里面拿出函数地址来调用，也就是调用 entry_SYSCALL_64。

entry_SYSCALL_64 先保存了很多寄存器到 pt_regs 结构里面，例如用户态的代码段、数据段、保存参数的寄存器，然后调用 entry_SYSCALL64_slow_path->do_syscall_64。

```
ENTRY(entry_SYSCALL_64)
        /* Construct struct pt_regs on stack */
        pushq   $__USER_DS                      /* pt_regs->ss */
        pushq   PER_CPU_VAR(rsp_scratch)        /* pt_regs->sp */
        pushq   %r11                            /* pt_regs->flags */
        pushq   $__USER_CS                      /* pt_regs->cs */
        pushq   %rcx                            /* pt_regs->ip */
        pushq   %rax                            /* pt_regs->orig_ax */
        pushq   %rdi                            /* pt_regs->di */
        pushq   %rsi                            /* pt_regs->si */
        pushq   %rdx                            /* pt_regs->dx */
        pushq   %rcx                            /* pt_regs->cx */
        pushq   $-ENOSYS                        /* pt_regs->ax */
        pushq   %r8                             /* pt_regs->r8 */
        pushq   %r9                             /* pt_regs->r9 */
        pushq   %r10                            /* pt_regs->r10 */
        pushq   %r11                            /* pt_regs->r11 */
        sub     $(6*8), %rsp                    /* pt_regs->bp, bx, r12-15 not saved */
        movq    PER_CPU_VAR(current_task), %r11
        testl   $_TIF_WORK_SYSCALL_ENTRY|_TIF_ALLWORK_MASK, TASK_TI_flags(%r11)
        jnz     entry_SYSCALL64_slow_path
......
entry_SYSCALL64_slow_path:
        /* IRQs are off. */
        SAVE_EXTRA_REGS
        movq    %rsp, %rdi
        call    do_syscall_64           /* returns with IRQs disabled */
return_from_SYSCALL_64:
  RESTORE_EXTRA_REGS
  TRACE_IRQS_IRETQ
  movq  RCX(%rsp), %rcx
  movq  RIP(%rsp), %r11
    movq  R11(%rsp), %r11
......
syscall_return_via_sysret:
  /* rcx and r11 are already restored (see code above) */
  RESTORE_C_REGS_EXCEPT_RCX_R11
  movq  RSP(%rsp), %rsp
  USERGS_SYSRET64
```

（4）在do_syscall_64里面，从rax里面拿出系统调用号，然后根据系统调用号，在系统调用表sys_call_table中找到相对应的函数进程调用，并将寄存器中保存的参数取出来，作为函数参数。

```
__visible void do_syscall_64(struct pt_regs *regs)
{
        struct thread_info *ti = current_thread_info();
        unsigned long nr = regs->orig_ax;
......
        if (likely((nr & __SYSCALL_MASK) < NR_syscalls)) {
                regs->ax = sys_call_table[nr & __SYSCALL_MASK](
                        regs->di, regs->si, regs->dx,
                        regs->r10, regs->r8, regs->r9);
        }
        syscall_return_slowpath(regs);
}
```

（5）系统调用返回后，执行的是USERGS_SYSRET64，返回用户态的指令变成了sysretq。

```
#define USERGS_SYSRET64        \
  swapgs;          \
  sysretq;
```

##### 三、系统调用表



##### 四、完整的64位系统调用

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145736.jpg" alt="64位系统调用全过程.jpg" style="zoom:67%;" />

