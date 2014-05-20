##Foreign key constraint
	create table foods(
  		id integer primary key,
  		type_id integer references food_types(id)
		on delete restrict
  		deferrable initially deferred,
		name text );

The differences are shown in bold and are easy to understand if taken one at a time. The first part of the foreign key instructs SQLite that the type_id column references the id column of the food_types table. From there, you move on to the integrity action clause, which does most of the hard work. In this example, you've used the option on delete restrict. This instructs SQLite to prevent any deletion from the food_types table that would leave a food in the foods table without a parent food id. restrict is one of five possible actions you can define. The full set is as follows:

<b>set null:</b> Change any remaining child value to NULL if the parent value is removed or changed to no longer exist.

<b>set default:</b> Change any remaining child value to the column default if the parent value is removed or changed to no longer exist.

<b>cascade:</b> Where the parent key is updated, update all child keys to match. Where it is deleted, delete all child rows with the key. Pay particular attention to this option, because cascading deletes can surprise you when you least expect them.

<b>restrict:</b> Where the update or delete of the parent key would result in orphaned child keys, prevent (abort) the transaction.

<b>no action:</b> Take a laid-back approach, and watch changes fly by without intervening. Only at the end of the entire statement (or transaction if the constraint is deferred) is an error raised.

Lastly, SQLite supports the deferrable clause, which controls whether the constraint as defined will be enforced immediately or deferred until the end of the transaction.


##Copying a table to another table command
	create table <new_table> as select <C1,C2> from <old_table>;