# Symfony2

Ansible role to easily deploy Symfony2 applications. It will clone a git repository, download and run composer install, and run assetic:dump when finished. The resulting directory structure is similar to what capifony creates:

```
project
    composer.phar
    releases
        release
    shared
        web/uploads
        app
            config
            logs
    current -> symlink to latest deployed release
```

## Requirements

None

## Installation

```
    $ ansible-galaxy install servergrove.symfony2
```


## Role Variables

### Possible variables

These are the possible role variables - you only need to have a small set defined, there are defaults.

```yaml
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
    symfony2_project_composer_opts: '--no-dev --optimize-autoloader'
    symfony2_project_keep_releases: 5
    symfony2_project_clean_versioning: true
```

### Role variable defaults

As you can see, the release number default is the current date/time with seconds to allow for easy multiple releases per day. But you can always overwrite with ```--extra-vars=""``` option.

```yaml
- vars
    symfony2_project_release: "{{ lookup('pipe', 'date +%Y%m%d%H%M%S') }}"
    symfony2_project_branch: master
    symfony2_project_php_path: php
    symfony2_project_keep_releases: 5
    symfony2_project_clean_versioning: true
    symfony2_project_console_opts: ''
    symfony2_project_composer_opts: '--no-dev --optimize-autoloader'
```

Dependencies
------------

None

Example Playbook
-------------------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
    roles:
        - { role: servergrove.symfony2, symfony2_project_root: /var/www/vhosts/example.com/, symfony2_project_name: demo, symfony2_project_branch: master, symfony2_project_release: 1 }
```

License
-------

[MIT](LICENSE) license

Author Information
------------------

Contributions are welcome: https://github.com/servergrove/ansible-symfony2
