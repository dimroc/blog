---
layout: post
title: "pundit-for-roles"
date: "Thu Oct 23 09:58:56 -0400 2014"
tags: rails
---

Roles are almost always a requirement for a web app. There are many good options out there, with the old guard being [CanCan](https://github.com/CanCanCommunity/cancancan)
and [Rolify](https://github.com/RolifyCommunity/rolify). But then I met this newcomer, [Pundit](https://github.com/elabs/pundit), and its
simplicity stole the show.

<!--more-->

The Rails 3 go-to was [CanCan](https://github.com/CanCanCommunity/cancancan), and I loved it.
CanCan worked great with Devise and helped encapsulate everything role related into **Abilities**.

Pundit encapsulates authorization and scoping through **Policies**, using pure ruby classes and amazingly intuitive convention over configuration.

This does mean that you will have to add `admin` (or whatever you need) columns manually, but the simplicity of it all will save you in the long run.

#### Policy

{% highlight ruby %}
class PostPolicy
  attr_reader :user, :post

  def initialize(user, post)
    @user = user
    @post = post
  end

  def update?
    user.admin? or not post.published?
  end
end
{% endhighlight %}

#### Usage in Controller

{% highlight ruby %}
def update
  @post = Post.find(params[:id])
  authorize @post
  if @post.update(post_params)
    redirect_to @post
  else
    render :edit
  end
end
{% endhighlight %}

Notice that the `Controller#update` action will automatically invoke the `Policy#update?` by convention.

Sure, this will work for actions on individual members, like **update**. But what about **index**? That's where scoping comes in:

{% highlight ruby %}
class PostPolicy < ApplicationPolicy
  class Scope
    attr_reader :user, :scope

    def initialize(user, scope)
      @user = user
      @scope = scope
    end

    def resolve
      if user.admin?
        scope.all
      else
        scope.where(:published => true)
      end
    end
  end

  def update?
    user.admin? or not post.published?
  end
end
{% endhighlight %}

The snippets here were ripped off the GitHub page, [check it out](https://github.com/elabs/pundit) for yourself It's a fantastic gem I am fully committed to using.
