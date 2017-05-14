---
layout: post
title: "Rails Migrations at Scale"
date: 2017-05-14
categories: Shopify, Rails, Backend, LHM
---

{% include image.html url="https://camo.githubusercontent.com/3d6afa3066f4746fbde9937d89e9f80f65ac0d75/687474703a2f2f6661726d342e7374617469632e666c69636b722e636f6d2f333039332f323834343937313939335f313766326464663261385f7a2e6a7067" title="Large Hadron Collider" %}

I recently completed an internship at Shopify as a Backend Developer. The most interesting thing I learned from my internship was how easily default technologies, like Ruby on Rails' Active Record migrations, can break at scale. We often take for granted the technology we rely on, until it lets us down. 

Shopify uses MySQL as their underlying database. For many years, Ruby on Rails' Active Record migrations were a sufficient and easy way to modify their databases' structure. It wasn't until their database tables grew into millions of rows that we realized Active Record migrations could no longer handle our scale without causing service disruptions. To stay agile and to continue to build Shopify, modifications to databases' schema are regularly required. Fortunately, Shopify wasn't the first to encounter this problem and I know we won't be the last.

## The Heart of Active Record MySQL migrations, ALTER TABLE

The behaviour of `ALTER TABLE` is the reason why some Active Record migrations are no longer a viable option for schema changes on large tables. The `ALTER TABLE` MySQL command is used by Active Record to change the structure of a database table, such as adding a new column. There are some exceptions to how `ALTER TABLE` will modify the schema of a table, depending on the type of schema change.

Typically, `ALTER TABLE` takes a safe migration approach by first creating a new table with the updated schema from the migration. `ALTER TABLE` waits until all in progress operations modifying the table to be migrated are finished then creates a table-wide write lock. With this write lock in place, all rows from the original table will be copied over to the new table. During this copy, reads to the original table can still occur.

{% include image.html img="AR-InProgress.png" title="Active Record migration in progress" caption="Active Record migration in progress." %}

Once the copy is complete, a read lock is also placed on the table. With no operations running against the original table, `ALTER TABLE` quickly renames the new table to the original table name and drops the original table. The last step to the migration is to direct the operations stalled by the read and write locks during the migration to the new table without any failures.  

{% include image.html img="AR-Finished.png" title="Finished Active Record migration" caption="Finished Active Record migration." %}

The approach `ALTER TABLE` takes is simple, and seemingly efficient. Problems can occur with active applications with sizable tables, as is especially the case with a write-heavy application like Shopify. The issue stems from the table-wide write lock in place during most of the migration. Until the migration is finished, an application's write operation against the table is stalled. With reasonable tables, the migration copies over rows from the original table to the new table in under a few seconds. Large database tables, like ones at Shopify, have millions of rows will numerous columns holding a variety of data. Migrations on these tables can take anywhere from minutes to even hours! Application initiated operations, such as those spawned from web requests, will timeout before the migration finishes, leading to a service disruption. Clearly a migration that can scale must not have a table-wide write lock dominate the duration of the migration. 

## Large Hadron Migrator to the Rescue

Large Hadron Migrator (LHM), named after the Large Hadron Collider, is a "gem for online Active Record migrations" initially developed by Soundcloud. LHM integrates cleanly with Active Record migrations without the scaling issues of typical migrations. LHM is open source on [GitHub](https://github.com/soundcloud/lhm) and Shopify maintains a public [fork](https://github.com/Shopify/lhm). 

Large Hadron Migrator migrations begin the same way as Active Record migrations, with a new table with the updated schema. LHM begins to copy over collections of rows from the original table to the new table. This eliminates the need for the table-wide write lock and LHM only locks the rows it is selectively copying. LHM exposes options for determining how many rows should be copied at one time and the delay between successive copies of row chunks so developers can find suitable choices for their particular tables. With chunks of rows being copied over to the new table, and writes still occurring to the original table, it is possible for data inconsistency to exists between the two tables, say if a copied row is updated on the original table. To resolve this, LHM registers for [MySQL triggers](https://dev.mysql.com/doc/refman/5.7/en/triggers.html). A trigger causes another database operation to execute on the occurrence of a database event, such as inserting a row in a table. With LHM's triggers, when a row on the original table is modified, created, or deleted, the change is also mirrored on the new table. 

{% include image.html img="LHM-InProgress.png" title="Large Hadron Migrator migration in progress" caption="Large Hadron Migrator migration in progress." %}

Once LHM has finished copying over all the rows to the new table, and any in progress triggers finish, LHM places a read-write lock on the entire table and atomically swaps the names of the two tables. This operation is fast, so the table-wide lock is not harmful to the Rails application. To prevent accidental data loss, LHM leaves the original (now renamed and unused) table in the database. 


{% include image.html img="LHM-Finished.png" title="Finished Large Hadron Migrator migration" caption="Finished Large Hadron Migrator migration." %}

{% highlight Ruby %}
require 'lhm'

class MigrateProducts < ActiveRecord::Migration[5.0]
  def self.up
    Lhm.change_table :products do |m|
      m.add_column :brand, "VARCHAR(255)"
      m.add_index  [:created_at]
    end
  end

  def self.down
    Lhm.change_table :products do |m|
      m.remove_column :brand
      m.remove_index  [:created_at]
    end
  end
end
{% endhighlight %}

A sample LHM migration.

---

Schema migrations are a normal part of a Rails application's development and should not be a daunting task. As an application scales, service disruptions and downtime for a minor schema change shouldn't be tolerated. With Large Hadron Migrator, Rails migrations at scale become easy again.


