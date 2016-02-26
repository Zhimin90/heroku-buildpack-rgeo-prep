# heroku-buildpack-rgeo-prep

This buildpack overwrites Heroku's default .bundle/config to set BUNDLE_BUILD__RGEO to Heroku's build directory.

This is needed beccause projects are actually built somewhere like `/tmp/build_1890cktlpat5d`.

## Setup

This solution closely follows this [blog article](https://web.archive.org/web/20120417213149/http://devcenter.spacialdb.com/Heroku.html) by SpacialDB.

Create this `.buildpacks` file in the root of your project:

    https://github.com/peterkeen/heroku-buildpack-vendorbinaries.git
    https://github.com/diowa/heroku-buildpack-rgeo-prep.git
    https://github.com/heroku/heroku-buildpack-ruby.git

Create this `.vendor_urls` file in the root of your project:

    https://s3.amazonaws.com/diowa-buildpacks/geos-3.5.0-heroku.tar.gz
    https://s3.amazonaws.com/diowa-buildpacks/proj-4.9.1-heroku.tar.gz

Add these files to git.

Now, set up your heroku configuration:

    heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git LD_LIBRARY_PATH=/app/lib

If you haven't already set up your heroku database for postgis, you need to run the following steps. You currently must have a production level database to enable postgis.

Since postgis uses different settings in the database.yml, you need to modify the `DATABASE_URL` variable. Run the following command and extract the nessecary components out of it:

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

## Credits

This solution draws from many people's research including

* https://github.com/jcamenisch/heroku-buildpack-rgeo
* https://github.com/davekapp for help with troubleshooting the extconf.rb build process
* https://gist.github.com/perplexes/5357663
* https://github.com/woahdae for updating BUNDLE_BUILD__RGEO
