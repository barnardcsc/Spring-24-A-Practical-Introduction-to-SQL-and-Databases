# A Practical Introduction to SQL & Databases

In this workshop, we'll cover the essentials of working with relational databases using SQL. You will learn how to store, retrieve, and manipulate data effectively. By the end of this workshop, you'll have a solid foundation in SQL and be ready to tackle your own database projects. 

### Learning goals
- Understand the importance and benefits of using a database
- Set up a database that meets your specific needs
- Develop a well-structured and efficient database schema
- Implement and create your database schema
- Perform essential database operations: create, update, delete, and query data

### PostgreSQL

There are many types of SQL databases available, such as Microsoft SQL Server, MySQL, Oracle, and SQLite. We'll use PostgreSQL, a powerful, open-source database, for this workshop. The concepts and skills you'll learn apply to other databases because the core principles remain consistent across different SQL implementations.

### Prerequisites

To make things easier, we'll access PostgreSQL through Supabase, a cloud database provider with a free plan. Supabase has a built-in editor and admin interface, so you don't need to download any extra software. Usually, you'd use a tool like DBeaver to interact with a database, but Supabase lets you write queries and manage your database inside of their web-based platform.

1. Create an account at [Supabase](https://supabase.com/dashboard/sign-up).
    1. Click `+ New Project`
    2. Fill out the `Name` & `Password` fields:
        - **Name**: `csc-workshop`
        - **Password**: Generated random
    3. Select a region (East US)
    4. Create and allow a couple of minutes for setup.
2. [Download Github repository ](https://github.com/barnardcsc/Spring-24-A-Practical-Introduction-to-SQL-and-Databases)

### Optional: Alternative Vendors

If you don't wish to use Supabase, PostgreSQL is open-source, so you can download and host it yourself.

- **Self-Hosting**: Download PostgreSQL from [the official website](https://www.postgresql.org/) and follow the installation instructions.

Other cloud vendors that offer PostgreSQL hosting include:

- AWS
- Google Cloud Compute
- Microsoft Azure
- Digital Ocean
- Render
- Neon

If you choose to host PostgreSQL yourself or use a different cloud vendor, you'll also need a database client. Here are some popular options:

- **Free Options**:
  - pgAdmin
  - DBeaver
- **Student Options**:
  - Datagrip (Free for students)
  - Postico (Discount for students, Mac only)
  - TablePlus (Preferred tool, discounted for students)

_Note: We cannot provide installation, troubleshooting, or support for any of these software._

### External Links & Resources

- [Workshop Github Repo](https://github.com/barnardcsc/Spring-24-A-Practical-Introduction-to-SQL-and-Databases)
- [PostgreSQL Exercises](https://pgexercises.com/)
- [Postgres Tutorials](https://www.postgresqltutorial.com/)
- [Official documentation](https://www.postgresql.org/docs/current/index.html)
- ["Awesome" Postgres Github Page](https://github.com/dhamaniasad/awesome-postgres)
- [The Art of PostgreSQL by Dimitri Fontaine](https://theartofpostgresql.com/)
- [Official Postgres repository](https://git.postgresql.org/gitweb/?p=postgresql.git;a=summary)

---

### 1.0 Why databases?

Why would use a dedicated SQL database like PostgreSQL instead of an Excel spreadsheet or Google Sheet?

- Scalability & Performance: Handles large data and multiple users; optimized for fast data retrieval.
- Data integrity: Enforces accuracy and consistency; spreadsheets lack built-in mechanisms.
- Security: Provides robust and detailed access controls.
- Complex queries: Supports powerful querying over large amounts of data

**SQL: Structured Query Language**
- Standardized: SQL ensures consistency across different database systems.
- Declarative: Describe what data you want, not how to retrieve it.
- Ubiquitous: Widely used and a valuable skill for data-related fields.
- Powerful: Efficiently store, retrieve, and analyze data with complex queries.
- Interoperable: Interact with databases from various programming languages.

**General Tips**
- Lowercase all column names
- Don’t use spaces or special characters inside of column names. 
- Only use underscores as the separator (`first name` v. `first_name`)
- Don’t use keywords as table or column names. You can find the list of [keywords here](https://www.postgresql.org/docs/current/sql-keywords-appendix.html).
- Strive to use the correct column types:
    - Don’t store dates as strings, etc. 
- Be extra careful with the `DELETE` command as it's very easy to accidentally delete data.


#### 1.1 Basic Operations

After you have created a project in Supabase, use the toolbar on the left side to navigate to the **SQL Editor** and run (⌘ + Return) the following commands. 

**Create table**

```SQL
CREATE TABLE "public"."states" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"name" TEXT NOT NULL UNIQUE,
	"abbreviation" TEXT NOT NULL UNIQUE,
	"population" INT NOT NULL,
	PRIMARY KEY ("id")
);
```

**Insert row**

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('New York', 'NY', 20000000);
```

Insert another row, but keep the 'NY' abbreviation. What happens? 

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('Virginia', 'NY', 8600000);
		
-- Unique constraint error
```

To fix the error: 

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('Virginia', 'VA', 8600000);
```

**Update row**

```SQL
UPDATE
	states
SET
	population = 8684000
WHERE
	name = 'Virginia';
```

### 1.2 Organizing City Data: Relational Tables, Foreign Keys, and One-to-Many Relationships

Imagine you want to store information about every city in New York (NY) and Virginia (VA). You might think of adding city data directly to an existing `states` table. However, this method has drawbacks:

**Not Recommended Approach:**

```SQL
-- This method is inefficient:
ALTER TABLE states
    ADD COLUMN city1 TEXT,
    ADD COLUMN city2 TEXT,
    ...;
```

**Issues with this Approach:**

- The number of columns would increase with each new city.
- Adding new attributes, like population for each city, requires even more columns.

**Solution: Create a Separate `cities` table:**

To manage this efficiently, we create a new table named `cities`:

```SQL
CREATE TABLE "public"."cities" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "state_id" INT NOT NULL,
    "name" TEXT NOT NULL,
    CONSTRAINT "state_id_fkey" FOREIGN KEY ("state_id") REFERENCES "public"."states" ("id"),
    PRIMARY KEY ("id")
);
```

In this setup, `cities` and `states` are linked through a `state_id` foreign key in the `cities` table. This creates a `one-to-many` relationship: each state can have multiple cities, but each city belongs to only one state.

**Next Steps - Adding City Data:**

Before adding city data, you'll need to check the state IDs in your database to ensure correct mapping.

```SQL
SELECT * FROM states
```

Then:


```SQL
INSERT INTO cities (state_id, name)
		VALUES (1, 'New York City'), (1, 'Buffalo'), (3, 'Alexandria'), (3, 'Virginia Beach')
```

### 1.3 Combining Data: Joining, Aliasing, and Creating Views

When working with databases, you'll often need to merge data from different tables. Let's say you want to see city data along with their corresponding states. You can achieve this by joining the `cities` and `states` tables:

**Example Query to Combine Data:**

```SQL
SELECT
    c.id,
    c.name AS "city",
    s.name AS "state"
FROM
    cities c
    JOIN states s ON s.id = c.state_id
```

**Addressing Column Name Ambiguity with Aliases:**

- When joining tables, identical column names (like `id`, `name`) in different tables can create confusion. 
- To clearly specify columns, we use aliases such as `cities c` and `states s`.
- Aliases can be any string and are placed after the table name. A common practice is using the first letter of the table name.
- If aliases are not used where column names overlap, you'll encounter errors like this:

```SQL
ERROR:  column reference "id" is ambiguous
```

**Handling Complex Queries with Views:**

- As queries grow in size and complexity, managing them can be challenging.
- A practical solution is to create views of commonly used queries. This simplifies future queries and improves readability.

**Creating and Using a View:**

You can create a view for the join operation above:

```SQL
CREATE VIEW vi_cities_with_states AS (
    SELECT
        c.id,
        c.name AS "city",
        s.name AS "state"
    FROM
        cities c
        JOIN states s ON s.id = c.state_id
);
```

Then, to retrieve data, simply query the view:

```SQL
SELECT * FROM vi_cities_with_states
```

Using views not only makes your SQL cleaner but also allows for easier data retrieval and maintenance.

---

### 2.0 Data Normalization Exercise

Data normalization is a technique that organizes data in a database efficiently by breaking it down into smaller, related tables. It reduces data redundancy and improves data integrity. 

By dividing data into separate tables and establishing relationships between them using keys, normalization ensures that each piece of data is stored only once, saving storage space and making the database easier to maintain and update without inconsistencies or errors.

**[Download the Exercise Workbook](https://github.com/barnardcsc/Spring-24-A-Practical-Introduction-to-SQL-and-Databases/raw/main/data-norm-exercise.xlsx)**


**Instructions:**

1. **Analyze the Denormalized Data**
   - Open the downloaded workbook.
   - Observe the data structure in its original form.
   - Ask yourself: What inefficiencies or redundancies do you notice? How might you reorganize the data more logically?

2. **Look at the Customers Example**
   - Navigate to the `Customers Example` tab.
   - Study how the data is restructured here. 
   - This tab serves as an example for what effective normalization looks like.

3. **Apply normalization to the Assignment**
   - Go to the `Assignment` tab.
   - Focus on the `item` column within the `sales` table.
   - Try to apply normalization principles to reorganize this data.
   - The goal is to separate out data in a way that reduces repetition.

---

### 3.0 Building a Normalized Database

Let's build the database from the data normalization exercise.

We'll create three tables: `customers`, `items`, and `sales`. 

**Customers Table:**

```SQL
CREATE TABLE "public"."customers" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "name" TEXT NOT NULL,
    "loyalty_member" INT NOT NULL DEFAULT 0, -- Default to 0 if no value is provided.
    "postal_code" TEXT, -- This field is optional.
    "date_of_birth" DATE, -- This field is optional.
    PRIMARY KEY ("id") -- Ensures each customer has a unique ID.
);
```

**Items Table:**

```SQL
CREATE TABLE "public"."items" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "name" TEXT NOT NULL,
    PRIMARY KEY ("id")
);
```

**Sales Table:**

```SQL
CREATE TABLE "public"."sales" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "customer_id" INT NOT NULL,
    "item_id" INT NOT NULL,
    CONSTRAINT "customer_id_fkey" FOREIGN KEY ("customer_id") REFERENCES "public"."customers" ("id"),
    CONSTRAINT "item_id_fkey" FOREIGN KEY ("item_id") REFERENCES "public"."items" ("id"),
    PRIMARY KEY ("id")
);
```

*Note: The `sales` table is created last because it references the other two tables.*


#### 3.1 Populating the Tables

Fill each table with relevant data:

**Populate `customers`:**

```SQL
INSERT INTO "customers" (name, loyalty_member, postal_code, date_of_birth)
    VALUES ('Mark', 0, '99524', '1980-02-01'), 
           ('Nia', 1, '85055', '1965-11-05'), 
           ('Fred', 0, '49036', '1977-06-15');
```

**Populate `items`:**

```SQL
INSERT INTO items (name)
    VALUES ('shoes'), ('pants'), ('shirt'); 
```

**Populate `sales`:**

```SQL
INSERT INTO sales (customer_id, item_id)
    VALUES (1, 1), (2, 2), (1, 1), (2, 2), (3, 2), (3, 3), (1, 3), (2, 1), (3, 3), (3, 3);
```

#### 3.2 Querying Data

**Find the Purchase Frequency for Each Item:**

```SQL
SELECT
    item_id,
    COUNT(item_id)
FROM
    sales
GROUP BY
    item_id
```

**Get Item Names with Purchase Frequency:**

```SQL
SELECT
    s.item_id,
    i.name,
    COUNT(s.item_id)
FROM
    sales s
    JOIN items i ON i.id = s.item_id
GROUP BY
    s.item_id,
    i.name
```

#### 3.3 Handling Monetary Values

There is a `money` column type, but I recommend you store monetary values as integers that are equal to the smallest fractional amount in your base currency:

- 1c = 1
- $5.34 = 534

**Add a `price` Column to `items`:**

```SQL
ALTER TABLE items ADD COLUMN price INT;
```

Check the table structure, then update item prices:

```SQL
SELECT * FROM items;
UPDATE items SET price = 3500 WHERE id = 1;
UPDATE items SET price = 2200 WHERE id = 2;
UPDATE items SET price = 1300 WHERE id = 3;
```

**Add a NOT NULL Constraint:**
- This constraint ensures all items have a price.

```SQL
ALTER TABLE items ALTER COLUMN price SET NOT NULL;
```

#### 3.4 Advanced Queries

**Calculate Total Revenue Per Customer:**

- Start by fetching item IDs and customer IDs from `sales`.
- Add prices by joining with the `items` table.
- Sum the prices and group by customer to get the total sales per customer.

```SQL
SELECT
    s.customer_id,
    SUM(i.price) as "total_sales"
FROM
    sales s
    JOIN items i ON i.id = s.item_id
GROUP BY
    s.customer_id
ORDER BY
    customer_id
```

**Create a View for Easy Access:**

Views simplify data retrieval, especially for complex queries.

```SQL
CREATE VIEW vi_total_sales_per_customer AS (
    SELECT
        s.customer_id,
        SUM(i.price) AS "total_sales"
    FROM
        sales s
        JOIN items i ON i.id = s.item_id
    GROUP BY
        s.customer_id
    ORDER BY
        customer_id)
```

And retrieve the data:

```SQL
SELECT * FROM vi_total_sales_per_customer
```

---

### 4.0 Vehicle Collision Database Project

This project involves working with a dataset of vehicle collisions in New York City for January 2022. The original dataset has been modified for manageability and focuses on approximately 7.8K records. 

- **CSV Files for Import**: Access the modified [CSV files](https://github.com/barnardcsc/Spring-24-A-Practical-Introduction-to-SQL-and-Databases/tree/main/csv-data).
- **Original Source Data**: The full dataset is available at [NYC Motor Vehicle Collisions - Crashes](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95).

#### 4.1 Data Inspection and Schema Development

First, analyze the data tab on [original source data](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95). Pay attention to:

- Columns dedicated to each vehicle involved in a collision.
  - `CONTRIBUTING FACTOR VEHICLE #`
- Most collisions involve only 1 or 2 vehicles, so the majority of row values are empty.
- Suggestions for Improvement:
  - Column names should be renamed, lowercased, and separated by underscores.
  - Find the primary key - COLLISION_ID is a good candidate, but need to check that it's NOT NULL and UNIQUE. Use Excel filters and Data > Remove Duplicates to check.

Normalize only certain columns like vehicle types, boroughs, and street names for simplicity.

#### 4.2 Creating the Tables

The database will consist of three main tables: `collisions`, `vehicles`, and `vehicle_collisions`.

**Collisions Table:**

```SQL
CREATE TABLE "public"."collisions" (
    "id" INT NOT NULL,
    "date" DATE NOT NULL,
    "time" TIME NOT NULL,
    "borough" TEXT,
    "zip_code" TEXT,
    "latitude" DOUBLE PRECISION,
    "longitude" DOUBLE PRECISION,
    "on_street_name" TEXT, 
    "cross_street_name" TEXT,
    "off_street_name" TEXT,
    PRIMARY KEY ("id")
);
```

**Vehicles Table:**

```SQL
CREATE TABLE "public"."vehicles" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "vehicle" TEXT NOT NULL,
    PRIMARY KEY ("id")
);
```

**Vehicle Collisions Table:**

```SQL
CREATE TABLE "public"."vehicle_collisions" (
    "id" INT GENERATED BY DEFAULT AS IDENTITY,
    "collision_id" INT NOT NULL,
    "vehicle_id" INT NOT NULL,
    CONSTRAINT "collision_id_fkey" FOREIGN KEY ("collision_id") REFERENCES "public"."collisions" ("id"),
    CONSTRAINT "vehicle_id_fkey" FOREIGN KEY ("vehicle_id") REFERENCES "public"."vehicles" ("id"),
    PRIMARY KEY ("id")
);
```


#### 4.3 Importing the CSV Data

Navigate to the `Table Editor` page and select **Import data from CSV**. 

Select the appropriate table for each CSV file. Note that you may encounter minor type errors when importing into the `collisions` table, but these should not be critical.


#### 4.4 Querying the Data

**Collisions per Day:**

  ```SQL
  SELECT
      date,
      COUNT(date)
  FROM
      collisions
  GROUP BY
      date
  ORDER BY
      date
  ```

**Collisions per Hourly Block:**

  ```SQL
  SELECT
      COUNT(id),
      time
  FROM
      collisions
  GROUP BY
      time, date_trunc('hour', time)
  ORDER BY
      count DESC
  ```

**Collisions per Day of the Week:**


  ```SQL
  SELECT
      date,
      COUNT(date),
      extract(isodow FROM date) AS "day_of_week"
  FROM
      collisions
  GROUP BY
      date
  ORDER BY
      date
  ```

**Average Daily Collisions per Day of the Week:**

  ```SQL
  SELECT
      AVG(daily_collisions),
      day_of_week AS avg_daily_collisions
  FROM (
      SELECT
          COUNT(date) AS "daily_collisions",
          extract(isodow FROM date) AS "day_of_week"
      FROM
          collisions
      GROUP BY
          date
      ORDER BY
          date) a
  GROUP BY
      day_of_week
  ```

**Collisions Per Vehicle Type:**

  ```SQL
  SELECT
      vc.vehicle_id,
      v.vehicle,
      COUNT

(vc.vehicle_id)
  FROM
      vehicle_collisions vc
      JOIN vehicles v ON v.id = vc.vehicle_id
  GROUP BY
      vc.vehicle_id, v.vehicle
  ORDER BY
      count DESC
  ```

These queries offer insights into the collision patterns by time and vehicle type, aiding in data-driven decision-making for urban safety and planning.

---

### 5.0 Next Steps and Ideas

Since the original data includes lat/long coordinates and Postgres has a powerful GIS module called [PostGIS](https://postgis.net). You can use this module to: 

- Calculate distances between points
- Identify point clusters
- Integrate with other geographic data
- Combine this data with vehicle type distribution to identify collision-prone vehicles
- Pinpoint high-risk roads and intersections
- Suggest improvements (signage, speed limits, etc.) to enhance safety and potentially save lives
