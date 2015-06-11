# Symfony2
[![Build Status](https://travis-ci.org/servergrove/ansible-symfony2.svg?branch=master)](https://travis-ci.org/servergrove/ansible-symfony2)

Ansible role to easily deploy Symfony2 applications.
It will clone a git repository, a specific branch or a tag, download and run composer install, and run assetic:dump when finished.
The resulting directory structure is similar to what capifony creates:

```
project
    composer.phar
    releases
        release
    shared
        web
            uploads
        app
            config
            logs
    current -> symlink to latest deployed release
```

It will also keep the ```releases``` directory clean and deletes all but your configured amount of releases.
You're also able to switch on/off whether ansible should execute doctrine migrations, or mongodb schema update, etc.

## Requirements

Installed version of Ansible.

## Installation

### Ansible galaxy

```
    $ ansible-galaxy install servergrove.symfony2
```

### Usage

This playbook is taken from the travis testcase. You can always pass these values as commandline parameters.

```yaml
---
- hosts: servers
  roles:
    - ansible-symfony2

  vars:
    symfony2_project_root: /test_app
    symfony2_project_name: travis-test
    symfony2_project_composer_path: /test_app/shared
    symfony2_project_repo: https://github.com/symfony/symfony-standard.git
    symfony2_project_branch: "2.6"
    symfony2_project_php_path: php
    symfony2_project_env: prod
    symfony2_project_console_opts: '--no-debug'
    symfony2_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
    symfony2_project_keep_releases: 5
    symfony2_project_clean_versioning: true
```

Commandline: ```~$ ansible-playbook -i inventory --extra-vars "symfony2_project_release=20150417142505,symfony2_project_branch=master" test.yml```

## Role Variables

### Possible variables

These are the possible role variables - you only need to have a small set defined, there are defaults.

```yaml
---
- vars:
    symfony2_project_root: Path where application will be deployed on server.
    symfony2_project_name: Name of project.
    symfony2_project_composer_path: path where composer.phar will be stored (e.g. project_root/shared)
    symfony2_project_repo: URL of git repository.
    symfony2_project_release: Release number, can be numeric, we recommend to set it to release date/time, 20140327100911
    symfony2_project_branch: git branch to deploy.
    symfony2_project_php_path: /usr/local/php54/bin/php
    symfony2_project_env: prod
    symfony2_project_console_opts: ''
    symfony2_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
    symfony2_project_keep_releases: 5
    symfony2_project_clean_versioning: true
    symfony2_fire_schema_update: false # Runs doctrine:mongodb:schema:update
    symfony2_fire_migrations: false # Runs doctrine migrations script 
```

### Role variable defaults

As you can see, the release number default is the current date/time with seconds to allow for easy multiple releases per day. But you can always overwrite with ```--extra-vars=""``` option.

```yaml
---
- vars
    symfony2_project_release: <datetime> # internally replaced with YmdHis
    symfony2_project_branch: master
    symfony2_project_php_path: /usr/bin/php
    symfony2_project_keep_releases: 5
    symfony2_project_clean_versioning: true
    symfony2_project_console_opts: ''
    symfony2_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
    symfony2_fire_schema_update: false
    symfony2_fire_migrations: false
```

## Dependencies

None

## Testing

The deployment contains a basic test, executed by travis. If you want to locally test the role, have a look into ```.travis.yml``` for the exceution statements and (maybe) remove the ```geerlingguy.php ``` from ```tests/test.yml``` if you have a local php executable (needed for composer install and symfony console scripts).

The test setup looks like this:

```
servergrove.symfony2
    tests
        inventory # hosts information
        test.yml # playbook
    .travis.yml # travis config
```

## License

[MIT](LICENSE) license

### Author Information

Contributions are welcome: https://github.com/servergrove/ansible-symfony2
