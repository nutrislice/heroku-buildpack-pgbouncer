Heroku buildpack: pgbouncer
=========================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) that
allows one to run pgbouncer and stunnel in a dyno alongside application code.
It is meant to be used in conjunction with other buildpacks as part of a
[multi-buildpack](https://github.com/ddollar/heroku-buildpack-multi).

The primary use of this buildpack is to allow for transaction pooling of
PostgreSQL database connections among multiple workers in a dyno. For example,
10 unicorn workers would be able to share a single database connection, avoiding
connection limits and Out Of Memory errors on the Postgres server.

It uses [stunnel](http://stunnel.org/) and [pgbouncer](http://wiki.postgresql.org/wiki/PgBouncer).


FAQ
----
- Q: Why should I use transaction pooling?
- A: You have many workers per dyno that hold open idle Postgres connections and
and you want to reduce the number of unused connections. [This is a slightly more complete answer from stackoverflow](http://stackoverflow.com/questions/12189162/what-are-advantages-of-using-transaction-pooling-with-pgbouncer)

- Q: Why shouldn't I use transaction pooling?
- A: If you need to use named prepared statements, advisory locks, listen/notify, or other features that operate on a session level.
Please refer to PGBouncer's [feature matrix](http://wiki.postgresql.org/wiki/PgBouncer#Feature_matrix_for_pooling_modes) for all transaction pooling caveats.


Disable Prepared Statements
-----
Some ORMs (like ActiveRecord [3.2.9](https://github.com/rails/rails/pull/5872))
allow prepared statements to be disabled
by appending `?prepared_statements=false` to the database's URI. Set
the `PGBOUNCER_PREPARED_STATEMENTS` config var to `false` for the buildpack
to do that for you.


Usage
-----

Example usage:

    $ ls -a
    .buildpacks  Gemfile  Gemfile.lock  Procfile  config/  config.ru

    $ heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git

    $ cat .buildpacks
    https://github.com/gregburek/heroku-buildpack-pgbouncer.git#v0.2.1
    https://github.com/heroku/heroku-buildpack-ruby.git

    $ cat Procfile
    web:    bin/start-pgbouncer-stunnel bundle exec unicorn -p $PORT -c ./config/unicorn.rb -E $RACK_ENV
    worker: bundle exec rake worker

    $ git push heroku master
    ...
    -----> Fetching custom git buildpack... done
    -----> Multipack app detected
    =====> Downloading Buildpack: https://github.com/gregburek/heroku-buildpack-pgbouncer.git
    =====> Detected Framework: pgbouncer-stunnel
           Using pgbouncer version: 1.5.4
           Using stunnel version: 4.56
    -----> Fetching and vendoring pgbouncer into slug
    -----> Fetching and vendoring stunnel into slug
    -----> Moving the configuration generation script into app/.profile.d
    -----> Moving the start-pgbouncer-stunnel script into app/bin
    -----> pgbouncer/stunnel done
    =====> Downloading Buildpack: https://github.com/heroku/heroku-buildpack-ruby.git
    =====> Detected Framework: Ruby/Rack
    -----> Using Ruby version: ruby-1.9.3
    -----> Installing dependencies using Bundler version 1.3.2
    ...

The buildpack will install and configure pgbouncer and stunnel to connect to
`DATABASE_URL` over a SSL connection. Prepend `bin/start-pgbouncer-stunnel`
to any process in the Procfile to run pgbouncer and stunnel alongside that process. 

The buildpack exports a `PGBOUNCER_URI` environment variable for the pgbouncer pseudo-DB, 
which you can use in place of `DATABASE_URL` for database connections. 

Tweak settings
-----
Some settings are configurable through app config vars at runtime. Refer to the appropriate documentation for
[pgbouncer](http://pgbouncer.projects.pgfoundry.org/doc/config.html#_generic_settings)
and [stunnel](http://linux.die.net/man/8/stunnel) configurations to see what settings are right for you.

- `PGBOUNCER_POOL_MODE` Default is transaction
- `PGBOUNCER_DEFAULT_POOL_SIZE` Default is 1
- `PGBOUNCER_RESERVE_POOL_SIZE` Default is 1
- `PGBOUNCER_RESERVE_POOL_TIMEOUT` Default is 5.0 seconds

For more info, see [CONTRIBUTING.md](CONTRIBUTING.md)
