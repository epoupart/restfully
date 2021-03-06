# Restfully
Restfully is a general-purpose client library for RESTful APIs. It is written in Ruby. Its goal is to abstract the nitty-gritty details of exchanging HTTP requests between the user-agent and the server. It also discovers resources at runtime, which means should the API change and add a new functionality, the client will automatically discover it.

It works on simple concepts:

1. All APIs are made of **resources**, and **collections of resources**.
2. The **media-type** of a resource dictates the relationships between that resource and the other resources, and what it is possible to do with them.

Therefore, Restfully can work with any reasonably RESTful API provided that:

* The API returns semantically correct HTTP status codes;
* The API make use of `GET`, `POST`, `PUT`, `DELETE` HTTP methods;
* The API returns a valid `Content-Type` header in all responses;
* The API returns a `Location` HTTP header on 201 and 202 responses;
* The API returns links to other resources in all responses (the so-called HATEOAS constraint of REST).

If one of the API `Content-Type` is not already supported by one of the `Restfully::MediaType` objects (see `lib/restfully/media_type`), then you just have to build it and register it with Restfully.

Documentation can be found at <http://rubydoc.info/gems/restfully>.


## Installation

    $ gem install restfully

If you require media-types that need an XML parser, you must also install the `libxml-ruby` library:

    $ gem install libxml-ruby


## Usage

### Command line

    $ export RUBYOPT="-rubygems"
    $ restfully --uri URI [-u username] [-p password]
  
e.g., for the [Grid'5000 API](https://www.grid5000.fr/mediawiki/index.php/API):

    $ restfully --uri https://api.grid5000.fr/sid/grid5000 -u username -p password

If the connection was successful, you should get a prompt. You may enter:

    ruby-1.8.7-p249 > pp root
    #<Resource:0x8108399c uri=https://api.grid5000.fr/sid/grid5000
      RELATIONSHIPS
        environments, self, sites, version, versions
      PROPERTIES
        "uid"=>"grid5000"
        "type"=>"grid"
        "version"=>"da6abdd13e2e626f64502a648b784372eac790b1">
     => nil

And then follow the links advertised under the `RELATIONSHIPS` header to discover the other API resources. For instance, the `sites` resource can be accessed as follows:

    ruby-1.8.7-p249 > pp root.sites
    #<Collection:0x8106d55c uri=https://api.grid5000.fr/sid/grid5000/sites
      RELATIONSHIPS
        self, version, versions
      ITEMS (0..9)/9
        #<Resource:0x81055a9c uri=https://api.grid5000.fr/sid/grid5000/sites/bordeaux>
        #<Resource:0x81040d54 uri=https://api.grid5000.fr/sid/grid5000/sites/grenoble>
        #<Resource:0x8102c070 uri=https://api.grid5000.fr/sid/grid5000/sites/lille>
        #<Resource:0x8101738c uri=https://api.grid5000.fr/sid/grid5000/sites/lyon>
        #<Resource:0x81002658 uri=https://api.grid5000.fr/sid/grid5000/sites/nancy>
        #<Resource:0x80fed924 uri=https://api.grid5000.fr/sid/grid5000/sites/orsay>
        #<Resource:0x80fd8bb4 uri=https://api.grid5000.fr/sid/grid5000/sites/rennes>
        #<Resource:0x80fc3dcc uri=https://api.grid5000.fr/sid/grid5000/sites/sophia>
        #<Resource:0x80faf070 uri=https://api.grid5000.fr/sid/grid5000/sites/toulouse>>
     => nil

Note that we're using `pp` to pretty-print the output, but it's not required.

A Collection is a specific kind of Resource, and it has access to all the methods provided by the Ruby [Enumerable](http://www.rubydoc.info/stdlib/core/1.9.2/Enumerable) module.

For ease of use and better security, you may prefer to use a configuration file to avoid re-entering the Restfully options every time:

    $ echo '
    uri: https://api.grid5000.fr/sid/grid5000
    username: MYLOGIN
    password: MYPASSWORD
    ' > ~/.restfully/api.grid5000.fr.yml && chmod 600 ~/.restfully/api.grid5000.fr.yml

And then:

    $ restfully -c ~/.restfully/api.grid5000.fr.yml

If you want to record the commands you enter in your interactive session, just
add the `--record` flag, and at the end of your session the commands you
entered will have been written into `SESSION_FILE` (by default:
`restfully-tape`).

Note: depending on your Readline installation, you might see the following
message: "Bond has detected EditLine and may not work with it. See the
README's Limitations section". You can safely ignore it.

### Replay

Restfully can replay a sequence of ruby expressions. Just pass the FILE (local
or HTTP URI) as argument to the `restfully` tool:

    $ restfully -c ~/.restfully/my-config.yml path/to/file.rb
    $ restfully -c ~/.restfully/my-config.yml http://server.ltd/script.rb

Or via STDIN:

    $ echo "pp root" | restfully -c ~/.restfully/config.yml

Don't hesitate to play with the `--replay` option, which outputs the content of the FILE line by line, and the result of each expression.

By default, the program exits when the content of the FILE has been executed.
Pass the `--shell` flag to keep a shell open in the same Restfully session
after FILE has been executed. This is useful if you want to manipulate the
variables defined by the FILE.

Also, note that any `Restfully::Session.new(...)` declaration in the code you
execute will have its configuration overridden with anything given on the
command line (either in a configuration file or as arguments). Therefore you
can easily execute scripts written by others, in your own context.

### As a library
See the `examples` directory for examples.


## Development

### Testing 

* `rake spec`; or
* run `autotest` in the project directory.
* we use [Travis CI](http://travis-ci.org/) on every push: [![Build Status](https://secure.travis-ci.org/crohr/restfully.png)](http://travis-ci.org/crohr/restfully)

### Note on Patches/Pull Requests
 
* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a future version unintentionally.
* Commit, do not mess with Rakefile, version, or history (if you want to have your own version, that is fine but bump version in a commit by itself I can ignore when I pull).
* Send me a pull request. 


## Copyright

Copyright (c) 2009-2012 [Cyril Rohr](http://crohr.me), INRIA Rennes - Bretagne Atlantique. 

See LICENSE for details ([CeCILL-B] [cecillb], a BSD style license).

[cecillb]: http://www.cecill.info/licences/Licence_CeCILL-B_V1-en.html