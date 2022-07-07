# **Case Study 1: Danny's Diner**
### Solution
### [View the SQL code](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/CaseStudy1SQL8WeekChallenge.sql)
---
**Case Study Questions:**

1. What is the total amount each customer spent at the restaurant?
``` SQL
SELECT CustomerID, SUM(PRICE)
FROM DannysDinner A
JOIN Menu B
ON A.ProductID =B.ProductID
GROUP BY CustomerID
```
Code Explanation: This is straightforward. We only need to join the 2 tables (DannysDinner and Menu ) together using a column common to both tables (ProductID), Then sum the price column.

### Output:

![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/2022-07-06%20(3).png)  



Customer A spent the highest amount, followed by B and C respectively

---
2. How many days has each customer visited the restaurant?
```SQL
SELECT CustomerID, COUNT(DISTINCT OrderDate)as NumDays
FROM Dinner..DannysDinner
GROUP BY CustomerID
```
Code Explanation: COUNT(DISTINCT) was used to strike out multiple times visits per day

### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques2.png)

Customer B visited the restaurant the most followed by customers A and C

---
3. What was the first item from the menu purchased by each customer?
```SQL
WITH CTE_FirstMenu as 
(SELECT  CUSTOMERID, OrderDate, ProductName, 
DENSE_RANK() OVER (PARTITION BY CUSTOMERID ORDER BY OrderDate) as MenuNum 
FROM Dinner..DannysDinner A
JOIN Dinner..Menu B
ON A.ProductID =B.ProductID)
SELECT  DISTINCT CUSTOMERID, OrderDate, ProductName
FROM CTE_FirstMenu
WHERE MenuNum = 1
```


### Output:

![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques3.png)

Customer A purchased Sushi and Curry, while B and C purcahsed Sushi and Ramen respectively on their first order.
Code Explanation: 

