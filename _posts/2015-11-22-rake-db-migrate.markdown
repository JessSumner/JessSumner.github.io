---
layout: post
title:  "rake db:migrate?"
date:   2015-11-22 14:04:00
---

When I first started learning Ruby on Rails it took me a little while to wrap my head around migrations. Once I figured it out I thought it would be a great first blog post. Six months later, lets give it a go.

There is a huge amount of magic involved with Rails, especially to a newbie with no computer science background. I began by following instructions and trying not to get too bogged down with every little detail. However one line of code kept popping up <code>$rake db:migrate</code> and this bugged me.

A new rails application has an empty database. Generating a Model in an application creates a migration. A migration includes instructions to alter the database for your application.

![Generating model screenshot](/assets/20151122_migrations/generate_model.png)
<h3>Caption: Generating a model from the command line.</h3>

Once a Model has been generated you can view the created migration in your text editor via app/db/migrate/{your_migration}. 

![Generating model screenshot](/assets/20151122_migrations/created_migration.png)
<h3>Caption: How the migration appears in the text editor.</h3>

This migration is very simple. It is creating a table called examples. In this example no data types or names were specified. The table only has two columns, <code>created_at</code> and <code>updated_at</code> both of which are datetimes. This is generated from the line <code>t.timestamps null: false</code> which is automatically added by Rails and in most cases is desired.

Now is the time to run <code>rake db:migrate</code> so let's do it!

![Generating model screenshot](/assets/20151122_migrations/running_rake.png)

This line of code makes the change in the database. Even though the migration has been created it has not changed the database. In this example the database would still be blank until <code>rake db:migrate</code> is run.

Piece of cake!

So a table with a column for timestamps isn't very useful. Let's make it more useful. Generating a migration from the command line can make changes to an existing Model.

![Generating model screenshot](/assets/20151122_migrations/generate_migration.png)

In this example several changes are being made. It adds a column for <code>:name</code> which will store strings. It also adds an index to this new column. This will allow the database to be searched using the <code>:name</code> which could be useful for some applications. Lastly it adds a column for <code>:age</code> which will take integers. 

Running <code>rake db:migrate</code> saves these new columns and our database table. A complete data set in this database would now have a <code>:name</code>, an <code>:age</code> and a timestamp for when the data was added to the database.

Migrations can be used to make many more changes to the database than just adding columns. For example, they can remove columns, change names of columns, change the name of the table and even join tables. But simply they provide instructions for changes to an application's database.
