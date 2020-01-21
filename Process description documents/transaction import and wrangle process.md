# Notes on the process

I have broken the process into 4 key parts

1. import new extracts.R (~140 lines)
2. import the categorisations done file.R (~140 lines)
3. detect presence of keywords.R (~110 lines)
4. save the new cats done excel file.R (~40 lines)
5. run the process.R (~10 rows)


## 1. import new extracts.R

Import all the key extract csv files from the "transactions" folder:
* cba
* ing
* paypal
* upbank
* cash

If any of these files do not exist, create a blank dataframe  (just the correct column headers, no rows) that will go in place of that import object.

Process each csv file to have only the following headers:
* Date
* Amount
* Description
* data_source

The paypal extract requires special attention because there are multiple rows for transactions requiring international currency exchanges.

[Cash out transactions could also be processed here... appending the description with "cash out -transaction", "cash out -cashout amount"]




## 2. import the categorisations done file.R
1. Source script 1
2. If the "Categorisations done.xlsx" file **doesn't** exist, create a dataframe of just the column headers (no rows) of the Categorisations and Categories sheets, to go in place of these uploads.
3. Import the "Categorisations" sheet from the "Categorisations done.xlsx" file. 
	- Convert Date to a date column type
	- Convert Amount to a numeric column type
	- Turn any 1's in the Cat Level 1 column to their suggested Cat Lavel 1.
	- Trim and uppercase any keywords (we want to keep any instances of these identical)
	- Trim and uppercase and Cat Level 1s (ditto to above).
3. Import the "Categories" sheet from the "Categorisations done.xlsx" file. 
	- filter to only rows containing keywords
	- remove the "type" column -we will be redermining this at a later stage.
	- trim and upper the Keyword and Cat Level 1 columns 
4. Collect a list of all keywords in both the categories and cats done tables:
	- find keywords in the categories table that exist in cats done (append "current existing" to "type" column)
	- find keywords only present in the categories table ("historical")
	- find keywords only in the cats done table ("new keywords")
	- bind all these together
5. From the list of all keywords, determine if any are contained in any of the others (create the "seconded" column).	
6. We may have new Cat Level1s contained in ether of our newly imported cats done or categories tables. We need to determine how to match each keyword to a Cat level 1.
	- For the cats done table -count rows for each keyword and Cat Level 1 combo where cat level 1s are present. There may be instances where the same keyword is assigned a different cat Level1, so keep the most common (or latest) assignment only. Create a warning in the warning column, if multiple were present.
	- For the categories table, much the same. Bind with the rows of the cats done summary (above) and take the distinct of Keyword because we want to give priority to those categorisation assignments in the categories table (if there is a conflict).
7. Left join the complete list of keywords with their categorisations detected (the second object above). This is going to become our replacement "categories" table.
8. We now add the detemined categorisations to the full list of keywords created earlier (with that seconded columns added). We are going to use this to test for their presence in any new transaction descriptions.


## 3. detect presence of keywords.R

1. Source script 2 (which in turn sources the one before it).
2. From the Cats done table, isolate the rows that already have Cat Level1s assigned (no need to touch these).
3. Isolate the rows that don't have any Cat Level1s assigned. Let's see if we can find keywords in their descriptions.
4. From the transactions imported from the extract files
	- anti_join with the rows from the object that contains the historical data that have already had Cat Level1s  determined.
	- anti-join with those historical data that didn't have any Cat Level1s determined
	- bind_rows with those historical data without any Cat Level1s (removing the Keyword, Suggested Category and Catagory Level1 columns, as we do). These two steps allow us to avoid duplication, retain historical rows not yet categorised, keep any additional notes that have been added to these historical rows and also re-evaluate keyword matches using all the newly collected data.
	- Create a row_id for each distinct transaction row.
	- Match every possible keyword to every transaction row.
	- Uppercase all descriptions and create a column ("contains_test") that test if the keyword is present.
5. Isolate those transactions without any keyword matches
6. Isolate those transactions that had atleast 1 keyword match
	- If they only had 1, that's great we'll use that keyword
	- If they had multiple keywords but one was the "seconded" (contained within) to the other (or others), just use the one that wasn't contained.
	- If they had multiple keywords and there was no key winner (as above), pick one row and replace the keyword with a bit of a warning ("_Multiple_").
7. Bind all the objects (with unique new and historical transactions) together.
	- Add an additional "in_scope" column, decided by a predetermined minimum date of where we want to focus our efforts.
	- Arrange the columns (include an everything(), as the end-user may have added additional not columns... and we don't want to delete these.  
	- sort descending by Date.


## 4. save the new cats done excel file.R

1. Source script 3 (which in turn sources all those before it).
2. Create an Excel workbook ("Categorisations done.xlsx" containing the relevant data for the "Categorisations" and "Categories" sheets.
3. *Hide* those rows that have been deemed to not be in scope.
4. Overide the existing Excel file (provided it is not currently open).


## 5. run the process process.R

If the "transactions" and "categorisations done" folders do not exist in the curernt directory, create them. The process will need these.

Run the "4 save the new cats done excel file.R" script (which in turns runs all those before it).












