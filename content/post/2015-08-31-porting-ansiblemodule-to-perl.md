---
date: "2015-08-31T00:00:00Z"
published: true
title: Porting AnsibleModule to Perl.
---

    “All animals are equal, but some animals are more equal than others.”
    ― George Orwell, Animal Farm

Ansible is a great orchestration tool, which I've used extensively for the last years to build infrastructure, or even to accomplish routine tasks like restarting a group of web servers one by one or patching a bunch of servers for critical security issues.

Ansible is written in Python,  but you can write modules in anything that can  generate JSON. In fact, I wrote about [making a Perl based cloudflare module]() earlier. However, to quote [the official documentation]():

    As mentioned, if you are writing a module in Python, there are some very powerful shortcuts you can use. Modules are still transferred as one file, but an arguments file is no longer needed, so these are not only shorter in terms of code, they are actually FASTER in terms of execution time.

Python is a fine language, but I'm more efficient at writing Perl, so I set out to port AnsibleModule to Perl. However, as I started looking at this new style of argument parsing, I got increasingly horrified. Seems that native Ansible modules are pre-processed to insert a lot of boilerplate code, including the passed arguments passed as a Python struct. Parsing this isn't really feasible from any other language than Python, so I decided to use the JSON support for non-native modules instead. Unfortunately, this argument passing still happens by passing a filename on the command line to the module, and the module slurping this in. Ansible also slurps in the entire file into memory to look for the WANT_JSON flag.

In the future I would love to see the module runner checking if the module opens STDIN, and pass the  the module arguments directly as a JSON struct. This seems like the most efficient and agnostic way to go. Reliably detecting if stdin is open over ssh [might be troublesome](http://stackoverflow.com/questions/911168/how-to-detect-if-my-shell-script-is-running-through-a-pipe), but even an explicit way of setting JSON_MODE (ENV?) would be an improvement over today's situation.

Using my AnsibleModule port still provides quite a bit of convenience though. For instance, you can do complex argument checking easily:

``` 
  my $m = AnsibleModule->new(
    argument_spec => {
      name    => {required => 1},
      enabled => {default  => 0, aliases => [qw/turned_on active/]},
      type    => {type => choices => [qw/one two three/]},
      similar => {type => 'dict'}
    },
    required_one_of => [qw/type similar/]
  );
```

Note that due to the malleability of Perl, we don't really care about typing beyond String/List/Dict and Bool (which supports the same values as the Python AnsibleModule bools). Everything else is treated as a scalar.

AnsibleModule also helps with supporting check mode in your module, just like the original Python module. I also provide [Test::AnsibleModule]() which lets you simply write unit tests for your module. As a convenience, you can test this module directly through the ansible-perl script command like this:

```
   ansible-perl -p -m <module> -a '{"foo":"bar"}'
```

Use -h to see all the available options. The -p flag means ansible-perl uses [App::Fatpacker](https://metacpan.org/pod/App::FatPacker) to help you pack up your dependencies inside your Ansible module (as long as they are pure Perl). [Mojolicious](http://mojolicio.us/) is fat-packable, and provides a great basis for writing HTTP-related Ansible modules. Incidentally I used it to write AnsibleModule as well.

As a bonus, if you fatpack your modules, you will automatically include the WANT_JSON text that Ansible is looking for in your module source code. Note that if you don't want to pack your module, you will have to include it yourself somewhere in your source code.

Once you've packed your module, you can just put it into library/ in your Ansible project, and use it directly. As long as your hosts have Perl installed, everything should just work.

I'm still putting the finishing touches on the module for the cpan 0.1 release, but meanwhile you can get it from [GitHub](http://github.com/marcusramberg/AnsibleModule). Please create an issue or make a pull request if you find any bugs or you miss any features. It might also be useful to look at the [examples](https://github.com/marcusramberg/AnsibleModule/tree/master/t/ext) used in the unit tests.

