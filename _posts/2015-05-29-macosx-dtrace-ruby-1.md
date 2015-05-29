---
layout: post
title:  "Running DTrace in Ruby on MacOS X"
date:   2015-05-29 07:19:13
categories: osx, dtrace, ruby
---
Since version 2 Ruby comes with [DTrace](http://dtrace.org/blogs/about/) probes, 
which lets you dynamically instrument a running program.

Unfortunately the Ruby shipped with MacOS X doesn't have DTrace enabled, when you try to list the available probes,
you'll see this:

    martin$ sudo dtrace  -c "ruby -v" -l -m ruby
       ID   PROVIDER            MODULE                          FUNCTION NAME
    dtrace: failed to match :ruby::: No probe matches description


So you need to install one that has. I perfer `rbenv` and how to install it is documented
[here](https://github.com/sstephenson/rbenv) so I am not going to show how do it. 
After a successful install you should see this:

    martin$ sudo dtrace  -c "`rbenv which ruby` -v" -l -m ruby
       ID   PROVIDER            MODULE                          FUNCTION NAME
    341053  ruby87551              ruby                       rb_ary_drop array-create
    341054  ruby87551              ruby                    rb_ary_product array-create
    341055  ruby87551              ruby       rb_ary_repeated_combination array-create
    341056  ruby87551              ruby       rb_ary_repeated_permutation array-create
    341057  ruby87551              ruby                rb_ary_combination array-create
    341058  ruby87551              ruby                rb_ary_permutation array-create
    341059  ruby87551              ruby                     rb_ary_sample array-create
    341060  ruby87551              ruby                        rb_ary_and array-create
    341061  ruby87551              ruby                       rb_ary_diff array-create
    341062  ruby87551              ruby                      rb_ary_times array-create
    341063  ruby87551              ruby                 rb_ary_slice_bang array-create
    341064  ruby87551              ruby                     rb_ary_reject array-create
    341065  ruby87551              ruby                   empty_ary_alloc array-create
    341066  ruby87551              ruby                     rb_ary_subseq array-create
    341067  ruby87551              ruby                        rb_ary_new array-create
    341068  ruby87551              ruby                           ary_new array-create
    341069  ruby87551              ruby                     vm_call_cfunc cmethod-entry
    341070  ruby87551              ruby                     vm_call0_body cmethod-entry
    341071  ruby87551              ruby                      vm_exec_core cmethod-entry
    341072  ruby87551              ruby                     vm_call_cfunc cmethod-return
    341073  ruby87551              ruby                     vm_call0_body cmethod-return
    341074  ruby87551              ruby             rb_vm_pop_cfunc_frame cmethod-return
    341075  ruby87551              ruby                      vm_exec_core cmethod-return
    341076  ruby87551              ruby               rb_require_internal find-require-entry
    341077  ruby87551              ruby               rb_require_internal find-require-return
    341078  ruby87551              ruby                          gc_start gc-mark-begin
    341079  ruby87551              ruby                          gc_start gc-mark-end
    341080  ruby87551              ruby                     gc_sweep_step gc-sweep-begin
    341081  ruby87551              ruby                     gc_sweep_step gc-sweep-end
    341082  ruby87551              ruby              m_core_hash_from_ary hash-create
    341083  ruby87551              ruby                      vm_exec_core hash-create
    341084  ruby87551              ruby                  empty_hash_alloc hash-create
    341085  ruby87551              ruby                         rb_f_load load-entry
    341086  ruby87551              ruby                         rb_f_load load-return
    341087  ruby87551              ruby    rb_clear_method_cache_by_class method-cache-clear
    341088  ruby87551              ruby               invoke_block_from_c method-entry
    341089  ruby87551              ruby                      vm_exec_core method-entry
    341090  ruby87551              ruby               invoke_block_from_c method-return
    341091  ruby87551              ruby                           vm_exec method-return
    341092  ruby87551              ruby                      vm_exec_core method-return
    341093  ruby87551              ruby                      rb_obj_alloc object-create
    341094  ruby87551              ruby                        yycompile0 parse-begin
    341095  ruby87551              ruby                        yycompile0 parse-end
    341096  ruby87551              ruby                   setup_exception raise
    341097  ruby87551              ruby               rb_require_internal require-entry
    341098  ruby87551              ruby               rb_require_internal require-return
    341099  ruby87551              ruby                   empty_str_alloc string-create
    341100  ruby87551              ruby                  rb_str_resurrect string-create
    341101  ruby87551              ruby                    str_new_static string-create
    341102  ruby87551              ruby                          str_new0 string-create
    341103  ruby87551              ruby         register_static_symid_str symbol-create
    341104  ruby87551              ruby                     dsymbol_alloc symbol-create

DTrace require elevated privileges and thus needs to be run as root, and to avoid having to type your password
over and over again, you can update `/etc/sudoers` (using `visudo` off course) and add the below line,
which will let you use DTrace without a password: 

    %admin ALL = (root) NOPASSWD: /usr/sbin/dtrace

# Trying it out

Now that we have a Ruby version with DTrace probes, we can use it to e.g. see how many object creations are involved
in printing a "hello world" message in.

First create a file called `hello.rb`

```ruby
puts "hello world from Ruby #{RUBY_VERSION}!"
```

Next create `count.d` which will count the number of times a new object is created, grouped by the class.
The `copyinstr()` is needed to convert `arg0`, which is the class name, from user space to kernel space where
the DTrace probes are run.

```dtrace
ruby*:::object-create
{
    @objects[copyinstr(arg0)] = count();
}
```

If you forget to use `copyinstr()`, you'll get a number like `140576469837096` instead of `String`.

When you run this, you should get:

    martin$ sudo dtrace  -s count.d -c "`rbenv which ruby` `pwd`/hello.rb"
    dtrace: script 'count.d' matched 2 probes
    hello world from Ruby 2.2.1!
    dtrace: pid 87601 has exited
    
      ARGF.class                                                        1
      Gem::Platform                                                     1
      IOError                                                           1
      Monitor                                                           1
      NoMemoryError                                                     1
      Range                                                             1
      SystemStackError                                                  1
      ThreadGroup                                                       1
      Time                                                              1
      fatal                                                             1
      LoadError                                                         2
      Mutex                                                             3
      Object                                                            3
      Gem::Specification                                                6
      Hash                                                              8
      Gem::Version                                                      9
      Gem::Requirement                                                 19
      Array                                                            72
      String                                                          254

As you can see, there are a lot of String objects created!