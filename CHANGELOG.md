# Changelog

## v2.1

* refactoring to have more control over folders when deploying
* more tests for current symfony versions
* compatibility with new symfony folder and command structure

## v2.0

* bugfixes for folders
* stability release

## v2.0.1-alpha

* bugfixes for parameters file
* bugfix for migrations (@vviippola)
* renamed includes to mirror their order

## v2.0.0-alpha
reworked and more flexible configuration

* creates release version in a file per release
* more control over shared folders with additional config options
* possibilities for shallow git copy
* optional cache warmup for sf2
* added tags for specific tasks to allow e.g. cache flush only in later versions
* minified composer.json read overhead
* allow for role hooks

## v1.0

* created stable release for later refactoring

## v0.3

* working travis-ci tests
* datetime release dirs by default
* configurable migrations and schema update
* README update

## v0.2

* fixed merge errors
* fixed deployment issues with config if already exists in project
* fixed issue with missing SYMFONY_ENV while composer install
* fixed composer path variable missing
* added tests directory and travis.yml for testing

## v0.1

* structural refactoring (creating files, directories, ...)
* added cleanup-script to remove old releases
