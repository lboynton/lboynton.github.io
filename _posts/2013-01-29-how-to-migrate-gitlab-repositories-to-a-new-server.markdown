---
layout: post
title: How to Migrate GitLab Repositories to a New Server
date: 2013-01-29 21:52:22.000000000 +00:00
categories: sys-admin
tags:
- gitlab
permalink: /2013/01/29/how-to-migrate-gitlab-repositories-to-a-new-server/
---

Just a quick post for me to refer back to in the future. I used the steps below to migrate all GitLab repositories and database from version 3.1 to 4.1.

# Migrate Database

In this case, I'm using MySQL.

1.  Use mysqldump to create dump of old database, then create a new database on the new server and import this.

    1.  On old: mysqldump gitlab | gzip &gt; gitlab.sql.gz
    2.  On new: gunzip &lt; gitlab.sql.gz | mysql gitlab

2.  Run the db migrate command to make sure the schema is updated to the latest version.

    1.  sudo -u gitlab -H bundle exec rake db:migrate RAILS_ENV=production

# Migrate Repositories

1.  Rather than creating a new SSH key on the new server, copy over the old SSH key pair to the new server. Also copy of the authorized_keys file.
2.  Set up gitolite on the new server using the old SSH key
3.  Copy contents of /home/git/.gitolite/keydir/ from old to new server so that users don’t have to add their SSH keys again and change ownership to git:git.
4.  Copy repositories from old to new
5.  sudo chmod -R ug+rwXs,o-rwx /home/git/repositories/
6.  sudo chown -R git:git /home/git/repositories/
7.  sudo -u gitlab -H bundle exec rake gitlab:gitolite:update_keys RAILS_ENV=production
8.  sudo -u gitlab -H bundle exec rake gitlab:gitolite:update_repos RAILS_ENV=production
9.  Update projects to use namespaces. This will move any projects that are in groups to a namespace. bundle exec rake gitlab:enable_namespaces RAILS_ENV=production

    I could not get this working, easier to remove all projects from groups on the old install before migrating.
10.  Run the rake:check command and see if it complains about anything. You may need to run the create satellites command or the run the script to update the hooks.

    bundle exec rake gitlab:check RAILS_ENV=production
    sudo -u git -H /home/gitlab/lib/support/rewrite-hooks.sh

## Troubleshooting

1.  If you get:

    FATAL: split conf set, gl-conf not present for 'gitolite-admin'

    fatal: The remote end hung up unexpectedly

    Then you need to re-run the gitolite setup. This I think is because gitolite-admin is still in version 2 format.
2.  If you get an error “cannot load such file -- rb-inotify”, you need to add gem “ib-notify” to the bottom of the gemfile and run bundle install again.
3.  If you get an error about invalid SSH keys, keep deleting the offending keys until the errors go away.