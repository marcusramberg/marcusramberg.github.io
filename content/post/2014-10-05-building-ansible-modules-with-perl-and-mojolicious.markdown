---
date: "2014-10-05T00:00:00Z"
published: true
title: Building Ansible Modules with Perl and Mojolicious
---

While setting up [Output](http://theoutput.co/), we decided to automate all our
infrastructure using [Ansible](http://ansible.com/). This lets us easily do 
zero downtime releases and create new workers as needed to scale. We've got 
roles for haproxy, postgresql, nginx, our app servers and a lot more. One thing 
we've been handling manually so far is DNS. We're happy Cloudflare users, and 
they have one of the best DNS admin tools I've used, but still I felt like we 
could do better.

My friend [Jan Henning](http://thorsen.pm) also felt this way, and had already 
written 
[Mojo::Cloudflare](http://search.cpan.org/~jhthorsen/Mojo-Cloudflare-0.03/README.pod), 
a simple API client for Cloudflare. Even tho Ansible is Python-based, you can 
write your plugins in whatever language you like, so I decided to glue the two 
together. This turned out to be quite easy. First you need to tell ansible you 
can handle json:

```
#!/usr/bin/env perl
# WANT_JSON
```
Then we get the module argument file from the args, and parse the JSON:

```
use Mojo::JSON;
use Mojo::Util qw/slurp/;

my $json = Mojo::JSON->new;
my $args = $json->decode(slurp($ARGV[0]));
```

And then you just process those arguments and do your thing. Lets say you want 
to check for required arguments:

```
for (qw/email api_key zone name/) {
  fail("$_ is a required argument") unless exists $args->{$_};
}
```

My fail method looks like this:

```
sub fail {
  my $msg = shift;
  print $json->encode({failed => 1, message => $msg});
  exit 1;
}
```

The important thing here is to exit 1, the json response is icing on the cake. 
For success you could could do something like this:

```
print $json->encode({changed => \$changed, record => $current->id});
```

Note the bool reference to make Mojo::JSON return true/false. ```$changed``` 
will make the difference between OK and CHANGED output in your ansible run.

Put your module in a library/ sub directory of your playbook folder, and you 
can just use it in your playbook like this:

```
local_action: cloudflare zone=theoutput.co name=sentry type=CNAME 
content=sentry.nordaaker.com service_mode=1 email={{cloudflare_user}} 
api_key={{cloudflare_api_key}}
```

I store my Cloudflare credentials in an ansible-vault and just include that in 
my playbook. Note that using this module will require having the latest version 
of Mojo::Cloudflare installed. If you want to run it on the remote server 
rather than as a local_action, that server must also have the module.

Anyways, if you want to try it yourself, you can find the latest version of my 
cloudflare ansible module 
[here](https://gist.github.com/marcusramberg/1ba1ecdc8fe2a3870602).
