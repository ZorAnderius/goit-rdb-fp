# Final project

## Content

- [Task1](#Task1)
  - [Task1-1](#Task1-1)
  - [Task1-2](#Task1-2)
  - [Task1-3](#Task1-3)
- [Task2](#Task2)
- [Task3](#Task3)
- [Task4](#Task4)
- [Task5](#Task5)



## Task1

Download data:

 - *Create the pandemic schema in the database using the SQL command.*
 - *Select it as the default schema using the SQL command.*
 - *Import the data using the Import wizard as you did in topic 3.*
 - *Look at the data for context.*


### Task1-1
*Create the pandemic schema in the database using the SQL command.*
 

SQL queries:

``` mysql 
    CREATE SCHEMA pandemic;
```

Result:

![](/Task1/1.Create_schema.png)

### Task1-2
*Select it as the default schema using the SQL command.*

SQL queries:

``` mysql 
    use pandemic;
```

Result:

![](/Task1/2.Default_schema.png)

### Task1-3
*Import the data using the Import wizard as you did in topic 3.*

Result:

![](/Task1/3.Import.png)

![](/Task1/4.Evidence_of_exist_table.png)

### Task2

Normalize the infectious_cases table to 3rd normal form. Save two tables with normalized data in the same schema.

Solution: 

#### Українська версія
Аналіз підтверджує, що вихідна таблиця не відповідає 3NF (третій нормальній формі). Цей висновок підтверджується наявністю таких полів, як Entity і Code, які демонструють чітку функціональну залежність. Зокрема, назву країни можна отримати з її коду, і навпаки, код можна використовувати для ідентифікації країни. Ця функціональна залежність порушує 3NF, який вимагає, щоб усі неключові атрибути повністю функціонально залежали лише від первинного ключа.

Крім того, існує значна надмірність у полях Entity та Code, що призводить до неефективного використання пам’яті. Поле Entity, що представляє назви країн, має середню довжину 9 символів, що вимагає приблизно 11 байт на запис, тоді як поле Code використовує 5 байт. Враховуючи, що таблиця містить 10 521 рядок, загальне споживання пам’яті для цих двох полів обчислюється як:

$$
10521×(11+5)=168336 байт
$$

Однак кількість унікальних пар Entity та Code становить лише 245. Нормалізуючи дані та створивши окрему таблицю, як-от Countries, для зберігання цих унікальних значень, а також шляхом призначення типів даних VARCHAR(60) для Entity та VARCHAR(15) для Code, ми можемо значно зменшити використання пам’яті. Нова таблиця споживатиме:

$$
245×(10+4)=3430 байт
$$

Крім того, враховуючи пам’ять, необхідну для первинних ключів (припускаючи 4-байтовий ідентифікатор для кожного рядка), загальна пам’ять для таблиць Countries і Infectious_cases буде:

$$
4×(10521+245)=43 064 байт
$$

Таким чином, загальна пам'ять, необхідна для нормалізованої структури, буде:

$$
43 064+3 430=46 494 байт
$$

Це призводить до того, що обсяг пам’яті приблизно в 3,6 рази менший, ніж у ненормализованої таблиці. Крім того зі збільшенням кількості даних, ця цифра буде також зростати у арифметичній прогресії.

Крім того, оскільки немає інших сильних кореляцій між іншими полями, подальша нормалізація вважається непотрібною. Надмірна нормалізація може призвести до проблем із продуктивністю через додаткову складність операцій JOIN або вкладених запитів. Таким чином, остаточний нормалізований дизайн складатиметься з двох таблиць: Countries і Infectious_cases, які дотримуються 3NF і оптимізують як використання пам’яті, так і продуктивність.

#### English version
The analysis confirms that the source table is not in 3NF (Third Normal Form). This conclusion is supported by the presence of fields such as Entity and Code, which demonstrate a clear functional dependency. Specifically, a country's name can be derived from its code, and vice versa, the code can be used to identify the country. This functional dependency violates 3NF, which requires that all non-key attributes must be fully functionally dependent only on the primary key.

Additionally, there is significant redundancy in the Entity and Code fields, leading to inefficient memory usage. The Entity field, representing country names, has an average length of 9 characters, requiring approximately 11 bytes per entry, while the Code field uses 5 bytes. Given that the table contains 10,521 rows, the total memory consumption for these two fields is calculated as:

$$
10521×(11+5)=168,336 bytes
$$

However, the number of unique Entity and Code pairs is only 245. By normalizing the data and creating a separate table, such as Countries, to store these unique values, and by assigning the data types VARCHAR(60) for Entity and VARCHAR(15) for Code, we can significantly reduce memory usage. The new table would consume:

$$
245×(10+4)=3,430 bytes
$$

Additionally, considering the memory needed for primary keys (assuming a 4-byte identifier for each row), the total memory for both the Countries and Infectious_cases tables would be:

$$
4×(10521+245)=43,064 bytes
$$

Thus, the total memory required for the normalized structure would be:

$$
43,064+3,430=46,494 bytes
$$

This results in a memory footprint approximately 3.6 times smaller than that of the non-normalized table.

Furthermore, since there are no other strong correlations among the remaining fields, further normalization is deemed unnecessary. Over-normalization might lead to performance issues due to the additional complexity of JOIN operations or nested queries. Therefore, the final normalized design will consist of two tables: Countries and Infectious_cases, which adhere to 3NF and optimize both memory usage and performance.

1)  Create table *'Countries'* and fill in data from *'entry_table'*

SQL queries:

``` mysql 
    use pandemic;

    CREATE TABLE IF NOT EXISTS countries (
        id INT PRIMARY KEY AUTO_INCREMENT NOT NULL,
        country_name VARCHAR(60),
        country_code VARCHAR(15)
    );

    INSERT INTO countries (country_name, country_code) 
    SELECT DISTINCT Entity, Code
    FROM entry_table;
```

Result:

![](/Task2/1.Create_countries.png)

2) Create table *'Infectious_cases'* and fill in data from *'entry_table'*

SQL queries:

``` mysql 
    use pandemic;

    CREATE TABLE IF NOT EXISTS infectious_cases (
        id INT PRIMARY KEY AUTO_INCREMENT NOT NULL,
        country_id INT,
        Year INT,
        Number_yaws TEXT,
        polio_cases TEXT,
        cases_guinea_worm TEXT,
        Number_rabies TEXT,
        Number_malaria TEXT,
        Number_hiv TEXT,
        Number_tuberculosis TEXT,
        Number_smallpox TEXT,
        Number_cholera_cases TEXT,
        FOREIGN KEY (country_id) REFERENCES countries(id)
    );

    INSERT INTO infectious_cases (
        country_id, 
        Year,
        Number_yaws,
        polio_cases,
        cases_guinea_worm,
        Number_rabies,
        Number_malaria,
        Number_hiv,
        Number_tuberculosis,
        Number_smallpox,
        Number_cholera_cases
    ) 
    SELECT c.id,
        e.Year,
        e.Number_yaws,
        e.polio_cases,
        e.cases_guinea_worm,
        e.Number_rabies,
        e.Number_malaria,
        e.Number_hiv,
        e.Number_tuberculosis,
        e.Number_smallpox,
        e.Number_cholera_cases
    FROM entry_table AS e
    JOIN countries AS c ON e.entity=c.country_name AND e.code=c.country_code;
```

Result:

![](/Task2/2.Create_infection_cases.png)


## Task3

Analyze the data:

 For each unique combination of Entity and Code or their id, calculate the average, minimum, maximum value and sum for the Number_rabies attribute.

 - *Sort the result by the calculated average in descending order.*
 - *Select only 10 lines to display.*


Solution: 


 SQL queries:

``` mysql 
    use pandemic;

    SELECT  c.country_name,
            c.country_code,
            AVG(i.Number_rabies) as average_number_rabies,
            MIN(i.Number_rabies) as min_number_rabies,
            SUM(i.Number_rabies) as sum_number_rabies
    FROM infectious_cases AS i
    JOIN countries AS c ON c.id=i.country_id
    WHERE Number_rabies IS NOT NULL
    GROUP BY c.country_name, c.country_code;
```

Result:

![](/Task3/1.AVG_MIN_SUM.png)
 
 
 - *Sort the result by the calculated average in descending order.*
 - *Select only 10 lines to display.*


1) *Sort the result by the calculated average in descending order.*

SQL queries:

``` mysql 
    use pandemic;

    SELECT  c.country_name,
            c.country_code,
        AVG(i.Number_rabies) as average_number_rabies,
        MIN(i.Number_rabies) as min_number_rabies,
        SUM(i.Number_rabies) as sum_number_rabies
    FROM infectious_cases AS i
    JOIN countries AS c ON c.id=i.country_id
    WHERE Number_rabies IS NOT NULL
    GROUP BY c.country_name, c.country_code;
    ORDER BY average_number_rabies DESC;
```

Result:

![](/Task3/3.Order_desc.png)



2) *Select only 10 lines to display.*

SQL queries:

``` mysql 
    use pandemic;

    SELECT  c.country_name,
            c.country_code,
        AVG(i.Number_rabies) as average_number_rabies,
        MIN(i.Number_rabies) as min_number_rabies,
        SUM(i.Number_rabies) as sum_number_rabies
    FROM infectious_cases AS i
    JOIN countries AS c ON c.id=i.country_id
    WHERE Number_rabies IS NOT NULL
    GROUP BY c.country_name, c.country_code;
    ORDER BY average_number_rabies DESC;
    LIMIT 10;
```

Result:

![](/Task3/2.limit.png)


 ## Task4

Construct a column of the difference in years.

For the original or normalized table for the Year column, build using the built-in SQL functions:

 - *an attribute that creates the date of January 1 of the corresponding year,*
 - *attribute equal to the current date,*
 - *attribute equal to the difference in years of the two columns mentioned above.*

SQL queries:

``` mysql 
    use pandemic;

    SELECT  Year,
            MAKEDATE(Year,1) as table_date,
            CURDATE() as `current_date`,
            DATEDIFF(CURDATE(), MAKEDATE(Year,1) ) as days_diff
    FROM infectious_cases;
```

Result:

![](/Task4/1.Date.png)



 ## Task5

Build your own function.

 Create and use a function that constructs the same attribute as in the previous task: the function should take as input a year value and return the difference in years between the current date and the date created from the year attribute (year 1996 → '1996-01-01 ').


SQL queries:

``` mysql 
    use pandemic;

    DELIMITER //

    CREATE FUNCTION CalculateDiff(year INT)
    RETURNS INT
    DETERMINISTIC
    NO SQL
    BEGIN
        RETURN IF(year IS NOT NULL AND year > 0, DATEDIFF(CURDATE(), MAKEDATE(year, 1)), 0); 
    END //

    DELIMITER ;

    SELECT  Year,
            MAKEDATE(Year,1) as table_date,
            CURDATE() as `current_date`,
            CalculateDiff(Year) as days_diff
    FROM infectious_cases;
```

Result:

![](/Task5/1.Create_function.png)


