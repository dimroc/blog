---
layout: post
title: "Arelastic for your Elasticsearch Queries pt.1"
date: "Wed Oct 29 08:45:25 -0400 2014"
tags: ruby elasticsearch
---

*Update 08/2015: [Elasticsearch-ruby has its own DSL](https://github.com/elastic/elasticsearch-ruby/tree/master/elasticsearch-dsl).
It's officially supported by Elasticsearch so be sure to check it out.*

When doing more than just a simple search with [Elasticsearch-rails](https://github.com/elasticsearch/elasticsearch-rails/tree/master/elasticsearch-model),
a naive approach will lead you to this mess:

{% highlight ruby %}
response = Article.search query:     { match:  { title: "Fox Dogs" } },
                          highlight: { fields: { title: {} } }
{% endhighlight %}

Don't be a sucker! Read on to see how [Arelastic](https://github.com/matthuhiggins/arelastic) can save you pain.

<!--more-->

If you don't know already, the Elasticsearch (ES) API takes a JSON hash when perfoming queries or filters or both.
Elasticsearch's DSL is incredibly flexibly, but can quickly lead you down the path of unmaintanable hashes.


Here is the json for the always popular **filtered query**, the search for a string after removing those outside
the filter. In this case, the filter being greater than or equal to yesterday:

{% highlight json %}
{
  "filtered": {
    "query": {
      "match": { "tweet": "full text search" }
    },
    "filter": {
      "range": { "created": { "gte": "now - 1d / d" }}
    }
  }
}
{% endhighlight %}

Now let's say you want to change the query from **"full text search"** to **"Rihanna"** (obviously). One might be tempted
to have a method that drops in a variable:

{% highlight ruby %}
def filtered_gte_query(query_term, filter_term)
  {
    filtered: {
      query: {
        match: { tweet: query_term }
      },
      filter: {
        range: { created: { gte: filter_term }}
      }
    }
  }
end
{% endhighlight %}

And now we're walking down the path to hell. Heaven forbid we want a filter than does less than or equal to, supports AND queries, or is chainable. The number
of methods and hashes will spiral out of control.

## Introducing [Arelastic](https://github.com/matthuhiggins/arelastic)

Modeled after Rails' [Arel](https://github.com/rails/arel) which is a SQL Abstract Syntax Tree (AST) manager,
Arelastic is an AST manager for Elasticsearch Queries.

Rather than working with hashes, you work with objects that represent nodes in the AST:

{% highlight ruby %}
range = Arelastic::Builders::Filter['book_id'].gteq(filter_term)
filter = Arelastic::Searches::Filter.new(range)

query = Arelastic::Queries::Match.new "tweet", query_term

dsl = Arelastic::Searches::Query.new(
  Arelastic::Queries::Filtered.new(query, filter)).as_elastic

Article.search dsl
{% endhighlight %}

Here's one interesting line of code: 
{% highlight ruby %}
Arelastic::Builders::Filter['book_id'].gteq(...)
{% endhighlight %}

The `gteq` method exists alongside many other range filters, allowing one to make a chainable API much alike ActiveRecord.
Funny enough, there exists an [ElasticRecord](https://github.com/data-axle/elastic_record) that's built on top of this,
and precedes the new official [Elasticsearch-rails](https://github.com/elasticsearch/elasticsearch-rails).

The community is behind Elasticsearch-rails, but it has yet to introduce an Arel equivalent, and we all want to move towards that.
Here's hoping we can take Arelastic and incorporate it into Elasticsearch-rails so we no longer have to tame unwieldy hashes.

Now the implementation might look cryptic, but to those familiar with ES, you can see on the ES DSL's page [here](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-filtered-query.html)
all the query and filter nodes available, and know there's an equivalent mapping in the Arelastic library. This would be the building block to an ORM like interface we're far more comfortable with.

Then we can write Elasticsearch queries like this:

{% highlight ruby %}
Gift.search_builder.filter('color' => 'red').match('flower')
{% endhighlight %}

After [my first bold attempt right here](https://gist.github.com/dimroc/2eec84498b6a35550f48) to create a chainable API using monads,
you can see how much simpler it is to create complicated Elasticsearch Queries:

###Consumption
{% highlight ruby %}
class GiftsController < ApplicationController
  def index
    search_params = SearchBuilder.
      filter(organization_id: current_organization.id).
      filter(user_id: current_user.id).
      filter(created_at: { gte: 5.days.ago }).
      sort('id desc')

    @gifts = Gift.search(search_params).page(params[:page]).records
  end
  ...
end
{% endhighlight %}

In part 2, we'll take the next step support even more ElasticSearch queries and see what it would take to make this badboy a gem.
