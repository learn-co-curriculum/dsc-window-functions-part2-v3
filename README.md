# Introduction to Window Functions Part 2

## Introduction
We looked at various window functions over some window. In this lesson, we'll look at how to compare preceding or following rows using other window functions, like `LAG()` and `LEAD()`.

## Objectives
- Learn when to use `LAG()` and `LEAD()`

### What these functions do?
These functions compare values between records, either preceding or following, compared to the current row that you're looking at. Like other window functions, the column to be compared needs to be specified, and optionally, the column that is getting partitioned by. `LAG()` compares values between the preceding row and the current row; `LEAD()` compares with the following row and the current row.

### How to use them

Let's move over to [db-fiddle](https://www.db-fiddle.com/).
Enter the following DDL to create and insert some rows into the table `SALES`.

```
CREATE TABLE SALES (
  id INTEGER PRIMARY KEY,
  name TEXT,
  sales INTEGER
);

INSERT INTO SALES VALUES (1, "Javier", 87000);
INSERT INTO SALES VALUES (2, "Stephanie", 65000);
INSERT INTO SALES VALUES (3, "Alice", 68000);
INSERT INTO SALES VALUES (4, "Roy", 79000);
```

Now, copy-paste this query on the right panel to see the result below.
```
SELECT name, sales,
  LAG(sales) OVER(ORDER BY sales) as previous_sale_value
FROM SALES;
```

The result looks like this below:

| name      | sales | previous_sale_value |
| --------- | ----- | ------------------- |
| Stephanie | 65000 | null                |
| Alice     | 68000 | 65000               |
| Roy       | 79000 | 68000               |
| Javier    | 87000 | 79000               |


The `null` is there because there is no previous row to compare with. `LAG()` function literally pulls the data from the previous rows.
Let's look at how to use `LEAD()`. All you need to do is replace `LAG` with `LEAD`, and you'll see something like below.

| name      | sales | previous_sale_value |
| --------- | ----- | ------------------- |
| Stephanie | 65000 | 68000               |
| Alice     | 68000 | 79000               |
| Roy       | 79000 | 87000               |
| Javier    | 87000 |  null               |

The null is there in the fourth row because there is no next row to compare with.

So these two functions look like they just replicate data. How can we use them better?


### Use differences!

Let's copy-paste this DDL again.

```
CREATE TABLE ANNUAL_SALES (
  id INTEGER PRIMARY KEY,
  year INTEGER,
  sales INTEGER
);

INSERT INTO ANNUAL_SALES VALUES (1, 2016, 87000);
INSERT INTO ANNUAL_SALES VALUES (2, 2017, 65000);
INSERT INTO ANNUAL_SALES VALUES (3, 2018, 68000);
INSERT INTO ANNUAL_SALES VALUES (4, 2019, 79000);
INSERT INTO ANNUAL_SALES VALUES (5, 2020, 83000);
INSERT INTO ANNUAL_SALES VALUES (6, 2021, 92000);
INSERT INTO ANNUAL_SALES VALUES (7, 2022, 98000);
```

And run this query on the right panel.

```
SELECT 
   year,
   sales AS current_total_sale,
   LAG(sales) OVER(ORDER BY year) AS previous_total_sale,
   sales - LAG(sales) OVER(ORDER BY year) AS difference
FROM ANNUAL_SALES;
```

You should see result like this below:

| year | current_total_sale | previous_total_sale | difference |
| ---- | ------------------ | ------------------- | ---------- |
| 2016 | 87000              |                     |            |
| 2017 | 65000              | 87000               | -22000     |
| 2018 | 68000              | 65000               | 3000       |
| 2019 | 79000              | 68000               | 11000      |
| 2020 | 83000              | 79000               | 4000       |
| 2021 | 92000              | 83000               | 9000       |
| 2022 | 98000              | 92000               | 6000       |

Because we used `LAG` function, we expected to see null values for the first rows. The useful column here is `difference` where we can see the difference between the previous year and current year's sales. It makes sense to use `LAG` function here because we mostly want to compare the sales to the previous years.


### Summary

We looked at how to use `LAG()` and `LEAD()` functions using examples. `LAG()` pulls data from the previous row, and `LEAD()` pulls data from the following rows. Depending on which kind of data you work with, you can choose the function of your choice to present your data.
