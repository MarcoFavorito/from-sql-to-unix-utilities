# From SQL to Unix Utilities
A guide about how to do SQL queries with Unix utilities.

Some of the UNIX utilities that we will use are: `cat`, `cut`, `sed`, `grep`, `awk`, `sort`, `uniq`, `comm`, `join`.

We try to focus on readability and memorizability, rather than performance or optimality.

We make the following assumptions:
- The relations (aka tables) are stored in different, well-formatted files.
- The separator for each field is '\t', unless otherwise specified.
- The first row contains the attributes name.

## Quick reference

[SELECT * FROM  RELATION](#SELECT-FROM-RELATION)
[SELECT Attribute FROM Relation](#SELECT-Attribute-FROM-Relation)
[SELECT Attribute1, Attribute2, ... FROM Relation](#SELECT-Attribute1-Attribute2--FROM-Relation)
[SELECT DISTINCT Attribute FROM Relation;](#SELECT-DISTINCT-Attribute-FROM-Relation)
[SELECT COUNT(DISTINCT Attribute) FROM Relation;](#SELECT-COUNTDISTINCT-Attribute-FROM-Relation;)
[SELECT * FROM Relation WHERE Attribute=value;](#SELECT-FROM-Relation-WHERE-Attributevalue;)
[SELECT * FROM Relation ORDER BY Attribute (ASC|DESC);](select--from-relation-order-by-attribute-ascdesc)

## Relations examples

Here we provide the definitions of the relations used in this guide:

- Customers(CustomerID, CustomerName, ContactName, Address City, PostalCode, Country)


## Rosetta stone


### SELECT * FROM  RELATION

     SELECT * FROM Customers;

Just look at all the content of the relation:

    cat customers.tsv

    CustomerID	CustomerName	ContactName	Address	City	PostalCode	Country
    1	Alfreds Futterkiste	Maria Anders	Obere Str. 57	Berlin	12209	Germany
    2	Ana Trujillo Emparedados y helados	Ana Trujillo	Avda. de la Constitución 2222	México D.F.	05021	Mexico
    3	Antonio Moreno Taquería	Antonio Moreno	Mataderos 2312	México D.F.	05023	Mexico
    4	Around the Horn	Thomas Hardy	120 Hanover Sq.	London	WA1 1DP	UK
    5	Berglunds snabbköp	Christina Berglund	Berguvsvägen 8	Luleå	S-958 22	Sweden


### SELECT Attribute FROM Relation

    SELECT CustomerName FROM Customers

- If we know the column number (in this case, 1):

        cut -f1 customer.tsv

        CustomerName
        Alfreds Futterkiste
        Ana Trujillo Emparedados y helados
        Antonio Moreno Taquería
        Around the Horn
        Berglunds snabbköp

- The same output with `awk`:

        awk -F'\t' '{print $2}' customers.tsv

- If you strictly need to make queries by column name, you can use:

        awk -F'\t' -vcol=CustomerName '(NR==1){colnum=-1;for(i=1;i<=NF;i++)if($(i)==col)colnum=i;}{print $(colnum)}' customers.tsv

Notice: you can explicitly specify a character delimiter by using the options:
- `cut -d '$separator'`
- `awk -F '$separator'`

### SELECT Attribute1, Attribute2, ... FROM Relation

    SELECT CustomerID, CustomerName, ContactName  FROM Customers

- As in the previous case, knowing the column numbers:

        cut -f1,2,3 customers.tsv

        CustomerID	CustomerName	ContactName
        1	Alfreds Futterkiste	Maria Anders
        2	Ana Trujillo Emparedados y helados	Ana Trujillo
        3	Antonio Moreno Taquería	Antonio Moreno
        4	Around the Horn	Thomas Hardy
        5	Berglunds snabbköp	Christina Berglund


or, equivalently:

    cut -f1-3 customers.tsv

- Using `awk`:

        awk -F'\t' '{print $1FS$2FS$3}' customers.tsv

- By using the column names:

        awk -F'\t' -vcols=CustomerID,CustomerName,ContactName '(NR==1){n=split(cols,cs,",");for(c=1;c<=n;c++){for(i=1;i<=NF;i++)if($(i)==cs[c])ci[c]=i}}{for(i=1;i<=n;i++)printf "%s" FS,$(ci[i]);printf "\n"}' customers.tsv


### SELECT DISTINCT Attribute FROM Relation;

     SELECT DISTINCT Country FROM Customers;

For this kind of queries (especially the ones that we implement using sorting) we need to ignore the header row and process it later.

- Using projection with `cut` and removing repetition with `sort -u`:

        tail -n +2 customers.tsv | cut -f 5  | sort -u
        Berlin
        London
        Luleå
        México D.F.

All at once (i.e. with the header):

    (head -n 1 customers.tsv | cut -f 5) && (tail -n +2 customers.tsv | cut -f 5 | sort -u)
    City
    Berlin
    London
    Luleå
    México D.F.


- Using `awk`:

        awk -F'\t' '(NR==1){print $5}(NR!=1){ a[$5]++ } END { print length(a) }' customers.tsv

### SELECT COUNT(DISTINCT Attribute) FROM Relation;

     SELECT COUNT(DISTINCT Country) FROM Customers;

- With `cut`, kust as the previous query (without header), but pipelining with `wc -l`:

        tail -n +2 customers.tsv | cut -f 5  | sort -u | wc -l
        4

- Using `awk`:

        awk -F'\t' '(NR==1){print "Count("$5")"}(NR!=1){ a[$5]++ } END { print length(a) }' customers.tsv

### SELECT * FROM Relation WHERE Attribute=value;

    SELECT * FROM Customers WHERE City='Berlin';

- Using `awk`:

        awk -F'\t' '$5 == "Berlin" { print }' customers.tsv
        1	Alfreds Futterkiste	Maria Anders	Obere Str. 57	Berlin	12209	Germany

### SELECT * FROM Relation ORDER BY Attribute (ASC|DESC);

    SELECT * FROM Customers ORDER BY Country;

- Using `sort`:

        tail -n +2 customers.tsv  | sort -t$'\t' -k 7

As usual, you can prefix `head -1 customers.tsv && ...` to add the header.

You can reverse by adding `r`:

    tail -n +2 customers.tsv  | sort -t$'\t' -k 7r

And you can add attributes by simply add another `-k` flag:

    tail -n +2 customers.tsv  | sort -t$'\t' -k 7 -k 1nr

The `n` stands for "numerical sorting".


## References

- [Relational Shell Programming](http://matt.might.net/articles/sql-in-the-shell/)
- [Implementing SQL With Unix Utilities](https://www.xaprb.com/blog/2012/10/12/implementing-sql-with-unix-utilities/)
- [The AWK programming language](https://ia802309.us.archive.org/25/items/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf)
- [SQL Tutorial](https://www.w3schools.com/sql/)
