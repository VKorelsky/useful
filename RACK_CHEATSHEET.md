# Rack Cheatsheet


## The basics 
This section inspired by following [this tutorial](https://gist.github.com/markbates/4240848)


Rack requires the following: 
-	that whatever we give to the run method respond_to `call`. 
-  that whatever we give to the run method return an array of three elements with the following signature
	- [status_code (integer), headers (hash), response (enumerable)] 

The most basic example -> in a file called `lambda_test.ru`, run with `rackup lambda_test.ru`. 
We call a lambda, which returns the desired array

```rb
run ->(env) { [200, {"Content-Type" => "text/html"}, ["Hello World!"]] }
```

Rack also gives us tools to work with requests -> 
In an example:

```rb
class HelloRack
  def call(env)
    req = Rack::Request.new(env)
    case req.path_info
    when /hello/
      [200, {"Content-Type" => "text/html"}, ["hello world"]]
    when /goodbye/
      [500, {"Content-Type" => "text/html"}, ["internal server error"]]
    else
      [404, {"Content-Type" => "text/html"}, ["not found"]]
    end
  end
end

run HelloRack.new
``` 

Scaling up, we can create a tiny web framework using Rack::Builder to register routes with blocks of code to run depending on which URL pattern is found in the request. This is spread over three files: 
	- rack file (.ru)
	- Framework class with a class method to create or return an instance of the Rack::Builder, and a plain method to register new routes (url pattern + block) with this instance
	- And an action class, which is used to evaluate the blocks of code defined for each route in context 

```rb
$:.unshift File.dirname(__FILE__)

require 'sonic'

route "/hello" do
  # params hash available to us here
  "hello #{params[:name] || "world"}"
end

route "/goodbye" do
  status 500
  "internal server error"
end

# Sonic.app returns an instance of Rack::Builder, which responds to call(env)
# When you call the instance, it builds a rack app from the routes registered with it
# When a request comes in, the instance routes it to the correct block of code to run
# The block is then run within an instance of the Action class with instance_eval
# which gives us access to the params, the status and any other methods we see fit 
run Sonic.app
```

```rb
require 'action'

class Sonic

  def self.app
    # if app does not exist, create a new instance of Rack::Builder with a default 404 route
    # state is kept across calls to self.app
    @app ||= begin

      Rack::Builder.new do
	map "/" do
	  run -> (env) { [404, {"Content-Type" => 'text/plain'}, ["Page not found!"]] }
	end
      end

    end
  end

end

def route(pattern, &block)
  # register new routes, add new routes to the instance of Rack::Builder returned by Sonic.app
  Sonic.app.map(pattern) do
    run Action.new(&block)
  end
end
```
```rb
class Action
  attr_reader :headers, :body, :request

  def initialize(&block)
    @block = block
    @status = 200
    @headers = {"Content-Type" => "text/html"}
    @body = ""
  end

  def status(value = nil)
    @status = value ? value : @status
  end

  def params
    params = request.params
    params.keys.each do |key|
      params[key.to_sym] = params[key]
      params.delete(key)
    end
  end

  def call(env)
    @request = Rack::Request.new(env)
    @body = self.instance_eval(&@block)
    [status, headers, [body]]
  end
end
```



