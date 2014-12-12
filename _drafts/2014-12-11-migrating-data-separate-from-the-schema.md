---
layout: post
title: "Not all Migrations are Equal"
date: "Thu Dec 11 18:42:07 -0500 2014"
tags: rails
---

You've been using [Active Record Migrations](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) to manage changes in
your database and you love it. But then a model's validations change, and all your existing data becomes invalid.

What do you do? Place it in an AR migration? Depends. Those are for primarily for schema migrations and this is not a schema change.

Boom: You need to run a data migration.

<!--more-->

What are your options?

## Stuff it into an AR migration? If it's simple enough.

It would probably be your first move. Sure, you can get away with a few hundred or even thousand of rows
and no one will break a sweat. Let's look at how you would do that.

### Bad Schema Migration

{% highlight ruby %}
class ChangeAdminDefaultToFalseOnUsers < ActiveRecord::Migration
  def up
    change_column_default(:users, :admin, false)
    User.reset_column_information

    # Bad: Use of application code that changes over time.
    User.update_null_to_false! 
  end
end
{%endhighlight%}

### Good Schema Migration

{% highlight ruby %}
class ChangeAdminDefaultToFalseOnUsers < ActiveRecord::Migration
  # Create empty AR model that will attach to the users table,
  # and isolate migration from application code.
  class User < ActiveRecord::Base; end

  def up
    change_column_default(:users, :admin, false)
    User.where(admin:nil).update_all(admin: false)
  end
end
{%endhighlight%}

### Better Schema Migration

{% highlight ruby %}
class ChangeAdminDefaultToFalseOnUsers < ActiveRecord::Migration
  def up
    change_column_default(:users, :admin, false)
    execute "UPDATE users SET admin = false WHERE admin IS NULL"
  end
end
{%endhighlight%}

Everything in `db/migrate/` has to live for the life
of the application, which is why using application code in an AR migration is frowned upon.

That application code will change months or even weeks from now, and then running `rake db:migrate` will be busted.

Most people get by using the **Good** and **Better** schema migration methods, but there comes a time when either the scale
or the complexity of the migration warrants its own code. The time when pure SQL will only get you so far or when the runtime of the migration
spans days not seconds.

What do you do?

## Create a one off rake task? No.

Perhaps, but the code will soon go stale, will be difficult to test, and won't automatically have mechanisms in place to roll back to changes.

**** TODO: Fill out this 'one off' rake task section.

This is where [datafix](https://github.com/Casecommons/datafix) comes in!

# Datafixes! Yes.

**** TODO: Fill out this section.

Notes:

Validations change over time
Tables and classes change over time
The migration does not change over time. It's frozen (until you nuke the migrations and only use the db:schema:load)
Datafixes also won't change over time

Uses:

Denormalize values into another table
Long running migrations that require API calls (honeybadger example)
Fix existing data to comply with new validations

### References

- [Casecommons/datafix](https://github.com/Casecommons/datafix)
- [Change data in migrations like a boss](http://railsguides.net/change-data-in-migrations-like-a-boss)
- [Zero Downtime Migrations of Large Databases - Honeybadger](https://www.honeybadger.io/blog/2013/08/06/zero-downtime-migrations-of-large-databases-using-rails-postgres-and-redis)

