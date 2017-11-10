<!-- README.md is generated from README.Rmd. Please edit that file -->
redshiftTools
=============

This is an R Package meant to easen the uploading of bulk data into Amazon Redshift.

Installation
------------

To install this package, you'll need to execute these commands:

``` r
    install.packages(c('devtools', 'httr'))
    devtools::install_github("RcppCore/Rcpp")
    devtools::install_github("rstats-db/DBI")
    devtools::install_github("rstats-db/RPostgres")
    install.packages('aws.s3')
    devtools::install_github("sicarul/redshiftTools")
```

Usage
-----

### Creating tables

For creating tables, there is a support function, `rs_create_statement`, which receives a data.frame and returns the query for creating the same table in Amazon Redshift.

``` r
n=1000
testdf = data.frame(
a=rep('a', n),
b=c(1:n),
c=rep(as.Date('2017-01-01'), n),
d=rep(as.POSIXct('2017-01-01 20:01:32'), n),
e=rep(as.POSIXlt('2017-01-01 20:01:32'), n),
f=rep(paste0(rep('a', 4000), collapse=''), n) )

cat(rs_create_statement(testdf, tableName='dm_great_table'))
```

This returns:

``` sql
CREATE TABLE dm_great_table (
a VARCHAR(8),
b int,
c date,
d timestamp,
e timestamp,
f VARCHAR(4096)
);
```

The cat is only done to view properly in console, it's not done directly in the function in case you need to pass the string to another function (Like a query runner)

### Uploading data

For uploading data, you'll have available now 2 functions: `rs_replace_table` and `rs_upsert_table`, both of these functions are called with almost the same parameters, except on upsert you can specify with which keys to search for matching rows.

For example, suppose we have a table to load with 2 integer columns, we could use the following code:

``` r
    library("aws.s3")
    library(RPostgres)
    library(redshiftTools)

    a=data.frame(a=seq(1,10000), b=seq(10000,1))
    n=head(a,n=10)
    n$b=n$a
    nx=rbind(n, data.frame(a=seq(5:10), b=seq(10:5)))

    con <- dbConnect(RPostgres::Postgres(), dbname="dbname",
    host='my-redshift-url.amazon.com', port='5439',
    user='myuser', password='mypassword',sslmode='require')

    b=rs_replace_table(a, dbcon=con, tableName='mytable', bucket="mybucket", split_files=4)
    c=rs_upsert_table(nx, dbcon=con, tableName = 'mytable', split_files=4, bucket="mybucket", keys=c('a'))
```
