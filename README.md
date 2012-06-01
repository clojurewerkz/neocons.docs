# Neocons documentation

This is a documentation site for [Neocons](http://github.com/michaelklishin/neocons), similar to [clojureriak.info](http://clojureriak.info), [clojuremongodb.info](http://clojuremongodb.info) and so on.


## Install Dependencies

With Bundler:

    bundle install --binstubs


## How to run a development server

    ./bin/jekyll --server


## How to regenerate the site

In order to modify contents and launch dev environment, run:

      ./bin/jekyll

In order to recompile haml and sass files for publishing, run

      ./recompile_haml.sh

## License & Copyright

Copyright (C) 2011-2012 Alexander Petrov, Michael S. Klishin.

Distributed under the Eclipse Public License, the same as Clojure.
