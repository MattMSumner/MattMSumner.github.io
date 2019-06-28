---
layout: post
title: "Using doobie; Scala with PostgreSQL vs ActiveRecord"
date: 2019-06-06 11:24:13
categories: ruby scala sql
excerpt: I've wrapped up my second Scala client project, and I enjoyed SQL.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/some-thoughts-on-postgres)*

## Some context

I joined thoughtbot as a Rails developer. Ruby on Rails was my first web
framework and my introduction to building web applications. From this, I've been
lucky enough to write applications in a variety of technologies, and each one
shows me features I take for granted in Rails and features I wish I had. I want
to talk a bit about my last Scala client project, and I'd like to talk mostly
about the database.

ActiveRecord is one of my favourite parts of Rails. I learnt the DSL first and
SQL second so I've always been comfortable thinking in terms of ActiveRecord.
Rolling onto a Scala project, I found myself in a situation where ActiveRecord
was not an option and, at first, I was sceptical.

## Migrations

We decided to use a project called [Flyway] for writing migrations. Before
diving into the differences, I'd like to have some comparison between a Rails
migration and a Flyway migration:

[Flyway]: https://davidmweber.github.io/flyway-sbt-docs/

ActiveRecord:

```rb
# db/migrate/20180824180134_create_companies.rb
class CreateCompanies < ActiveRecord::Migration[5.2]
  def change
    create_table :companies do |t|
      t.text :name, null: false
      t.text :url, null: false
    end
  end
end
```

Flyway:

```sql
-- src/main/resources/db/migration/V1__create_companies.sql
CREATE TABLE companies (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  url TEXT NOT NULL
)
```

One of the immediate differences is that Flyway migrations are in SQL, which can
be a little annoying when it comes to the things that Rails will do "for free"
such as primary keys and timestamps. Since I missed having `created_at` and
`updated_at` timestamps, I added them manually:

```rb
# db/migrate/20180824180134_create_companies.rb
class CreateCompanies < ActiveRecord::Migration[5.2]
  def change
    create_table :companies do |t|
      t.text :name, null: false
      t.text :url, null: false

      t.timestamps
    end
  end
end
```

```sql
-- src/main/resources/db/migration/V1__create_updated_at_trigger.sql
CREATE FUNCTION updated_at_column()
  RETURNS TRIGGER AS $$
  BEGIN
    NEW.updated_at = (now() at time zone 'utc');
    RETURN NEW;
  END;
  $$ language 'plpgsql';

-- src/main/resources/db/migration/V2__create_companies.sql
CREATE TABLE companies (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT (now() at time zone 'utc') NOT NULL,
  updated_at TIMESTAMP DEFAULT (now() at time zone 'utc') NOT NULL
)

CREATE TRIGGER update_companies_updated_at
  BEFORE UPDATE ON companies
  FOR EACH ROW EXECUTE PROCEDURE updated_at_column();
```

In our Scala project, we set up a procedure in our first migration, and then our
second migration can set up a trigger on updates to set the `updated_at` column
to the current time. We had to remember to add this trigger to every new table,
but it was nice to have some familiar timestamps for debugging purposes.

## Versions

Let's talk about the versioning in Flyway. Similar to ActiveRecord, there is a
table created to keep track of when migrations have been applied or not. The
table row includes a checksum of the file; if a migration is modified then
an error would get raised. I would appreciate this with ActiveRecord, but since
it's written in Ruby, you can reference objects in your system that might
change. Since Flyway is using SQL, these migrations should never be modified.

## PostgreSQL Enums

Let's now use a native feature of PostgreSQL that ActiveRecord handles
differently.

```rb
# db/migrate/20180824181029_add_reminder_cadence_to_companies.rb
class CreateCompanies < ActiveRecord::Migration[5.2]
  def change
    add_column :companies, :reminder_cadence, :integer, null: false, default: 0
  end
end
```

