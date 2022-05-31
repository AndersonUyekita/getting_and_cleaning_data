`Quiz 2` Getting and Cleaning Data
================
Anderson H. Uyekita

-   Specialization: [Data Science: Foundations using R
    Specialization](https://www.coursera.org/specializations/data-science-foundations-r)
-   Course: [Getting and Cleaning
    Data](https://www.coursera.org/learn/data-cleaning)
    -   Instructor: Jeffrey Leek
-   Week 2
    -   Start: 2022/05/21
    -   Finish: 2022/05/21

------------------------------------------------------------------------

This Quiz is part of the Getting and Cleaning Data Course.

To turn the folder more structured, I will create a subfolder to host
the dataset.

``` r
# Checking if the subfolder already exists.
if (!dir.exists("data")) {
    
    # Creating a subfolder to store the data.
    dir.create(path = "./data")
}
```

## Question 1

``` r
# 1. Find OAuth settings for github:
#    http://developer.github.com/v3/oauth/
oauth_endpoints("github")
```

    ## <oauth_endpoint>
    ##  authorize: https://github.com/login/oauth/authorize
    ##  access:    https://github.com/login/oauth/access_token

``` r
# 2. My credentials App.
#
# Go to https://github.com/settings/developers and create a "New OAuth App".
myapp <- oauth_app("github",
                   key = "a4828bb1e49fd57761a3",
                   secret = "e3903d74395b7762d0e3d8201354a2b6021a21ba")

# 3. Get OAuth credentials
github_token <- oauth2.0_token(endpoint = oauth_endpoints("github"), app = myapp)

# 4. Requesting the Github API to gather info from Jeff repositories.
request <- GET(url = "https://api.github.com/users/jtleek/repos",
           config = config(token = github_token))

# Checking the GET. If status == 200 means success!
request$status_code
```

    ## [1] 200

``` r
# Converting into R object from a JSON file.
df_req <- content(x = request, as = "parsed")

# Number of Repositories in Jeff Leek Github
length(df_req)
```

    ## [1] 30

``` r
# List of repositories initialization
repos_names <- vector()
data_creation <- vector()

# Loop to gather the repo's names.
for (i in 1:length(df_req)) {
    repos_names <- append(repos_names, df_req[[i]]$name)
    data_creation <- append(data_creation, df_req[[i]]$created_at)
}

# Filtering the datasharing repository and its data creation.
data.frame(cbind(repos_names, data_creation)) %>% filter(repos_names =="datasharing")
```

    ##   repos_names        data_creation
    ## 1 datasharing 2013-11-07T13:25:07Z

## Question 2

``` r
# Downloading the file
download.file(url = "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv",
              destfile = "./data/american_community_survey.csv")

# Loading the CSV file.
acs <- read.csv(file = "./data/american_community_survey.csv")
```

How many rows and columns?

``` r
# Dimensions of this dataset.
dim(acs)     # [ROWS] [COLUMNS]
```

    ## [1] 14931   239

Comparing the options:

``` r
# Solving this problem using tidyverse.
method_tidyverse <- acs %>% filter(AGEP < 50) %>% select(pwgtp1)

# Solving using the SQL
method_sql <- sqldf("select pwgtp1 from acs where AGEP < 50")

# Checking if both are identical.
identical(method_tidyverse, method_sql)
```

    ## [1] TRUE

## Question 3

``` r
# Subsetting the ACS dataset
method_r <- unique(acs$AGEP)

# The unique(acs$AGEP) retuns a vector.
# The sqldf("select distinct AGEP from acs") returns a dataframe. For this reason,
# I have selected the only columns of its dataframe to be comparable.
identical(method_r,
          sqldf("select distinct AGEP from acs")$AGEP)
```

    ## [1] TRUE

## Question 4

``` r
# Open a connection to a website.
#
# ATENTION!! I have to add the "www" to the given URL.
#
soup <- url("https://www.biostat.jhsph.edu/~jleek/contact.html")

# Parse the content of the given CONNECTION in text.
soup_parse <- readLines(con = "https://www.biostat.jhsph.edu/~jleek/contact.html")

# Closing connection.
close(soup)

# Delving the data.
nchar(soup_parse[10])
```

    ## [1] 45

``` r
nchar(soup_parse[20])
```

    ## [1] 31

``` r
nchar(soup_parse[30])
```

    ## [1] 7

``` r
nchar(soup_parse[100])
```

    ## [1] 25

## Question 5

``` r
# Downloading the .FOR file.
download.file(url = "https://d396qusza40orc.cloudfront.net/getdata%2Fwksst8110.for",
              destfile = "./data/data_q5.for")

# Readings: https://www.rdocumentation.org/packages/utils/versions/3.6.2/topics/read.fortran
# I have read this website to understand how the format works.

# Loading the file into R Object
data_q5 <- read.fortran(file = "./data/data_q5.for",
                        format = c("A10", "X5",
                                   "F4.0",
                                   "F4.0", "X5",
                                   "F4.0",
                                   "F4.0", "X5",
                                   "F4.0",
                                   "F4.0", "X5",
                                   "F4.0",
                                   "F4.0", "X5"),
                        skip = 4)

# Sum of the 4th and 9th columns.
sum(data_q5$V4 + data_q5$V9)
```

    ## [1] 32463.2