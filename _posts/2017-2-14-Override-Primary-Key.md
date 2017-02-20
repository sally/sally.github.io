---
layout: post
title: Timezone Troubles
date: 2017-02-14
---

Adapting a social-media approach to Publications made by administrators meant that we should include an option to attach images, videos, and other actors to publications.

We decided to create a Rails/Postgres microservice, an 'actor database,' that would handle the uploads and store the actors properly.

In our Postgres database, we needed to create an Employer model to reflect that certain media belonged to certain employers, and these Employer objects needed to be tied back to objects of the same name in some way.

We decided to use an `external_uuid` (e.g. "30b59371-b7d5-4af6-912c-4e4eaac66a2a") variable that the two Employer models in the two different databases would share to connect the two.

In the actor database, we wanted to eliminate the default primary key of just 'ID' that Postgres provides, and instead use the aforementioned `external_uuid` as the primary key to make things easier for us.

Step 1 was to modify the migration. We would have to turn the primary key off manually:

```ruby
# in the employers table creation migration
...
create_table :employers, id: false do |t|
   ...
end
...
```

Unfortunately, it's slightly hard to set something that isn't an integer to a primary key without being a bit hacky. You could regularly do a `t.primary_key :field_name` inside of the `create_table` block which would indeed set your `field_name` to the primary key, but Postgres would automatically set the datatype as `integer`.

One work-around is this:

```ruby
# in the employers table creation migration
def change
  create_table :employers, id: false do |t|
    t.string :external_uuid, null: false
    ...
  end

  add_index :employers, :external_uuid, unique: true
end
```

Checking our schema, we see that Postgres has detected the `external_uuid` field to be `not null` and also `UNIQUE` indices. As far as Postgres is concerned:

  a primary key constraint is simply a combination of a unique constraint and a not-null constraint

Our `external_uuid` is now the primary key of our `employers` table.

(To be continued with how this translates into modifications for the Model, Routes, URI, etc.)
