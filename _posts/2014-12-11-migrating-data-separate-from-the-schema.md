---
layout: post
title: "Not all Migrations are Equal"
date: "Thu Dec 11 18:42:07 -0500 2014"
tags: rails
---

You've been using [Active Record Migrations](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) to manage changes in
your database and you love it. But then a model's validations change, and all your existing data becomes invalid.

What do you do? Place it in an AR migration? Depends. Those are primarily for **schema migrations** and this is not a schema change.

Boom: You need to run a **data migration**.

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

Everything in **db/migrate/** has to live for the life
of the application, which is why using application code in an AR migration is frowned upon (sure, you can move onto schema:load and delete migrations, but let's keep things simple for now).

That application code will change months or even weeks from now, and then running `rake db:migrate` will be busted.

Most people get by using the **Good** and **Better** schema migration methods, but there comes a time when either the scale
or the complexity of the migration warrants its own code. The time when pure SQL will only get you so far or when the runtime of the migration
spans days not seconds.

What do you do?

## Create a one off rake task? No.

Perhaps, but the code will be difficult to test and won't have mechanisms in place to roll back to changes. Even if you refactor the logic out of
the rake task into a separate ruby class, you will now have to maintain code that is ephemeral in nature. It merely exists for this one off data migration.

One approach is to create a **oneshots.rake** file, but that ends up being a ghetto of random tasks with no test coverage that never gets cleaned up 

# [Datafixes!](https://github.com/dimroc/datafix) Yes.

Basically a mirror of AR migrations, every rails user will feel right at home with [datafixes](https://github.com/dimroc/datafix).

Install the gem from my repo:

{% highlight ruby %}
gem 'datafix', github: 'dimroc/datafix' # The changes will eventually be incorporated into the main gem `datafix`
{% endhighlight %}

Run the generator to create the datafix template:

{% highlight bash %}
> rails g datafix AddValidWholesalePriceToProducts
  create  db/datafixes/20141211143848_add_valid_wholesale_price_to_products.rb
  create  spec/db/datafixes/20141211143848_add_valid_wholesale_price_to_products_spec.rb
{% endhighlight %}

Fill out the datafix with your data migration:

{% highlight ruby %}
class Datafixes::AddValidWholesalePriceToProducts < Datafix
  def self.up
    Product.where(wholesale_price_cents: nil).each do |product|
      product.update_attributes({
        wholesale_price_cents: product.fetch_price_from_amazon
      })
    end
  end

  def self.down
  end
end
{% endhighlight %}

Then just run the rake tasks:

{% highlight bash %}
> rake db:datafix
  migrating AddValidWholesalePriceToProducts up

> rake db:datafix:status

  database: somedatabase_development

   Status   Datafix ID            Datafix Name
  --------------------------------------------------
     up    20141211143848       AddValidWholesalePriceToProducts
{% endhighlight %}

Unlike AR migrations, it generates specs:

{% highlight ruby %}
require "rails_helper"
require Rails.root.join("db", "datafixes", "20141211143848_add_valid_wholesale_price_to_products")

describe Datafixes::AddValidWholesalePriceToProducts do
  describe ".up" do
    # Fill out the describe block
    let!(:product) do
      product = FactoryGirl.build(:product, wholesale_price_cents: nil)
      product.save(validate: false)
      product
    end

    it "should fix the price and be valid" do
      expect(product).to_not be_valid
      subject.migrate('up')
      expect(product.reload).to be_valid
    end
  end
end
{% endhighlight %}

And the real kicker: when the code has overstayed its welcome, you can just delete the datafix. That's not so simple with a schema migration in **db/migrate/**. 
The datafix is ephemeral in nature and isn't worth maintaining months down the road.

This is super handy in all the scenarios:

1. Denormalizing values to another table
2. Changing data to comply with changing validations
3. Long running data migrations that span days
4. Migrating from one table to another

## Wrap Up

For data migrations, datafixes are far better than anything out there, but it's still brand new and rough around the edges. It doesn't even have
**rake db:datafix:rollback** yet! [Check it out!](https://github.com/dimroc/datafix)

### Note
*The* [**dimroc**](https://github.com/dimroc/datafix) *fork has many upgrades to the Casecommons version, including the rake tasks that function like *rake db:migrate*. It will eventually be incorporated into the Casecommons version
when they stop sending email and look at the PR.*

### References

- [Casecommons/datafix](https://github.com/Casecommons/datafix)
- [Change data in migrations like a boss](http://railsguides.net/change-data-in-migrations-like-a-boss)
- [Zero Downtime Migrations of Large Databases - Honeybadger](https://www.honeybadger.io/blog/2013/08/06/zero-downtime-migrations-of-large-databases-using-rails-postgres-and-redis)

