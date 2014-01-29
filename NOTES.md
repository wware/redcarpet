Getting this working on Ubuntu 12.04
===

There are big problems with the `apt-get` installation of ruby, rubygems, etc. Follow
these instructions to do a correct current installation of Ruby stuff:

* http://stackoverflow.com/questions/9056008
* http://ryanbigg.com/2010/12/ubuntu-ruby-rvm-rails-and-you/

and you still need `gem install nokogiri; gem install rake-compiler`, which probably
needs to be done as root. Then you can `rake test`.
