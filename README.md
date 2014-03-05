# ResqueSolo

[![Gem Version](https://badge.fury.io/rb/resque_solo.png)][gem]
[![Build Status](https://api.travis-ci.org/neighborland/resque_solo.png)][build]

[gem]: http://badge.fury.io/rb/resque_solo
[build]: https://travis-ci.org/neighborland/resque_solo

ResqueSolo is a resque plugin to add unique jobs to resque.

It is a re-write of [resque-loner](https://github.com/jayniz/resque-loner).

It requires resque 1.25 and works with ruby 1.9.3 and later.

It removes the dependency on `Resque::Helpers`, which is deprecated for resque 2.0.

## Install

Add the gem to your Gemfile:

```ruby
gem 'resque_solo'
```

## Usage

```ruby
class UpdateCat
  include Resque::Plugins::UniqueJob
  @queue = :cats

  def self.perform(cat_id)
    # do something
  end
end
```

If you attempt to queue a unique job multiple times, it is ignored:

```
Resque.enqueue UpdateCat, 1
=> "OK"
Resque.enqueue UpdateCat, 1
=> "EXISTED"
Resque.enqueue UpdateCat, 1
=> "EXISTED"
Resque.size :cats
=> 1
Resque.enqueued? UpdateCat, 1
=> true
Resque.enqueued_in? :dogs, UpdateCat, 1
=> false
```

### `lock_after_execution_period`

By default, lock_after_execution_period is 0 and `enqueued?` becomes false as soon as the job
is being worked on.

The `lock_after_execution_period` setting can be used to delay when the unique job key is deleted
(i.e. when `enqueued?` becomes `false`). For example, if you have a long-running unique job that
takes around 10 seconds, and you don't want to requeue another job until you are sure it is done,
you could set `lock_after_execution_period = 20`. Or if you never want to run a long running
job more than once per minute, set `lock_after_execution_period = 60`.
