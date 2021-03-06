# SidekiqScheduler

## Note

The master branch and releases from 1.0 are compatible with sidekiq ~> 3, for sidekiq ~> 2 support
use versions ~> 0.

## Description

sidekiq-scheduler is an extension to [Sidekiq](http://github.com/mperham/sidekiq)
that adds support for running scheduled.

Scheduled jobs are like cron jobs, recurring on a regular basis.

### Documentation

This README covers what most people need to know. If you're looking for
details on individual methods, you might want to try the [rdoc](http://rdoc.info/github/moove-it/sidekiq-scheduler/master/frames).


## Installation

Add this to your Gemfile:
```ruby
gem 'sidekiq-scheduler', '~> 1'
```

If you are using rails you are set

If you are not using rails create a file with this content:
```ruby
require 'sidekiq-scheduler'
```

and the execute:
```sh
sidekiq -r created_file_path.rb
```

Look at [Loading the schedule][https://github.com/moove-it/sidekiq-scheduler/tree/0.x#loading-the-schedule]
for information on how to load your schedule.

You can add sidekiq-scheduler configuration options to sidekiq.yml config file.
Available options are:

    :schedule: <the schedule to be run>
    :dynamic: <if true the schedule can we modified in runtime>

## Scheduled Jobs (Recurring Jobs)

Scheduled (or recurring) jobs are logically no different than a standard cron
job.  They are jobs that run based on a fixed schedule which is set at
startup.

The schedule is a list of Sidekiq worker classes with arguments and a
schedule frequency (in crontab syntax).  The schedule is just a hash, but
is most likely stored in a YAML like so:

    CancelAbandonedOrders:
      cron: "*/5 * * * *"

    queue_documents_for_indexing:
      cron: "0 0 * * *"
      # you can use rufus-scheduler "every" syntax in place of cron if you prefer
      # every: 1hr
      # By default the job name (hash key) will be taken as worker class name.
      # If you want to have a different job name and class name, provide the 'class' option
      class: QueueDocuments
      queue: high
      args:
      description: "This job queues all content for indexing in solr"

    clear_leaderboards_contributors:
      cron: "30 6 * * 1"
      class: ClearLeaderboards
      queue: low
      args: contributors
      description: "This job resets the weekly leaderboard for contributions"

You can provide options to "every" or "cron" via Array:

    clear_leaderboards_moderator:
      every: ["30s", :first_in => '120s']
      class: CheckDaemon
      queue: daemons
      description: "This job will check Daemon every 30 seconds after 120 seconds after start"


NOTE: Six parameter cron's are also supported (as they supported by
rufus-scheduler which powers the sidekiq-scheduler process).  This allows you
to schedule jobs per second (ie: "30 * * * * *" would fire a job every 30
seconds past the minute).

A big shout out to [rufus-scheduler](http://github.com/jmettraux/rufus-scheduler)
for handling the heavy lifting of the actual scheduling engine.


### Loading the schedule

Let's assume your scheduled jobs are defined in a file called "config/scheduler.yml" under your Rails project,
you could create a Rails initializer called "config/initializer/scheduler.rb" which would load the job definitions:

```ruby
require 'sidekiq/scheduler'
Sidekiq.schedule = YAML.load_file(File.expand_path("../../../config/scheduler.yml",__FILE__))
```

If you were running a non Rails project you should add code to load the workers classes before loading the schedule.

```ruby
require 'sidekiq/scheduler'
Dir[File.expand_path('../lib/workers/*.rb',__FILE__)].each do |file| load file; end
Sidekiq.schedule = YAML.load_file(File.expand_path("../../../config/scheduler.yml",__FILE__))
```

You can also put your schedule information inside sidekiq.yml and load it with:

```sh
sidekiq -C ./config/sidekiq.yml
```

### Time zones

Note that if you use the cron syntax, this will be interpreted as in the server time zone
rather than the `config.time_zone` specified in Rails.

You can explicitly specify the time zone that rufus-scheduler will use:

    cron: "30 6 * * 1 Europe/Stockholm"

Also note that `config.time_zone` in Rails allows for a shorthand (e.g. "Stockholm")
that rufus-scheduler does not accept. If you write code to set the scheduler time zone
from the `config.time_zone` value, make sure it's the right format, e.g. with:

    ActiveSupport::TimeZone.find_tzinfo(Rails.configuration.time_zone).name

A future version of sidekiq-scheduler may do this for you.

## Note on Patches / Pull Requests

* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a future version unintentionally.
* Commit, do not mess with rakefile, version, or history.
  (if you want to have your own version, that is fine but bump version in a commit by itself I can ignore when I pull)
* Send me a pull request. Bonus points for topic branches.

## Credits

This work is a partial port of [resque-scheduler](https://github.com/bvandenbos/resque-scheduler) by Ben VandenBos.  
Modified to work with the Sidekiq queueing library by Morton Jonuschat.
Scheduling of recurring jobs has been added to v0.4.0, thanks to [Adrian Gomez](https://github.com/adrian-gomez).

## Maintainers

* [Morton Jonuschat](https://github.com/yabawock)
* [Adrian Gomez](https://github.com/adrian-gomez)

## License

MIT License

## Copyright

Copyright 2013 Moove-IT
Copyright 2012 Morton Jonuschat
Some parts copyright 2010 Ben VandenBos