---
date: "2014-05-10T00:00:00Z"
tags:
- Docker
- Drone
- devops
- testing
title: Continous Delivery with Drone and Dokku
---

A part of my job as CTO of [Output](http://theoutput.co/) is managing our
deployment and production systems. To this end, I've spent a lot of time
working with virtualization, Docker and automation recently. I'll try to share
some of our experiences here in the months to come. One of our biggest
challenges has been to get a good testing culture internally. To that end, I
wanted a solid solution for CI.

This seems like a solved problem. Just fire up Jenkins, and you're done.
However, there are always complications. Hard to install dependencies, Selenium
tests, required infrastructure services. That is why I have grown to love
[Drone](http://github.com/drone/drone). Run your test suite in a clean docker
image every time, cache your dependencies (just like Heroku does), and run a
deploy action of your choice if the test succed. Drone also supports various
notification mechanisms, including Email (of course), IM and our favorite
Slack.

It also has support for a host of linked services, including PostgreSQL, MySQL,
MongoDB and Redis, but writing your own is also easy if needed. I've added
support to Beanstalkd for us, and plan to contribute this back soon.

To make this even sweeter, I've coupled Drone with
[Dokku](http://dokkuapp.com/), a simple Heroku like PAAS built on top of
Docker. We follow the [Github
flow](http://scottchacon.com/2011/08/31/github-flow.html), and this setup
allows us to automatically stage all feature branches that pass their tests.

Installing dokku and drone is simple enough, and well documented in their
respective projects, so I won't cover that here. However, here is the
.drone.yml config we use in the Output web backend repository:

{{< highlight yaml >}}
image: bradrydzewski/ruby:2.1.1 
cache: 
  - /tmp/bundler
env:
  - DATABASE_URL=postgres://postgres@localhost/input_test 
  - RACK_ENV=test
  - DISPLAY=:10
services:
  - postgres
  - nordaaker/beanstalkd
notify:
  slack:
    team: nordaaker
    token: [redacted]
    channel: '#io'
    username: drone
    on_started: true
    on_success: true
    on_failure: true
script:
  - mkdir -p /tmp/bundler
  - sudo chown ubuntu:ubuntu /tmp/bundler
  - bundle install --path=/tmp/bundler
  - bundle
  - createdb input_test -E UTF8 -T template0 -U postgres -h localhost
  - bundle exec rake db:
  - Xvfb :10 -ac &
  - firefox &
  - bundle exec rake test
deploy:
  bash:
    script: 
      - 'perl deploy.pl'
{{< / highlight >}}

Let's go through it quickly; The backend is built on Padrino, and we run it
using the most recent ruby image. The cache setting is to avoid having to
reinstall all the rubygems on every test run, and then we setup ENV for the
Database connection as well as setting padrino in test mode, and the X11
display for integration tests.

Then we configure two services to be used; the built in postgres service, and
the 
beanstalkd service we built. We also set up notifications for every event to the excellent
[Slack](http://slack.com).

Next comes the test script. These are just run in sequence we setup bundler
with the cache, create the database, run migrations and then start up the
virtual X11 server. Finally we run the actual tests.

If they succeed, the deploy script is run. Drone supports many deploy scenarios
out of the box, but to get what I wanted, I wrote this little Perl script:

{{< highlight perl >}}
#!/usr/bin/env perl
system('git config --global user.name $(git --no-pager log -1 --pretty=format:\'%an\')') == 0 || die 'Could not set name';
system('git config --global user.email $(git --no-pager log -1 --pretty=format:\'%ae\')') == 0|| die 'Could not set email';
my $branch=$ENV{DRONE_BRANCH};
$branch=~s/\W//g;
die "No branch" unless $branch;
print 'Deploying to dokku@stage:'.  ( $branch eq 'master' ? 'input' : 'input-'.$branch ) ."\n";
system('git remote add deploy dokku@stage:'. 
  ( $branch eq 'master' ? 'input' : 'input-'.$branch )) == 0 || die 'Could not add target';
system('git push deploy +'.$ENV{DRONE_BRANCH}.':master') == 0 or die 'Unable to push to remote';
{{< / highlight >}}

It sets up the git environment, normalizes the current branch name,
then pushes to dokku, which either creates or updates an app based on the repo
name we push to, and deploys it. Once it's done, we get a success message in
Slack.
