---
layout: post
title: "Eager Pagination with Elasticsearch-rails"
date: "Wed Nov 12 08:43:15 -0500 2014"
tags: ruby rails elasticsearch
---

So you're searching millions of records with [Elasticsearch-model](https://github.com/elasticsearch/elasticsearch-rails/tree/master/elasticsearch-model)
but want to eagerly load all associations when rendering your page for performance reasons. But wait, you're using [Kaminari](https://github.com/amatsuda/kaminari) for pagination,
and when you `includes(:association)`, you lose the pagination support.

Below, you'll see how a simple delegator will relieve your woes and eagerly load associations with support for pagination.
<!--more-->

## The Problem

#### Controller
{% highlight ruby %}
@gifts = Gift.search('flower').page(params[:page]).records.includes(:user)
{% endhighlight %}

#### View
{% highlight ruby %}
= paginate @gifts
{% endhighlight %}

#### Error!
{% highlight ruby %}
undefined method `current_page' for #<Gift::ActiveRecord_Relation:0x007fa1845de2d8>
{% endhighlight %}

## The Solution

#### Controller
{% highlight ruby %}
records = Gift.search('flower').page(params[:page]).records
@gifts = EagerPagination.new(records, :scope_with_eager_loading)
{% endhighlight %}

#### Model
{% highlight ruby %}
class Gift
  include Elasticsearch::Model
  belongs_to :user
  scope :scope_with_eager_loading, -> { includes(:user) }
end
{% endhighlight %}

#### Delegator
{% highlight ruby %}
class EagerPagination < SimpleDelegator
  attr_reader :records, :scope

  def initialize(records, scope)
    super(records)
    @records = records
    @scope = scope
  end

  def each
    records.public_send(scope).each do |r|
      yield r
    end
  end
end
{% endhighlight %}

What's going on here? `EagerPagination` overrides the `def each` method to use the eager loading scope,
but delegates all other calls to the underlying records object that still has Kaminari pagination support.

My initial solution started to dive into the [Elasticsearch-model](https://github.com/elasticsearch/elasticsearch-rails/tree/master/elasticsearch-model)
implementation, but in the end I felt that the ES gem shouldn't burden itself with the responsibilities of handing AR scopes.

I'm happy with how light this final outcome is. Simplicity is king.

Full example can be found in the gist below.

{% gist dimroc/ccc6c80c747ee8f957f3 %}
