---
layout: post
title: "Basic routing DSL"
published: true
ctm: true
---
# Basic routing DSL

<small>On Monday, January 20, 2014 by Darko Fabijan</small>

In previous post we saw how we can parse request paths and route patterns. Just as a remainder I will quickly sum up what we did.

We had a path that we received in HTTP request:

{% highlight elixir %}
/albums/123/photos/5
{% endhighlight %}

After parsing we turned it into

{% highlight elixir %}
[segment: "albums", segment: "123", segment: "photos", segment: "5"]
{% endhighlight %}

We also have route pattern, which is not very different from request path:

{% highlight elixir %}
/albums/:album_id/photos/:id
{% endhighlight %}

After parsing we got:

{% highlight elixir %}
[segment: "albums", binding: "album_id", segment: "photos", binding: "id"]
{% endhighlight %}

It's almost the same structure that we got after parsing path but it has *binding* elements which enabled us to match requested path to defined routes. Also keep in mind that we haven't included HTTP method in matching so far.

## Towards DSL

In this section we will see what routing DSL should produce for us so we can use it in the system that we have created so far. Pull request relevant to this post is [#2](https://github.com/thunder-ex/thunder/pull/2)

Obviously we need path pattern and HTTP method. Along with those two need controller and action that should be called upon a match. So first we will define data structure that would be nice for that use.

{% highlight elixir %}
[[method: "GET",  path: "/photos", controller: :photos, action: :index],
 [method: "POST", path: "/photos", controller: :photos, action: :create]]
{% endhighlight %}

This is the part where it started to get a bit slippery because I haven't really figured out would be easy way to aggregate data. Immutability is a concept that can hurt a bit at first. This is where three excellent posts by Saša Jurić really helped so I am referencing them here.

1. [Working with immutable data](http://www.theerlangelist.com/2013/05/working-with-immutable-data.html)
2. [Immutable programming, OO style](http://www.theerlangelist.com/2013/06/immutable-programming-oo-style.html)
3. [Immutable programming, FP style](http://www.theerlangelist.com/2013/07/immutable-programming-fp-style.html)

Along with those post [Building a Web Framework. Part I](http://elixir-lang.org/blog/2012/04/21/hello-macros/) by Alexei Sholik came very handy.

In the end I ended up with following "DSL" which is built with very only one macro and is not real DSL but it can serve as a prof of concept at this stage.

{% highlight elixir %}
defmodule ResourcesRoutesTest do
   use Thunder.Router.Drawer

   draw resources(:albums)
     |> resources(:photos)
end
{% endhighlight %}

It gives us `ResourcesRoutesTest.all` function which returns structure defined in the begging of this section. Once we return to request handler module it will be used instead of current stubbed routes. Very rough prototype like this one will be enough at the moment. This part will need a lot of love in future because I would not like to settle with anything less pleasant to use than Rails router.

If you have have experience with macros this could be an interesting exercise. Ideal routing DSL should look something like:

{% highlight elixir %}
defmodule ResourcesRoutesTest do
   use Thunder.Router.Drawer

   draw do
     resources(:albums, [only: [:index, :show]]) do
       resources(:photos, [controller: :images])
     end

     resources(:users)

     namespace(:admin) do
       resources(:users)
     end
   end
end
{% endhighlight %}

Next I will work on Thunder project generator, try to connect what we have so far together and based on that continue.