* The [Dense_Rank](https://www.sqlshack.com/overview-of-sql-rank-functions/) function was used to rank the products' purchase order according to the order date.
*  [Partition BY](https://www.sqlshack.com/sql-partition-by-clause-overview/) is used like a GROUP BY FUNCTION, in this case, CustomerID was grouped into A, B, and C, then it was ranked based on the orderdate (first-order date get rank 1, second-order date get rank 2, shey you get.....)

* run the CTE table first to have a glance of what it looks like 
 ```SQL
SELECT  CUSTOMERID, OrderDate, ProductName, 
DENSE_RANK() OVER (PARTITION BY CUSTOMERID ORDER BY OrderDate) as MenuNum 
FROM Dinner..DannysDinner A
JOIN Dinner..Menu B
ON A.ProductID =B.ProductID
```
* The first order date recieved a rank 1

---
4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT TOP 1 ProductName, COUNT(ProductName) as NumTimesPurchased
FROM Dinner..DannysDinner A
JOIN Dinner..Menu B
ON A.ProductID =B.ProductID
GROUP BY ProductName
ORDER BY COUNT(ProductName) DESC
```
### Answer:
![Image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques4.png)

Code Explanation: 
* Run the CTE table first to have a view of the output-Just like the previous question.
* top 1 is the same as limit 1- meaning, get me the first row, however, the order date is in descending order, so the highest value comes first in the first row
---

5. Which item was the most popular for each customer?
```SQL
WITH CTE_PopMenu as 
(SELECT  CUSTOMERID,  ProductName, COUNT(ProductName) AS ProductCount,
DENSE_RANK () OVER (PARTITION BY CUSTOMERID ORDER BY COUNT(ProductName)DESC) as PopProduct
FROM Dinner..DannysDinner A
JOIN Dinner..Menu B
ON A.ProductID =B.ProductID
GROUP BY CUSTOMERID, ProductName
)
SELECT CUSTOMERID, PRODUCTNAME,  ProductCount
FROM CTE_PopMenu 
WHERE  PopProduct = 1
order by CustomerID, ProductCount DESC
```

### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques5.png)

This output further validates question 4, as Ramen has been the most ordered food item. 

Code Explanation: 

* We counted the number of times a food was ordered (COUNT(ProductName)) then we ranked the food with the most order rank 1 using DENSE_RANK  function after ordering the count of food in descending order so that the highest number would be ranked 1

---

6. Which item was purchased first by the customer after they became a member?

```SQL
WITH CTE_ProductRank as (
	SELECT  B.CustomerID, A.JoinDate, B.OrderDate, B.ProductID,
	RANK () OVER (PARTITION BY JoinDate ORDER BY OrderDate) as Ranks 
FROM Dinner..Members A
JOIN Dinner..DannysDinner B
ON A.CUSTOMERID = B.CUSTOMERID
WHERE JoinDate >= OrderDate)
SELECT E.CustomerID, E.JoinDate, E.OrderDate, Ranks, C.ProductName 
FROM CTE_ProductRank E
JOIN Dinner..Menu C
ON E.ProductID = C.ProductID
WHERE Ranks =1
```
### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques6.png)

Customer A purchased Curry while Customer B purchased Sushi first after they became members

Code Explanation 
* The member's table contains the date when each member joined. So far only A and B are members. So we joined the Member's table to the DannyDinner's table to extract the rows where the joindate = order date or when the join date is greater than the orderdate WHERE JoinDate >= OrderDate, Then we give the orderdate ranks, so that the first order ranks 1, and the next 2

* the output of the CTE table containing A and B customers' order date details is then joined to the Menu's table to extract the food they ordered first with Rank 1
---

7. Which item was purchased just before the customer became a member

```SQL
WITH CTE_ProductRank as (
	SELECT  B.CustomerID, A.JoinDate, B.OrderDate, B.ProductID,
	RANK () OVER (PARTITION BY JoinDate ORDER BY OrderDate DESC) as Ranks 
FROM Dinner..Members A
JOIN Dinner..DannysDinner B
ON A.CUSTOMERID = B.CUSTOMERID
WHERE JoinDate  < OrderDate)
SELECT E.CustomerID, E.JoinDate, E.OrderDate, Ranks, C.ProductName 
FROM CTE_ProductRank E
JOIN Dinner..Menu C
ON E.ProductID = C.ProductID
WHERE Ranks =1
```
### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques7.png)

Customer A ordered curry and sushi, while Customer B ordered sushi just before they became members.

Code Explanation 
* The catch is to obtain all the order date data before both customers became members(WHERE JoinDate  < OrderDate), then order the column in decending order so that the last day before the joined date would be on top.

* The rank function would give positions, which would be recalled in the Where function WHERE Ranks =1.
----

8. What is the total items and amount spent for each member before they became a member?
```SQL

WITH CTE_ProductRank as (
	SELECT  B.CustomerID, A.JoinDate, B.OrderDate, B.ProductID
FROM Dinner..Members A
JOIN Dinner..DannysDinner B
ON A.CUSTOMERID = B.CUSTOMERID
WHERE JoinDate  > OrderDate)
SELECT E.CustomerID, COUNT(ProductName)as ItemsNum,SUM(C.Price ) as TotalSpent
FROM CTE_ProductRank E
JOIN Dinner..Menu C
ON E.ProductID = C.ProductID
GROUP BY E.CustomerID


```
### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques8.png)

Customer A spent a $25 on two items while Customer B spent $40 on three items

Code Explanation:
* Create a CTE table, join Members and DannyDinner's tables together to extract the orderdate after the customers became members (WHERE JoinDate  > OrderDate) 

* Then extract the count of items purchased  and sum up their prices 

---

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```SQL
WITH CTE_PointMultiplier as
(
SELECT A.CustomerID, B.ProductName, B.Price,
CASE
WHEN ProductName = 'sushi' THEN  Price* 2
ELSE Price
END Sushi2
FROM Dinner..DannysDinner A
JOIN Dinner..Menu B
ON A.ProductID = B.PRODUCTID) 
SELECT CustomerID, SUM(Sushi2 *10) Points
FROM CTE_PointMultiplier 
GROUP BY CustomerID
```
### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques9.png)

Customer B got the highest point of 940, followed by A and C.

Code Explanation:

* There are 2 tasks- The first is to equate each dollar to a 10 point and then 2x multiplier for sushi, meaning while other items get 10 points, sushi would get 20 points.

* create a CTE point. here you could decide to deal with everything ( i.e Price *2 for sushi and Price*10 for all items), but I chose to do it one after the other.

---

10. In the first week after a customer joins the program (including their join date), they earn 2x points on all items, not just sushi. How many points do customers A and B have at the end of January?

```SQL
WITH CTE_Date as (
   SELECT *,
      DATEADD(DAY, 6, JoinDate) AS ValidDate, 
      EOMONTH('2021-01-31') AS LastDate
   FROM Dinner..Members)   
 SELECT A.CustomerID,
   SUM(
		CASE 
		WHEN ProductName ='Sushi' THEN Price*20
		WHEN OrderDate BETWEEN JoinDate and ValidDate THEN Price*20
		ELSE Price*10
		End) as Points
   FROM CTE_Date A
   JOIN Dinner..DannysDinner B
   ON A.CustomerID =B.CustomerID
   JOIN Dinner..Menu C
   ON B.ProductID = C.ProductID
   WHERE B.OrderDate < A.LastDate
   GROUP BY A.CustomerID
```
### Output:
![image](https://github.com/olusolaolagunju/Case-Study-1-DannysDinner-SQL-Challenge/blob/main/image/Ques10.png)

Customer B got the highest point of 940, followed by A and C.

Code Explanation:
* Two task here 
1. The first is to create 2 columns that represent one week after customers became members. a week is seven days and since the joindate is included in the question, this means Six days and is added to the join date DATEADD(DAY, 6, JoinDate) AS ValidDate, then the second column would be for dates after the joindate but before the month of February, that is the last day of January EOMONTH('2021-01-31') AS LastDate (very tricky right?, I know) 
2. The second task, is to find the order date that falls between OneWeekAfterJoinDate and Lastday in January and then scoring point as appropriate.
---



### **Thank you for reading, please don't hesitate to contribute to the code when you fish out an error. See you next time. XOXO** 
