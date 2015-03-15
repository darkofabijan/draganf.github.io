---
layout: post
title: "HTTP server with cowboy"
published: true
ctm: true
---
# HTTP server with cowboy

<small>On Sunday, January 19, 2014 by Darko Fabijan</small>

As a first step I wanted to have working HTTP server. After a bit of research it turned out that [cowboy](https://github.com/extend/cowboy) is a good library to go with. It is well maintained and has support for Websockets, chunked responses, SPDY, and so on. Documentation is also quite nice and I really liked [Cowboy User Guide](http://ninenines.eu/docs/ en/cowboy/HEAD/guide/) which is still work in progress but has enough information already to get you started with cowboy.

The [initial commit](https://github.com/thunder-ex/thunder/commit/b977e5a88c42a4caa7eff951ccfabf1efda48775) is just freshly generated Elixir application. In the [second commit](https://github.com/thunder-ex/thunder/commit/b64e872faa4ac66c454e5cd6bed14ee6f5fd1aa3) I have added cowboy. [John Rotenberg](https://github.com/joshrotenberg) translated cowboy examples into Elixir and the [repository](https://github.com/joshrotenberg/elixir_cowboy_examples) is available on GitHub. So introducing cowboy was pretty much reorganising his hello world example.

I wrapped cowboy in `Thunder.HttpServer` module so only [one line](https://github.com/thunder-ex/thunder/commit/7a33a36d6313f8904089afa374b5c51694c3192c#diff-ccbbcc6457dfea4c7a0787e60b0dd04fR7) `Thunder.HttpServer.run` needs to be added to our main application file. And since cowboy is separate application it needs to be started with our app and that's done by adding [`applications [:cowboy]` to mix file.

The most important part is `Thunder.RequestHandler`. This is the entry point of a request into our application. In `handle(req, state)` function we currently just extract request path, compose response message and call `reply` function on cowboy request. All the interesting things will be happening in this function. It's `int main()` of framework (it's just my C background speaking).

{% highlight elixir %}
defmodule Thunder.RequestHandler do

  def init({_any, :http}, req, []) do
    {:ok, req, :undefined}
  end

  def handle(req, state) do
    {path, _} = :cowboy_req.path(req)

    response = "You have requested: #{path}"

    :cowboy_req.reply(200, [], response, req)
    {:ok, req, state}
  end

  def terminate(_reason, _req, _state) do
    :ok
  end

end
{% endhighlight %}

For the following posts I plan to create pull requests so it would be issier to reference and discuss code.
