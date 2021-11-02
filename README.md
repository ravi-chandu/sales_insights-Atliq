# AtliQ - Sales Insights Data Analysis - Power BI


`Power BI - Dashboard`
<iframe width="100%" height="373.5" src="https://app.powerbi.com/view?r=eyJrIjoiYjYxNmIwMjAtN2U2OC00YWQ0LTljMTMtMDU2OTE4OGNmYzFiIiwidCI6ImU5ZjMyNWZkLTkzMjYtNDJjNi1iNGNjLTBlZmJhNWQ4OTE3OCJ9&pageName=ReportSectiona170e58523a9c686e839" frameborder="0" allowFullScreen="true"></iframe>


## Extract

Data is stored in mysql localhost

Extract the data -
we need to connect to `MySQL Database`
- Server - localhost
- Database - sales(I saved data as sales)

then import the 5 tables(Transform) into Power Query

*all the tables are fine so load into Power bI desktop*

<br/>

here Date Table doesn't have all the details so I created custom date table

## 1) Custom Date Table

```DAX
Date table = 
ADDCOLUMNS(
    CALENDAR(DATE(2017,10,4),DATE(2021,12,31)),
    "Year",YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Quarter No", QUARTER([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Month No", MONTH([Date])
    )
```
choose the starting and ending Date and specify the required columns.

we can also create week and day same way depends on requirememnt.

<br/>

## 2) Automatic Forecast using Analytics

![image](https://user-images.githubusercontent.com/92777166/139290227-7bba9706-db23-402f-aa42-b1091e5514e1.png)


We have option to create Forecast in analytics under forecast by filling the required options.

## 3) Manual(Dynamic) forecast using Dax

![image](https://user-images.githubusercontent.com/92777166/139291368-88d80c3f-62ed-4bc3-bfe0-2bc895c522e8.png)

here forecast is based on last year sales - how much percentage change we expect from previous year

```DAX
Profit forecast = 
   VAR forecast =
          CALCULATE(
                [Total Profit],
                DATEADD('Date table'[Date], -1,YEAR)
          )*
(1 + 'Profit growth rate %'[Profit growth rate % Value])
VAR RESULT =
    If(
       MAx('sales transactions'[order_date]) < 26/6/2020,
       forecast,
       [Total Profit]
   )
RETURN 
RESULT
```

for dynamically changing the forecast - I used What If Parameter `'Profit growth rate %'[Profit growth rate % Value]`

<br/>

## 3) Dynamic Top N

![image](https://user-images.githubusercontent.com/92777166/139294908-c0b59692-4899-4400-b661-a53f9bb4a076.png)


here I created like this - Top N customers & others(remaining all customers sum)

here we can change Top N dynamically using what if parameter

and Top N combined % share in Total sales 

Here I created some measures to get what I needed

*In this whole Dashboard majority of time I spent on this topic - after learning and practicing many tutorials I finally made it, how I wanted❤️*


1) First of all we need to create "others" in the customers list <br>

  Creating "others" in customers column 
  
```DAX
customers names = 
UNION(
    ALLNOBLANKROW('sales customers'[custmer_name]),
    {"others"}
)
```

Here we added others in the cutomers list

<br/>

2) Creating Top N sales & others dynamically 
```
Top N sum sales = 
Var Topcus = 
 TOPN(
      'TOP customers revenue'[TOP customers Value],
      ALLSELECTED('customers names'),
      [Total Revenue]
  )
Var Topcussales =
CALCULATE(
    [Total Revenue],
     KEEPFILTERS(
         Topcus   
    )
     
)
VAR othersales =
 CALCULATE(
     [Total Revenue],
     ALLSELECTED('customers names')
 ) - 
 CALCULATE(
     [Total Revenue],
     Topcus
 )
VAr currentcus = SELECTEDVALUE('customers names'[all custmer_name])
Return
 IF(
     currentcus <> "others",
     Topcussales,
     othersales
 )
 ```
 <br/>
 
 3) Creating Rank and order (others should be last) 
  
  sort by this rank
  ```
  Rank sales = 
IF(
    [Top N sum sales] <> BLANK(),
    RANKX(
        TOPN(
            SELECTEDVALUE('TOP customers revenue'[TOP customers]),
            ALLSELECTED('customers names'),
            [Total Revenue]
        ),
        [Total Revenue],,DESC,Dense
    )
)
```

and some more measures 

for dynamic Text and card in donut chart

```
profit desc = 
VAR Topcus = 
 CALCULATE(
     [Total Profit],
     TOPN(
        SELECTEDVALUE('Top customer Profit'[Top customer Profit]),
        ALLSELECTED('customers names'[all custmer_name]),
        [Total Profit]
     )
 )
 VAR TOPcuspercent =
  DIVIDE(
      Topcus, 
      [Total Profit]
  )
Return 
"Top " & SELECTEDVALUE('Top customer Profit'[Top customer Profit]) & " generates " & FORMAT(TOPcuspercent, "#.##% profit ")
```


`Thank you😊`



