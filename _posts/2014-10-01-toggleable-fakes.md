---
layout: post
title: Toggleable Fakes for better integration testing
---

During the development of [iOS Project Monitor](https://github.com/dimroc/iOS.ProjectMonitor), I ran into the need
to test against a few third party services. Engineyard's Andy Delcambre gave a great talk on how to use test matrices and fake services
to write one test that hits both your fake and the service.

<!--more-->

Most people leap towards [WebMock](https://github.com/bblimke/webmock) or [VCR](https://github.com/vcr/vcr) to test against services. This is an alternate approach that is a little more
upfront work but adds incredible confidence in your code.

### Overview

* Use Sinatra to mount a `FakeThirdParty` service.

  {% highlight ruby %}

  require 'sinatra/base'

  class FakePusher < Sinatra::Base
    post "/apps/:app_id/events", provides: :json do
      JSON.parse(request.body.read)
      {}.to_json
    end

    # start the server if ruby file executed directly
    run! if app_file == $0
  end

  {% endhighlight %}

* Using WebMock, route all traffic to `www.thirdparty.com` to your mounted sinatra fake.

  {% highlight ruby %}

  require 'webmock/rspec'

  module FakeSpecHelpers
    def servers_return_healthy
      puts "WARNING: Stubbing out healthy servers"
      WebMock.stub_request(:any, /.*api.pusherapp.com\/.*/).to_rack(FakePusher)
    end

    def servers_return_error
      WebMock.stub_request(:any, /.*/).to_rack(FakeError)
    end

    def servers_return_unauthorized
      WebMock.stub_request(:any, /.*/).to_rack(FakeUnauthorized)
    end
  end

  RSpec.configure do |config|
    config.include FakeSpecHelpers

    config.before(:each) do
      if ENV["INTEGRATION"] == "true"
        WebMock.allow_net_connect!
      else
        WebMock.disable_net_connect!
        servers_return_healthy
      end
    end
  end

  {% endhighlight %}

* Using an environment variable, `INTEGRATION=true`, you can turn off the WebMock routing and have the tests hit the real service.

### Challenges

1. Performing a Tear down, or cleaning, your third party test environment can be hard, if not impossible.
2. Data from one test can then pollute other tests.

For something a little more elaborate, check out iOS Project Monitor's [Fake Parse Service](https://github.com/dimroc/iOS.ProjectMonitor/blob/master/backend/spec/fakes/fake_parse.rb) and the [WebMock Routing](https://github.com/dimroc/iOS.ProjectMonitor/blob/master/backend/spec/support/webmock.rb).

<iframe src="//player.vimeo.com/video/26510145" width="500" height="283" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="http://vimeo.com/26510145">[16M05] Toggleable Mocks and Testing Strategies in a Service Oriented Architecture (en)</a> from <a href="http://vimeo.com/rubykaigi">rubykaigi</a> on <a href="https://vimeo.com">Vimeo</a>.</p>
