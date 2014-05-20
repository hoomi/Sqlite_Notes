## Foreign key constraint
</br>

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


## Copying a table to another table command
</br>

	create table <new_table> as select <C1,C2> from <old_table>;
	
## Comparing Value types

1. The NULL storage class has the lowest class value. A value with a NULL storage class is considered less than any other value (including another value with storage class NULL). Within NULL values, there is no specific sort order.

2. integer or real storage classes have higher value than NULLs and share equal class value. integer and real values are compared numerically.

3. The text storage class has higher value than integer or real. A value with an integer or a real storage class will always be less than a value with a text storage class no matter its value. When two text values are compared, the comparison is determined by the collation defined for the values.

4. The blob storage class has the highest value. Any value that is not of class blob will always be less than a value of class blob. blob values are compared using the C function memcmp().

## Views
</br>

		create view name as select-stmt;
		
<b>The contents of views are dynamically generated. Thus, every time you use details, its associated SQL will be reexecuted, producing results based on the data in the database at that moment. </b>

to drop a table: 
	
		drop view name;
		
## Indexes

	create index [unique] index_name on table_name (columns)
		
To remove an index, use the drop index command, which is defined as follows:

	drop index index_name;
	
## Triggers

	create [temp|temporary] trigger name
	[before|after] [insert|delete|update|update of columns] on table
	action
	
A trigger is defined by a name, an action, and a table. The action, or trigger body, consists of a series of SQL commands. Triggers are said to fire when such events take place. Furthermore, triggers can be made to fire before or after the event using the before or after keyword, respectively. Events include delete, insert, and update commands issued on the specified table. 
### Update Triggers
Update triggers, unlike insert and delete triggers, may be defined for specific columns in a table. 
		
	create trigger name
	[before|after] update of column on table
	action
	
an example of how the command works:

	create temp table log(x);

	create temp trigger foods_update_log update of name on foods
	begin
	insert into log values('updated foods: new name=' || new.name);
	end;

SQLite provides access to both the old (original) row and the new (updated) row in update triggers. The old row is referred to as old and the new row as new. ou could have just as easily recorded new.type_id or old.id.

### Error handling

In order to raise an error we use the following function in SQlite:

	raise(resolution, error_message);
	
The first argument is a conflict resolution policy (abort, fail, ignore, rollback, and so on). The second argument is an error message. 

## Rolling back the changes
If you want to test something on a database you could use the follwoing command to avoid changing the data in the database (It is always the best that you create a new instance of the database when you want to do tests)

	begin;
	<Do the tests>
	rollback;