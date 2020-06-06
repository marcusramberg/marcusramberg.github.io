---
date: "2014-05-27T00:00:00Z"
published: true
tags:
  - mojolicious
  - perl
  - mojoconf
title: Mojolicious::Plugin::RenderSteps
---

This weekend I attended the [MojoConf](http://mojoconf.org/) hackathon, which was great fun. I had
some interesting talks with the rest of the core team, and I collaborated with
Joel Berger on [Mojo::PG](http://github.com/marcusramberg/mojo-pg), an adaptor for the Mojo::IOLoop for Postgres.
Joel is almost done with a Pool implementation as well, and we'll probably be
on CPAN sometime this week.

I also wrote a simple plugin-helper, which I think greatly simplify working
with async controllers in Mojolicious. This what you have to do to setup async
actions in mojolicious at the moment:

{{< highlight perl >}}
#!/usr/bin/env perl
use Mojolicious::Lite;

get '/foo' => sub {
my $self = shift;
  $self->render_later;

# Concurrent requests

my $delay=Mojo::IOLoop->delay(
    sub {
      my $delay = shift;
my $url   = Mojo::URL->new('api.metacpan.org/v0/module/_search');
      $url->query({sort => 'date:desc'});
$self->ua->get($url->clone->query({q => 'mojo'}) => $delay->begin);
      $self->ua->get($url->clone->query({q => 'mango'}) => $delay->begin);
},

    # Delayed rendering
    sub {
      my ($delay, $mojo, $mango) = @_;
      $self->stash (
        mojo  => $mojo->res->json('/hits/hits/0/_source/release'),
        mango => $mango->res->json('/hits/hits/0/_source/release')
      );
      $self->render;
    }

)->catch(sub { shift->render_exception(shift) });
\$delay->wait unless Mojo::IOLoop->is_running;
{{< / highlight >}}

With the help of my new helper, that can be turned into this:

{{< highlight perl >}}
#!/usr/bin/env perl
use Mojolicious::Lite;

plugin 'RenderSteps';

get '/foo' => sub {

# Concurrent requests

my $self=shift;
  $self->render*steps(
sub {
my $delay = shift;
      my $url = Mojo::URL->new('api.metacpan.org/v0/module/\_search');
$url->query({sort => 'date:desc'});
      $self->ua->get($url->clone->query({q => 'mojo'})  => $delay->begin);
$self->ua->get($url->clone->query({q => 'mango'}) => $delay->begin);
    },
    # Automatic rendering at after last step
    sub {
      my ($delay, $mojo, $mango) = @*;
$self->stash (
        mojo  => $mojo->res->json('/hits/hits/0/\_source/release'),
mango => \$mango->res->json('/hits/hits/0/\_source/release')
);
}
);
};
{{< / highlight >}}

PS. We are looking for someone to host Mojoconf 2015, and we've donated 2000
EUR to get the next host kickstarted. Contact Oslo.pm if you're interested.
