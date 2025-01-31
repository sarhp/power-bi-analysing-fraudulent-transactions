# PowerBI DAX measures

<br>

1. Calculate discrepancies between transaction balances
```DAX
senderBalDiscrepency = fraud[senderOpeningBal] – fraud[senderClosingBal]

payeeBalDiscrepency = fraud[payeeOpeningBal] – fraud[payeeClosingBal]
```

<br><br>

2. Convert step(hour) to day e.g. steps 1-23 as day 1, steps 24-47 as day 2... etc
```DAX
day = INT(DIVIDE([hour] - 1, 23)) + 1
```

<br><br>

3. Index each row within a day, starting from 1 and resetting to 1 at the start of a new day
```DAX
indexWithinDay = 
RANKX(
    FILTER(
        'fraud', 
        [day] = EARLIER([day])  
    ),
    [index],    
    , 
    ASC,            
    DENSE              
)
```

<br><br>

## Combining `day` and `index` in the same column

<br>

4. Sort index hierachy numbers as 1.1, 1.2, 1.3... instead of 1.1, 1.10, 1.100, 1.2, 1.3 etc. 
To to this, we shift the index to the right side by adding ‘0000000’.
The day will appear first, followed by the index.
```DAX
SortKey = [day] * 1000000 + [indexWithinDay]
```
Output should look like: 1000001, 1000002, 1000003...1000010, 1000100 etc

<br><br>

5. Concatenate `day` and `indexWithinDay`, separating them with a dot to distinguish the day from the index.
```DAX
dayAndIndex = 
VAR Day = [day]
VAR Index = [indexWithinDay]
RETURN FORMAT(Day, "0") & "." & FORMAT(Index, "0000000")
```
Output should look like: 1.000001 (day 1, index 1), 1.000002, 1.000003...1.000010, 1.000100 etc


