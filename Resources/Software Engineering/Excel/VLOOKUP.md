# VLOOKUP

```less
=VLOOKUP (What you want to look up, the range in which you want to look for it, the column number in the range containing the value to return, return an Approximate or Exact match – indicated as 1/TRUE, or 0/FALSE)
```

You always write the formula in the sheet which has the values that we want to look up.

For selecting entire column as range in Excel, do a A2:A, meaning everything in the column from A2 onwards; or A2:B; meaning everything in the first two columns.

Columns to be returned are indexed from 1 and not 0.

An example for syntax

```java
=ARRAYFORMULA(IFERROR(VLOOKUP(<Range of cells with NLC FSNs>, <Table to fetch data from>!<Range to fetch data from>, <Array of columns to fetch>, 0), "Not Available"))
```

```java
=ARRAYFORMULA(IFERROR(VLOOKUP(A2:A416, Tax!A2:B1870, {1,2}, 0), "Not Available"))
```

<aside>
📌 **WARNING** VLOOKUP WILL ONLY SEARCH FOR THE GIVEN VALUE IN THE FIRST COLUMN OF THE SPECIFIED RANGE (timestamp 12:50 of [Google Sheets VLOOKUP Function Tutorial - INDEX & MATCH, QUERY - Part 1](https://youtu.be/0rWeMHdWvOc?t=762))
This also means you can only return values that are to the right of your range column. To return values from the left, [you'll have to reorder your columns](ARRAYS.md), and then search from that array range.

</aside>

<aside>
📌 **WARNING** It won't match 5 and '5' because one is a number and other is a string.

</aside>

- This is a perfect explanation: [https://support.microsoft.com/en-us/office/vlookup-function-0bbc8083-26fe-4963-8ab8-93a18ad188a1](https://support.microsoft.com/en-us/office/vlookup-function-0bbc8083-26fe-4963-8ab8-93a18ad188a1)
- This is a more detailed explanation: [https://www.youtube.com/watch?v=0rWeMHdWvOc&ab_channel=LearnGoogleSpreadsheets](https://www.youtube.com/watch?v=0rWeMHdWvOc&ab_channel=LearnGoogleSpreadsheets)
- VLOOKUP will only look for one piece of data in your given range at a time. If you want to look up multiple pieces of data (from different rows) and populate cells in those respective rows, you'll have to use ArrayFormula.