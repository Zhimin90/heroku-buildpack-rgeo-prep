# heroku-buildpack-rgeo-prep

This buildpack overwrites Heroku's default .bundle/config to set BUNDLE_BUILD__RGEO to Heroku's build directory.

This is needed because projects are actually built somewhere like `/tmp/build_1890cktlpat5d`.

## Setup

Create this `.vendor_urls` file in the root of your project:

    https://s3.amazonaws.com/diowa-buildpacks/geos-3.6.1-heroku.tar.gz
    https://s3.amazonaws.com/diowa-buildpacks/proj-4.9.3-heroku.tar.gz


Add this file to git and make sure it ends with a newline.

Now, set up your Heroku configuration:

    heroku buildpacks:set https://github.com/diowa/heroku-buildpack-rgeo-prep.git
    heroku buildpacks:add https://github.com/peterkeen/heroku-buildpack-vendorbinaries.git
    heroku buildpacks:add heroku/ruby
    heroku config:set LD_LIBRARY_PATH=/app/lib

If you haven't already set up your Heroku database for postgis, you need to run the following steps. You currently must have a production level database to enable postgis.

Since postgis uses different settings in the database.yml, you need to modify the `DATABASE_URL` variable. Run the following command and extract the necessary components out of it:

    $ heroku config:get DATABASE_URL
    postgres://<username>:<password>@<host>:<port>/<database>

With those variables, run the following command

    $ heroku config:set DATABASE_URL="postgis://<username>:<password>@<host>:<port>/<database>?postgis_extension=true&search_schema_path=public,postgis"

Enable PostGIS

    $ heroku pg:psql
    => CREATE EXTENSION postgis;

Deploy

    git push heroku master

Verify it worked

    $ heroku run console

    >> RGeo::Geos.supported?
    => true
    >> RGeo::CoordSys::Proj4.supported?
    => true

If both of these are true, you should be ready to go.

## References

* [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
* [Using Multiple Buildpacks for an App](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app)
* [app.json Schema](https://devcenter.heroku.com/articles/app-json-schema#buildpacks)

## Credits

This solution draws from many people's research including

* https://github.com/jcamenisch/heroku-buildpack-rgeo
* https://github.com/davekapp for help with troubleshooting the extconf.rb build process
* https://gist.github.com/perplexes/5357663
* https://github.com/woahdae for updating BUNDLE_BUILD__RGEO
