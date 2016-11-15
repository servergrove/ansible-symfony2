# Symfony2
[![Build Status](https://travis-ci.org/servergrove/ansible-symfony2.svg?branch=master)](https://travis-ci.org/servergrove/ansible-symfony2)

If you use ansible <2.0, please do a ```ansible-galaxy install servergrove.symfony2,v1.9.0``` to get the last backwards-compatible version.
Otherwise you can stick with stable "master" :-)

Ansible role to easily deploy Symfony2 applications.
It will clone a git repository, a specific branch or a tag, download and run composer install, and run assetic:dump when finished.
The resulting directory structure is similar to what capifony/capistrano creates:

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
    - servergrove.symfony2

  vars:
    symfony_project_root: /tmp/test_app
    symfony_project_name: travis-test
    symfony_project_composer_path: /tmp/test_app/shared/composer.phar
    symfony_project_repo: https://github.com/symfony/symfony-standard.git
    symfony_project_env: prod

    symfony_project_console_opts: '--no-debug'
    symfony_project_keep_releases: 5

    symfony_project_branch: "2.6"
    symfony_project_php_path: php
    symfony_project_keep_releases: 5

    symfony_project_manage_composer: True
    symfony_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
```

Commandline: ```~$ ansible-playbook -i inventory --extra-vars "symfony_project_release=20150417142505,symfony_project_branch=master" test.yml```

## Role Variables

### Possible variables

These are the possible role variables - you only need to have a small set defined, there are defaults.

```yaml
---
- vars:
    # necessary project vars
    symfony_project_root: Path where application will be deployed on server.
    symfony_project_composer_path: path where composer.phar will be stored (e.g. project_root/shared)
    symfony_project_repo: URL of git repository.
    symfony_project_release: Release number, can be numeric, we recommend to set it to release date/time, 20140327100911
    symfony_project_env: prod

    # optional parameters, covered by defaults
    symfony_project_post_folder_creation_tasks: task hook after folder creation
    symfony_project_pre_cache_warmup_tasks: after cache warmup
    symfony_project_pre_live_switch_tasks: before live symlink is switched
    symfony_project_post_live_switch_tasks: after live symlink is switched

    symfony_project_branch: git branch, commit hash or version tag to deploy - defaults to master
    symfony_project_php_path: php
    symfony_project_php_options: ""
    symfony_project_keep_releases: 5
    symfony_project_git_clone_depth: 1 # uses git shallow copy
    symfony_project_github_token: Auth token for github rate limits
    symfony_project_console_opts: ''
    symfony_project_console_command: 'app/console' # sf >= 3.0 bin/console
    symfony_project_config_dir: 'app/config' # symfony configuration dir
    symfony_project_parameters_file: parameters.yml # optional fixed parameters file in shared
    symfony_project_cache_command: cache:warmup

    symfony_project_manage_composer: True
    symfony_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
    symfony_project_composer_run_install: True

    symfony_project_enable_cache_warmup: True warmup symfony cache, check out cache command!
    symfony_project_fire_schema_update: False # rund mongodb schema update if installed
    symfony_project_fire_migrations: run doctrine migrations, if installed
    symfony_project_symlink_assets: run assets:create with symlink options

    symfony_project_shared_folders: # folders to be linked from shared directory to release dir
      - {name: logs, src: app/logs, path: app/logs}

    symfony_project_managed_folders: # folderst to be created/checked in release dir
      - {name: cache, path: app/cache}
```

### Role variable defaults

As you can see, the release number default is the current date/time with seconds to allow for easy multiple releases per day. But you can always overwrite with ```--extra-vars=""``` option.

```yaml
---
- vars
    symfony_project_release: <datetime> # internally replaced with YmdHis
    symfony_project_branch: master
    symfony_project_php_path: /usr/bin/php
    symfony_project_keep_releases: 5
    symfony_project_git_clone_depth: 1
    symfony_project_console_opts: ''
    symfony_project_composer_opts: '--no-dev --optimize-autoloader --no-interaction'
    symfony_project_fire_migrations: False
    symfony_project_symlink_assets: True
```

## hooks
If you need any more tasks and stuff in your deployment, you now have the option to include hook scripts.
In my projects there's often e.g. a gulp task that has to be started before finishing the release. You're also free to create more folders yourself or do whatever you need.
As an additional goodie, you can use the internal dynamically created facts from main role:

```yaml
---
  symfony_project_release # release timestamp
  symfony_current_release # release name
  symfony_current_release_dir # fully qualified path to release
  symfony_shared_dir # shared folder base path
  symfony_console # fully qualified console command path
```
possible hooks:

```yaml
---
  symfony_project_post_folder_creation_tasks: task hook after folder creation
  symfony_project_pre_cache_warmup_tasks: after cache warmup
  symfony_project_pre_live_switch_tasks: before live symlink is switched
  symfony_project_post_live_switch_tasks: after live symlink is switched
```

These hooks trigger an include when defined.
Define hooks:
```yaml
  symfony_project_post_folder_creation_tasks: "{{playbook_dir}}/hooks/post_folder_creation.yml"
```
The "hooks" dir should be in your deployment project as a subfolder. I'd recommend to use this name as a convention. Also it's convinient to use the name of the hook task as a yml name.

## hook examples
These examples can be found in the package's hooks dir.

### php-fpm process restart
E.g. restart php-fpm after successfully finishing deployment.
This is much easier to maintain for different environments.

Create ```<your deployment>/hooks/post_live_switch.yml```:

```yaml
---
- name: hook | Restart php-fpm
  service: name=php5-fpm state=restarted
  when: symfony_project_env == "prod"
```

### create and maintain web/downloads folder
Example for managing additional directories

Create ```<your deployment>/hooks/post_folder_creation.yml```:

```yaml
---
- name: hook | Create web/uploads folder.
  file: state=directory path={{symfony_shared_dir}}/web/uploads

- name: hook | Symlink to release.
  file: state=link src="{{symfony_shared_dir}}/web/uploads" path="{{symfony_current_release_dir}}/web/uploads"
```
As an alternative to managing folders via hooks, you can also configure either the shared folders or the creation of folders in your release directory in your confguration:

```yaml
---
symfony_project_shared_folders: # folders to be linked from shared directory to release dir
  - {name: logs, src: app/logs, path: app/logs}
  - {name: uploads, src: web/uploads, path: web/uploads}
```
## Passing PHP options

Suppose you need to overide some of php's options on the command line. Simply set the symfony_project_php_options. For example
```yaml
---
- hosts: servers
  roles:
    - servergrove.symfony2

  vars:
    symfony_project_root: /tmp/test_app
    symfony_project_name: travis-test
    symfony_project_composer_path: /tmp/test_app/shared/composer.phar
    symfony_project_repo: https://github.com/symfony/symfony-standard.git
    symfony_project_env: prod

    symfony_project_console_opts: '--no-debug'
    symfony_project_keep_releases: 5

    symfony_project_php_path: php
    symfony_project_php_options: -dmemory_limit=512M -dzend.enable_gc=0
```
This will set the php variables memory_limit to 512M and zend.enable_gc to 0 when any php command is run, such as composer install or cache:warmup.

## Dependencies

None

## Testing

The deployment contains a basic test, executed by travis. If you want to locally test the role, have a look into ```.travis.yml``` for the exceution statements. Make sure you have a local php executable (needed for composer install and symfony console scripts).
Add a file ```ansible.cfg```to your deployment project folder with contents:

```
[defaults]
roles_path = ../
```

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