```sql
-- src/main/resources/db/migration/V3__add_reminder_cadence_to_companies.sql
CREATE TYPE reminder_cadence AS enum(
  'never',
  'week',
  'one_month',
  'three_months',
  'six_months'
);

ALTER TABLE companies
  ADD COLUMN last_reminder_at TIMESTAMP,
  ADD COLUMN reminder_cadence reminder_cadence NOT NULL DEFAULT 'never';
```

Here we're implementing a new attribute/concept that has a defined set of values
mapping to our business logic. In the Rails migration, very little of that gets
represented. We can maybe infer from the column name that this data represents
more than just an integer, but we're missing many of the fail-safes provided by
using a PostgreSQL enum. For example, if we decide to remove one of these
options from our enum in a later migration, the Scala example will fail before
running the migration. In the ActiveRecord case, we would almost certainly only
realise once we start getting errors in our deployed app at runtime.

In the case of the Rails, when exploring the raw data, there's an additional
mapping for this column to make any sense from integer to an enumerated value.
We ended up using the database more to explore our data than I typically would
in a Rails app because I knew that the data would almost always make sense
without having to convert to ActiveRecord objects.

## Types, Tests and doobie

Now that we've gone over migrations I want to start talking about the interface
between Postgres and our Scala code. [doobie] is not an ORM but describes itself
as a principled way to construct programs that use the JDBC. One of the things
that impressed me with the doobie library was how seamless that interface was in
terms of translating our Postgres tables into statically typed classes. However,
there was a fair amount of boilerplate required that ActiveRecord would provide
for us out of the box. Let's take a look at our model in Rails:

[doobie]: https://tpolecat.github.io/doobie/

```rb
class Company < ApplicationRecord
  enum reminder_cadence: [:active, :archived]
end
```

as compared to our Scala class:

```scala
/** src/main/scala/data/Company.scala */
package data

case class CompanyId(value: Int)

trait Company {
  def name: String
  def url: Option[NormalizedUrl]
  def reminderCadence: ReminderCadence
}

object Company {
  case class New(name: String,
                 url: Option[NormalizedUrl],
                 reminderCadence: ReminderCadence)
      extends Company

  case class Saved(id: CompanyId,
                   name: String,
                   url: Option[NormalizedUrl],
                   reminderCadence: ReminderCadence)
      extends Company
}
```

But this also required that we have the `ReminderCadence` defined. There's quite
a lot of code here since this is where we're wiring Postgres enums to our Scala
objects. We have a list of concrete objects that the reminder cadence can be
that all extend the sealed trait `ReminderCadence`. We then define a `fromString`
and an unsafe version to use when reading values from Postgres.

```scala
/** src/main/scala/data/ReminderCandence.scala */
package data

import doobie.Meta
import doobie.postgres.implicits._

sealed trait ReminderCadence {
  val convertToString: String
}

object ReminderCadence {
  case object Never extends ReminderCadence { val convertToString = "never" }
  case object Week extends ReminderCadence { val convertToString = "week" }
  case object OneMonth extends ReminderCadence { val convertToString = "one_month" }
  case object ThreeMonths extends ReminderCadence { val convertToString = "three_months" }
  case object SixMonths extends ReminderCadence { val convertToString = "six_months" }

  def fromString(s: String): Option[ReminderCadence] =
    Option(s) collect {
      case "never"        => Never
      case "week"         => Week
      case "one_month"    => OneMonth
      case "three_months" => ThreeMonths
      case "six_months"   => SixMonths
    }

  def unsafeFromString(s: String) =
    fromString(s).getOrElse(
      throw doobie.util.invariant.InvalidEnum[ReminderCadence](s)
    )

  implicit val ReminderCadenceMeta: Meta[ReminderCadence] =
    pgEnumString(
      "reminder_cadence",
      ReminderCadence.unsafeFromString(_),
      _.convertToString
    )
}
```

There's no denying that we've written a ton more code in the Scala version. But
there is also far less magic. Because Scala is a statically typed language, if I
try to access an attribute on `Company` that doesn't exist, my program won't
compile. Additionally, I know that a company must have a name, but the URL may
not be there. We've written before on [alternatives in Ruby to nil] and also on
the [maybe type in Elm]. `Option` is a similar type in Scala for representing the
idea that we either have a value or we have nothing. The compiler will make sure
we handle both cases, like a good friend.

