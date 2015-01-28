Forkbak
=======

Creates backups of a recent fork of your primary Heroku Postgres database.

WARNING
=======

1. This script creates forks of your database to take a backup and then
   destroys it. This is a regular fork, and as such it will incur costs on your
   heroku account.
2. Your Heroku Postgres database is protected by [Continuous
   Protection](https://devcenter.heroku.com/articles/heroku-postgres-data-safety-and-continuous-protection)
   which allow you to use
   [Rollback](https://devcenter.heroku.com/articles/heroku-postgres-rollback) to
   go to a prior database state, protecting against accidental data loss.
   Therefore this approach is not strictly necessary.

Usage
-----

Assuming the app you want to take backups from is called `myapp`, this will
take backups from a fork of `DATABASE_URL` on `myapp`

```bash
export APP=myapp
git clone git@github.com:hgmnz/forkbak.git
cd forkbak
heroku create $APP-backups
heroku addons:add pgbackups --app $APP-backups
heroku addons:add scheduler --app $APP-backups
heroku config:set APP=$APP-backups \
  FORK_FROM_APP=$APP \
  HEROKU_API_KEY=$(heroku auth:token) \
  --app $APP-backups
git push heroku master
```

Finally, set up a scheduled job to kick off the process every night:

```
heroku addons:open scheduler --app $APP-backups
```

Use the following values:

Settings  | Value
--------- | ----------------------
Task      | `bundle exec bin/run`
Dyno size | 1X
Frequency | Daily


*Note*: It is best to create a special user account for this process, and use
it's API key in the `HEROKU_API_KEY` config var.

*Recommended*: Add a logging addon to $APP.

Running manually
----------------

```
heroku run bundle exec bin/run --app $APP-backups
```

## License

forkbak is copyright (c) Harold Giménez and is released under the terms of the
MIT License found in the LICENSE file.
