---
layout: post
title: "migrating-data-separate-from-the-schema"
date: "Thu Dec 11 18:42:07 -0500 2014"
---


References:

https://github.com/Casecommons/datafix
http://railsguides.net/change-data-in-migrations-like-a-boss
https://github.com/ka8725/migration_data
https://github.com/amorphid/will_it_migrate
https://www.honeybadger.io/blog/2013/08/06/zero-downtime-migrations-of-large-databases-using-rails-postgres-and-redis

Notes:

Validations change over time
Tables and classes change over time
The migration does not change over time. It's frozen (until you nuke the migrations and only use the db:schema:load)
Datafixes also won't change over time

Uses:

Denormalize values into another table
Long running migrations that require API calls (honeybadger example)
Fix existing data to comply with new validations