[alternatives in Ruby to nil]: https://robots.thoughtbot.com/if-you-gaze-into-nil-nil-gazes-also-into-you
[maybe type in Elm]: https://robots.thoughtbot.com/maybe-thinking

Let's take a look at the query interface. ActiveRecord gives us access to all of
the queries I need to write in Scala out of the box. Here's the CompanyQuery
file we had for our Company model:

```scala
/** src/main/scala/data/CompanyQuery.scala */
package data

import doobie.{Fragment, Query0, Update0}
import doobie.implicits._

object CompanyQuery {
  def find(id: CompanyId): Query0[Company.Saved] =
    (selectFragment ++ fr"WHERE id=$id")
      .query[Company.Saved]

  def findByUrl(url: NormalizedUrl): Query0[Company.Saved] =
    (selectFragment ++ fr"WHERE url=$url limit 1")
      .query[Company.Saved]

  private def selectFragment: Fragment =
    fr"SELECT id, name, url, reminder_cadence FROM companies "

  def insert(company: Company.New): Update0 =
    fr"""INSERT INTO companies (name, url, reminder_cadence) VALUES (
      ${company.name},
      ${company.url},
      ${company.reminderCadence}
    )""".update
}
```

Once again, I'm writing SQL to interact with my database. How do we know that
the query I've written will translate into the object I'm constructing? It turns
out that it's difficult to get the compiler to know for certain that the SQL
you've written will match your schema and produce the results you're expecting,
so doobie decided not to bother. Instead, the library ships with a test helper
that's super easy to add to your test suite:

```scala
/** src/test/scala/data/CompanyQuerySpec.scala */
package data

import model.CompanyId
import org.scalatest.FunSuite

class CompanyQueryCheckSpec extends FunSuite {
  test("find") {
    check(CompanyQuery.find(CompanyId(1)))
  }

  test("findByUrl") {
    check(CompanyQuery.findByUrl(NormalizedUrl("https://example.com")))
  }

  test("insert") {
    check(
      CompanyQuery.insert(
        Company.New(
          name = "test",
          url = Some(NormalizedUrl("https://example.com")),
          reminderCadence = ReminderCadence.Week
        )
      )
    )
  }
}
```

These are awesome. The tests make sure that:

- The SQL we're trying to run is considered valid by the database.
- Every Scala type we're trying to map lines up with a correct type for that
  column.
- We don't make any unsafe conversions, like `String` to `Long`.

Here are some examples of the types of errors these tests can expose:

```
✕ SQL Compiles and TypeChecks
  ERROR: column "ids" does not exist
  Hint: Perhaps you meant to reference the column "companies.id".

✕ C04                                 →  Option[BigDecimal]
  Too few columns are selected, which will result in a runtime
  failure. Add a column or remove mapped Option[BigDecimal] from the
  result type.

✕ P01 CompanyId  →  INTEGER (int4)
  CompanyId is not coercible to INTEGER (int4) according to the JDBC
  specification. Fix this by changing the schema type to BIGINT, or the Scala
  type to Int or JdbcType.

✕ C04 revenue NUMERIC (numeric) NULL      →  BigDecimal
  Reading a NULL value into BigDecimal will result in a runtime failure. Fix
  this by making the schema type NOT NULL or by changing the Scala type to
  Option[BigDecimal]
```

## Summary

Overall, I felt a lot more confident on this Scala project in the shape of my
data and that the code I'd written would behave as expected once shipped to
production. On a typical Rails application, I'd gain that confidence through
test-driven development, but I've often found myself triple checking to ensure I
hadn't forgotten any edge cases. With Scala, I discovered that a lot of the
cases I would usually consider towards the end of the process were presented to
me as problems earlier, leading to a more robust system.

Another advantage of interacting with our database via SQL is clarity when
debugging. I enjoyed writing SQL queries because when we had to debug an issue,
I felt more comfortable popping into my SQL console and looking at data.

:+1: would work with again.
