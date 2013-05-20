# HTTP Compression in Rails

!SLIDE

## HTTP Compression in Rails
### Caleb Woods
#### RoleModel Software

!SLIDE left

## Problem Environment

* iOS app connecting to a Rails backed API
* Large requests (response body ~ 230 kb)
* Large downloads, hundreds of requests

!SLIDE left

## Reasons for compression

* Mobile Device Client
* Low Bandwidth Environment
* Performance

!SLIDE left

## Solution

### Gzip compression

* Widely supported
* Easy to use in both Rails and iOS

!SLIDE left

## Gzipped Responses

### Rack Deflater

* Built into Rack
* `Accept-Encoding: gzip` header

```ruby
# application.rb

config.middleware.use Rack::Deflater

```

!SLIDE left

## Gzipped Requests

* Custom Mime Type
* Use ActiveSupport::Gzip to decompress the request

```ruby
# config/initializers/mime_types.rb

Mime::Type.register "gizp/json", :gzipjson

ActionDispatch::ParamsParser::DEFAULT_PARSERS[Mime::Type.lookup('gzip/json')]=lambda do |raw_body|
  body = ActiveSupport::Gzip.decompress(raw_body)
  JSON.parse(body).with_indifferent_access
end

```
<!-- Sources -->
<!-- http://schneems.com/post/16765433402/hi-im-looking-for-the-correct-way-to-convert-a-gzip -->
<!-- http://stackoverflow.com/questions/8700332/rails-3-and-json-default-renderer-but-custom-mime-type -->

!SLIDE

## Testing

```ruby
# rack_deflater_spec.rb

require 'spec_helper'

describe Rack::Deflater do
  include_context 'API'

  it "encodes response if requested" do
    api_get "posts"
    response.headers["Content-Encoding"].should be_nil
    content_length = response.headers["Content-Length"].to_i
    api_get_gzipped "posts"
    response.headers["Content-Length"].to_i.should <= content_length
    response.headers["Content-Encoding"].should == "gzip"
  end

  it "accepts gzip encoded data" do
    api_post_gziped "posts", post: { name: 'Compressed' }
    response.headers["Content-Encoding"].should == "gzip"
    response.status.should eq 201
    decompressed_json_body.should include 'name' => 'Compressed'
  end
end

```
Test helper methods gist: [https://gist.github.com/calebwoods/5615260](https://gist.github.com/calebwoods/5615260)

!SLIDE left

## Results

* ~90% response size decrease
* 40% speed increase iOS app large downloads

!SLIDE left

## iOS Tools

* [AFNetworking](https://github.com/AFNetworking/AFNetworking)
  * Automatically decompresses Gzipped responses
* [Godzippa](https://github.com/mattt/Godzippa)
  * Must be properly configured to used Gzip

!SLIDE left

## Feedback

* Twitter: [@calebwoods](https://twitter.com/calebwoods)
* Repo: [github.com/calebwoods/http-compression-rails](https://github.com/calebwoods/http-compression-rails)
* Email: [caleb.woods@rolemodelsoftware.com](cmailto:caleb.woods@rolemodelsoftware.com)
