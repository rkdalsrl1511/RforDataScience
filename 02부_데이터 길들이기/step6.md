9장 : tidyr로 하는 타이디 데이터
================
huimin
2019년 5월 2일

기초 설정하기
=============

``` r
library(tidyverse)
```

    ## Registered S3 methods overwritten by 'ggplot2':
    ##   method         from 
    ##   [.quosures     rlang
    ##   c.quosures     rlang
    ##   print.quosures rlang

    ## Registered S3 method overwritten by 'rvest':
    ##   method            from
    ##   read_xml.response xml2

    ## -- Attaching packages --------------------- tidyverse 1.2.1 --

    ## √ ggplot2 3.1.1       √ purrr   0.3.2  
    ## √ tibble  2.1.1       √ dplyr   0.8.0.1
    ## √ tidyr   0.8.3       √ stringr 1.4.0  
    ## √ readr   1.3.1       √ forcats 0.4.0

    ## -- Conflicts ------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readr)
```

들어가기
========

지저분한 데이터를 tidyr 패키지를 통해서 정리할 수 있다.

타이디 데이터
=============

데이터를 타이디하게 만드는, 서로 연관된 세 가지 규칙은 다음과 같다.

-   변수마다 해당되는 열이 있어야 한다.
-   관측값마다 해당되는 행이 있어야 한다.
-   값마다 해당하는 하나의 셀이 있어야 한다.

이 규칙에 따르기 위해서는,

1.  데이터셋을 티블에 각각 넣어라.
2.  변수를 열에 각각 넣어라.

데이터가 타이디할 경우 장점은 두 가지이다.

-   데이터를 일관된 방식으로 저장하면 보편적인 장점이 있다. 일관된 데이터 구조를 사용하면 이에 적용할 도구들이 공통성을 가지게 되어, 이들을 배우기가 더 쉬워진다.
-   변수를 열에 배치하면 R의 벡터화 속성이 가장 잘 발휘된다는 점에서 구체적인 장점이 생긴다.

**내가 너무 R에 익숙한 나머지, 기본이라고 생각했던 부분이 사실은 타이디 데이터였다.**

연습문제(145p)
--------------

``` r
# table2와 table4a + table4b에서 비율(rate)을 계산하라.
# a. 연도별, 국가별로 결핵 사례 수(case)를 추출하라.
# b. 연도별, 국가별로 해당하는 인구를 추출하라.
# c. 사례 수를 인구로 나누고 10,000을 곱하라.
# d. 적절한 곳에 다시 저장하라.

head(table2, n=10)
```

    ## # A tibble: 10 x 4
    ##    country      year type            count
    ##    <chr>       <int> <chr>           <int>
    ##  1 Afghanistan  1999 cases             745
    ##  2 Afghanistan  1999 population   19987071
    ##  3 Afghanistan  2000 cases            2666
    ##  4 Afghanistan  2000 population   20595360
    ##  5 Brazil       1999 cases           37737
    ##  6 Brazil       1999 population  172006362
    ##  7 Brazil       2000 cases           80488
    ##  8 Brazil       2000 population  174504898
    ##  9 China        1999 cases          212258
    ## 10 China        1999 population 1272915272

``` r
head(table4a, n=10)
```

    ## # A tibble: 3 x 3
    ##   country     `1999` `2000`
    ##   <chr>        <int>  <int>
    ## 1 Afghanistan    745   2666
    ## 2 Brazil       37737  80488
    ## 3 China       212258 213766

``` r
head(table4b, n=10)
```

    ## # A tibble: 3 x 3
    ##   country         `1999`     `2000`
    ##   <chr>            <int>      <int>
    ## 1 Afghanistan   19987071   20595360
    ## 2 Brazil       172006362  174504898
    ## 3 China       1272915272 1280428583

``` r
# 방법 1 : table4a + table4b를 통해서 비율 계산하기
new.tibble <- tibble(country = table4a$country,
                   `1999rate` = table4a$`1999`/table4b$`1999` * 10000,
                   `2000rate` = table4a$`2000`/table4b$`2000` * 10000)

new.tibble
```

    ## # A tibble: 3 x 3
    ##   country     `1999rate` `2000rate`
    ##   <chr>            <dbl>      <dbl>
    ## 1 Afghanistan      0.373       1.29
    ## 2 Brazil           2.19        4.61
    ## 3 China            1.67        1.67

``` r
# 방법 2 : table2만 사용하여 계산하기
# 1단계 : case랑 population을 변수로 바꿔야한다.
# 그렇게 하기 위해서는 그냥 차라리 새로 만드는 편이 낫다.
```

Gather와 Spread
===============

대부분의 사람들은 타이디 데이터의 원리에 익숙하지 않으며, 데이터 작업에 많은 시간을 써야만 타이디 데이터로 만들 수 있다. 또한, 데이터는 분석보다도 다른 용도로 사용하려고 구성하는 경우도 많다. 그렇기 때문에 타이디한 데이터가 아니다.

**따라서 대부분의 실제 분석에서는 타이디하게 만드는 작업이 필요하다.**

**첫 번째 단계는 항상 변수와 관측값이 무엇인지 파악하는 것이다.**

**두 번째 단계는 자주 일어나는 두 가지 문제 중 하나를 해결하는 것이다.**

-   하나의 변수가 여러 열에 분산되어 있을 수 있다.
-   하나의 관측값이 여러 행에 흩어져 있을 수 있다.

이러한 문제들을 해결하려면 tidyr에서 **gather()와 spread()함수**가 필요하다.

먼저 gather와 spread에 대하여 말하자면, **gather는 기존의 key를 value로 바꾸고 기존의 key에 있던 value에 새로운 이름을 붙여서 추가**하는 것이고, **spread는 기존의 value를 새로운 key로 바꾸고, 하나의 key를 선택하여 그 key의 value 값들을 새로운 key에 편입시키는 것이다.**

gather()로 모으기
-----------------

자주 생기는 문제는 **데이터셋의 일부 열 이름이 변수 이름이 아니라 변수 값인 경우**이다. 이와 같은 데이터셋을 타이디하게 만들려면 해당 열을 새로운 두 변수로 모아야 한다. 이 작업을 설명하기 위해 세 가지 파라미터가 필요하다.

-   변수가 아니라 값을 나타내는 열 집합. 예에서는 열 1999와 열 2000이다.
-   key : 열 이름 자리링리에 나타난 값의 변수 이름
-   value : 셀에 값이 분산되어 있는 변수의 이름

``` r
table4a
```

    ## # A tibble: 3 x 3
    ##   country     `1999` `2000`
    ## * <chr>        <int>  <int>
    ## 1 Afghanistan    745   2666
    ## 2 Brazil       37737  80488
    ## 3 China       212258 213766

``` r
# 변수가 아니라 값을 나타내는 열 집합. 열 1999와 열 2000이다.
# (key)열 이름 자리에 나타난 값을 변수 이름은 year이다.
# (value)셀에 값이 분산되어 있는 변수의 이름은 cases이다.

tidy4a <- table4a %>% 
  gather(`1999`, `2000`, key = "year", value = "cases")

tidy4b <- table4b %>% 
  gather(`1999`, `2000`, key = "year", value = "population")


left_join(tidy4a, tidy4b)
```

    ## Joining, by = c("country", "year")

    ## # A tibble: 6 x 4
    ##   country     year   cases population
    ##   <chr>       <chr>  <int>      <int>
    ## 1 Afghanistan 1999     745   19987071
    ## 2 Brazil      1999   37737  172006362
    ## 3 China       1999  212258 1272915272
    ## 4 Afghanistan 2000    2666   20595360
    ## 5 Brazil      2000   80488  174504898
    ## 6 China       2000  213766 1280428583

spread()로 펼치기
-----------------

펼치기는 수집하기의 반대이다. 관측값이 여러 행에 흩어져 있을 때 사용한다. table2가 그 예시이다. 2개의 파라미터가 필요하다.

-   변수 이름을 포함하는 열, 즉 key 열. 여기에서는 type이다.
-   여러 변수를 형성하는 값을 포함하는 열, 즉 value 열. 여기는 count이다.

``` r
table2
```

    ## # A tibble: 12 x 4
    ##    country      year type            count
    ##    <chr>       <int> <chr>           <int>
    ##  1 Afghanistan  1999 cases             745
    ##  2 Afghanistan  1999 population   19987071
    ##  3 Afghanistan  2000 cases            2666
    ##  4 Afghanistan  2000 population   20595360
    ##  5 Brazil       1999 cases           37737
    ##  6 Brazil       1999 population  172006362
    ##  7 Brazil       2000 cases           80488
    ##  8 Brazil       2000 population  174504898
    ##  9 China        1999 cases          212258
    ## 10 China        1999 population 1272915272
    ## 11 China        2000 cases          213766
    ## 12 China        2000 population 1280428583

``` r
table2 %>% 
  spread(key = "type", value = "count")
```

    ## # A tibble: 6 x 4
    ##   country      year  cases population
    ##   <chr>       <int>  <int>      <int>
    ## 1 Afghanistan  1999    745   19987071
    ## 2 Afghanistan  2000   2666   20595360
    ## 3 Brazil       1999  37737  172006362
    ## 4 Brazil       2000  80488  174504898
    ## 5 China        1999 212258 1272915272
    ## 6 China        2000 213766 1280428583

연습문제
--------

``` r
# 1. gather과 spread가 완벽하게 대칭이 아닌 이유는?
stocks <- tibble(year = c(2015,2015,2016,2016),
                 half = c(1,2,1,2),
                 return = c(1.88, 0.59, 0.92, 0.17))

stocks
```

    ## # A tibble: 4 x 3
    ##    year  half return
    ##   <dbl> <dbl>  <dbl>
    ## 1  2015     1   1.88
    ## 2  2015     2   0.59
    ## 3  2016     1   0.92
    ## 4  2016     2   0.17

``` r
stocks %>% 
  spread(key ="year", value = "return") %>% 
  gather(`2015`:`2016`, key = "year",value = "return")
```

    ## # A tibble: 4 x 3
    ##    half year  return
    ##   <dbl> <chr>  <dbl>
    ## 1     1 2015    1.88
    ## 2     2 2015    0.59
    ## 3     1 2016    0.92
    ## 4     2 2016    0.17

``` r
# gather와 spread에 있는 convert 인수는 열이 문자열로 강제 변환된 경우 유용하다.


# 2. 다음 데이터를 타이디하게 하라.
preg <- tribble(~pregnant, ~male, ~female,
                "yes",NA,10,
                "no",20,12)

preg %>% 
  gather(male, female, key = "sex", value = "month") %>% 
  select(sex, pregnant, month)
```

    ## # A tibble: 4 x 3
    ##   sex    pregnant month
    ##   <chr>  <chr>    <dbl>
    ## 1 male   yes         NA
    ## 2 male   no          20
    ## 3 female yes         10
    ## 4 female no          12

Separate와 Unite
================

table3에는 지금까지 볼 수 없었던 다른 문제가 있다. **두 개의 변수(cases 및 population)가 포함된 한 개의 열(rate)이 있다.** 이 문제를 해결하려면 separate() 함수가 필요하다. 또한 하나의 변수가 여러 열에 분산되어 있는 경우에 사용하는, separate()의 보완함수인 unite()에 대해서도 학습한다.

separate()로 분리하기
---------------------

separate()는 구분 문자가 나타나는 곳마다 쪼개서 하나의 열을 여러 열로 분리한다.

``` r
table3
```

    ## # A tibble: 6 x 3
    ##   country      year rate             
    ## * <chr>       <int> <chr>            
    ## 1 Afghanistan  1999 745/19987071     
    ## 2 Afghanistan  2000 2666/20595360    
    ## 3 Brazil       1999 37737/172006362  
    ## 4 Brazil       2000 80488/174504898  
    ## 5 China        1999 212258/1272915272
    ## 6 China        2000 213766/1280428583

``` r
#참고 : sep은 정규표현식이다.
table3 %>% 
  separate(rate, into = c("cases","population"), sep = "/")
```

    ## # A tibble: 6 x 4
    ##   country      year cases  population
    ##   <chr>       <int> <chr>  <chr>     
    ## 1 Afghanistan  1999 745    19987071  
    ## 2 Afghanistan  2000 2666   20595360  
    ## 3 Brazil       1999 37737  172006362 
    ## 4 Brazil       2000 80488  174504898 
    ## 5 China        1999 212258 1272915272
    ## 6 China        2000 213766 1280428583

``` r
# convert옵션을 설정할 경우, 알아서 형 변환을 시도한다.
table3 %>% 
  separate(rate, 
           into = c("cases","population"), 
           sep = "/", 
           convert = TRUE)
```

    ## # A tibble: 6 x 4
    ##   country      year  cases population
    ##   <chr>       <int>  <int>      <int>
    ## 1 Afghanistan  1999    745   19987071
    ## 2 Afghanistan  2000   2666   20595360
    ## 3 Brazil       1999  37737  172006362
    ## 4 Brazil       2000  80488  174504898
    ## 5 China        1999 212258 1272915272
    ## 6 China        2000 213766 1280428583

``` r
# sep에 정수를 전달할 경우, 해당 정수의 인덱스 번호를 기준으로 쪼갠다.

table3 %>% 
  separate(year,
           into = c("century","year"),
           sep = 2,
           convert = FALSE)
```

    ## # A tibble: 6 x 4
    ##   country     century year  rate             
    ##   <chr>       <chr>   <chr> <chr>            
    ## 1 Afghanistan 19      99    745/19987071     
    ## 2 Afghanistan 20      00    2666/20595360    
    ## 3 Brazil      19      99    37737/172006362  
    ## 4 Brazil      20      00    80488/174504898  
    ## 5 China       19      99    212258/1272915272
    ## 6 China       20      00    213766/1280428583

unite()로 결합하기
------------------

unite()는 separate()의 반대이다. 여러 열을 하나의 열로 결합한다.

``` r
table5 %>% 
  unite(new, century, year, sep = "")
```

    ## # A tibble: 6 x 3
    ##   country     new   rate             
    ##   <chr>       <chr> <chr>            
    ## 1 Afghanistan 1999  745/19987071     
    ## 2 Afghanistan 2000  2666/20595360    
    ## 3 Brazil      1999  37737/172006362  
    ## 4 Brazil      2000  80488/174504898  
    ## 5 China       1999  212258/1272915272
    ## 6 China       2000  213766/1280428583

연습문제(153p)
--------------

``` r
# 1. separate()의 extra인수와 fill인수의 역할은 무엇인가?

# extra의 경우, 넘치는 경우 warn, drop, merge를 선택하여 처리한다.
tibble(x = c("a,b,c","d,e,f,g","h,i,j")) %>% 
  separate(col = x, into = c("one","two","three"), extra = "drop")
```

    ## # A tibble: 3 x 3
    ##   one   two   three
    ##   <chr> <chr> <chr>
    ## 1 a     b     c    
    ## 2 d     e     f    
    ## 3 h     i     j

``` r
tibble(x = c("a,b,c","d,e,f,g","h,i,j")) %>% 
  separate(col = x, into = c("one","two","three"), extra = "merge")
```

    ## # A tibble: 3 x 3
    ##   one   two   three
    ##   <chr> <chr> <chr>
    ## 1 a     b     c    
    ## 2 d     e     f,g  
    ## 3 h     i     j

``` r
# fill의 경우, 부족한 경우에 warn, right, left를 선택하여 처리한다.
tibble(x = c("a,b,c","d,e","h,i,j")) %>% 
  separate(col = x, into = c("one","two","three"), fill = "right")
```

    ## # A tibble: 3 x 3
    ##   one   two   three
    ##   <chr> <chr> <chr>
    ## 1 a     b     c    
    ## 2 d     e     <NA> 
    ## 3 h     i     j

``` r
tibble(x = c("a,b,c","d,e","h,i,j")) %>% 
  separate(col = x, into = c("one","two","three"), fill = "left")
```

    ## # A tibble: 3 x 3
    ##   one   two   three
    ##   <chr> <chr> <chr>
    ## 1 a     b     c    
    ## 2 <NA>  d     e    
    ## 3 h     i     j

결측값
======

데이터셋의 표현 방식을 변경하면 결측값에 중요하고도 미묘한 이슈가 나타난다. 두 가지 방식으로 결측될 수 있다.

-   명시적으로, 즉 NA로 표시된다.
-   암묵적으로, 즉 단순히 데이터에 존재하지 않는다.

``` r
stock <- tibble(year = c(2015,2015,2015,2015,2016,2016,2016),
                qtr = c(1,2,3,4,2,3,4),
                return = c(1.88,0.59,0.35,NA,0.92,0.17,2.66))

stock
```

    ## # A tibble: 7 x 3
    ##    year   qtr return
    ##   <dbl> <dbl>  <dbl>
    ## 1  2015     1   1.88
    ## 2  2015     2   0.59
    ## 3  2015     3   0.35
    ## 4  2015     4  NA   
    ## 5  2016     2   0.92
    ## 6  2016     3   0.17
    ## 7  2016     4   2.66

위의 데이터 프레임을 본다면, 2015년 4분기의 값은 NA로서, 명시적으로 표현되어 있다. 하지만 2016년 1분기는 데이터 자체가 없다. 이를 암묵적인 결측값이라고 한다.

**다음과 같이 암묵적 값을 명시적으로 만들 수도 있다.**

``` r
stock %>% 
  spread(key = "year", value = "return") %>% 
  gather(col = `2015`:`2016`, key = "year", value = "return") %>% 
  select(year, qtr, return)
```

    ## # A tibble: 8 x 3
    ##   year    qtr return
    ##   <chr> <dbl>  <dbl>
    ## 1 2015      1   1.88
    ## 2 2015      2   0.59
    ## 3 2015      3   0.35
    ## 4 2015      4  NA   
    ## 5 2016      1  NA   
    ## 6 2016      2   0.92
    ## 7 2016      3   0.17
    ## 8 2016      4   2.66

**또한, gather에서 na.rm옵션을 설정하면 명시적 값을 암묵적으로 전환한다.**

``` r
stock %>% 
  spread(key = "year", value = "return") %>% 
  gather(col = `2015`:`2016`, 
         key = "year", 
         value = "return",
         na.rm = TRUE) %>% 
  select(year, qtr, return)
```

    ## # A tibble: 6 x 3
    ##   year    qtr return
    ##   <chr> <dbl>  <dbl>
    ## 1 2015      1   1.88
    ## 2 2015      2   0.59
    ## 3 2015      3   0.35
    ## 4 2016      2   0.92
    ## 5 2016      3   0.17
    ## 6 2016      4   2.66

타이디 데이터에서 결측값을 명시적으로 표현하는 중요한 도구로 **complete()**도 있다. complete에 열 집합을 입력하면, **고유한 조합을 모두 찾아낸다.** 그런 다음 원본 데이터셋에 필요한 곳에 명시적으로 NA를 채워 넣는다.

``` r
stock %>%
  complete(year, qtr)
```

    ## # A tibble: 8 x 3
    ##    year   qtr return
    ##   <dbl> <dbl>  <dbl>
    ## 1  2015     1   1.88
    ## 2  2015     2   0.59
    ## 3  2015     3   0.35
    ## 4  2015     4  NA   
    ## 5  2016     1  NA   
    ## 6  2016     2   0.92
    ## 7  2016     3   0.17
    ## 8  2016     4   2.66

결측값 작업할 때 알아야 할 중요한 도구가 하나 더 있다. 데이터 소스가 주로 데이터 입력에 사용된 경우 결측값은 이전 값이 전달되어야 함을 나타낸다.

``` r
treatment <- tribble(~person, ~treatment, ~response,
                     "Derrick Whitmore", 1, 7,
                     NA, 2, 10,
                     NA, 3, 9,
                     "Katherine Burke", 1, 4)
```

이러한 결측값은 **fill()**을 통하여 채울 수 있다. **이 함수는 결측값을 가장 최근의 비결측값으로 치환하고자 하는 열(집합)을 입력으로 한다.** 이를 전문 용어로는 **마지막 관측값 이월** 이라고 한다.

``` r
treatment %>% 
  fill(person)
```

    ## # A tibble: 4 x 3
    ##   person           treatment response
    ##   <chr>                <dbl>    <dbl>
    ## 1 Derrick Whitmore         1        7
    ## 2 Derrick Whitmore         2       10
    ## 3 Derrick Whitmore         3        9
    ## 4 Katherine Burke          1        4

사례연구
========

데이터를 타이디하게 만들기 위해 배운 것을 모두 정리하면서 이 장을 끝내자. **tidyr::who 데이터셋**에는 결핵 사례가 연도, 국가, 나이, 성별 및 진단 방법별로 세분화되어 있다.

이 데이터셋을 타이디하게 만들어서 작업을 하려고 한다.

``` r
who
```

    ## # A tibble: 7,240 x 60
    ##    country iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534
    ##    <chr>   <chr> <chr> <int>       <int>        <int>        <int>
    ##  1 Afghan~ AF    AFG    1980          NA           NA           NA
    ##  2 Afghan~ AF    AFG    1981          NA           NA           NA
    ##  3 Afghan~ AF    AFG    1982          NA           NA           NA
    ##  4 Afghan~ AF    AFG    1983          NA           NA           NA
    ##  5 Afghan~ AF    AFG    1984          NA           NA           NA
    ##  6 Afghan~ AF    AFG    1985          NA           NA           NA
    ##  7 Afghan~ AF    AFG    1986          NA           NA           NA
    ##  8 Afghan~ AF    AFG    1987          NA           NA           NA
    ##  9 Afghan~ AF    AFG    1988          NA           NA           NA
    ## 10 Afghan~ AF    AFG    1989          NA           NA           NA
    ## # ... with 7,230 more rows, and 53 more variables: new_sp_m3544 <int>,
    ## #   new_sp_m4554 <int>, new_sp_m5564 <int>, new_sp_m65 <int>,
    ## #   new_sp_f014 <int>, new_sp_f1524 <int>, new_sp_f2534 <int>,
    ## #   new_sp_f3544 <int>, new_sp_f4554 <int>, new_sp_f5564 <int>,
    ## #   new_sp_f65 <int>, new_sn_m014 <int>, new_sn_m1524 <int>,
    ## #   new_sn_m2534 <int>, new_sn_m3544 <int>, new_sn_m4554 <int>,
    ## #   new_sn_m5564 <int>, new_sn_m65 <int>, new_sn_f014 <int>,
    ## #   new_sn_f1524 <int>, new_sn_f2534 <int>, new_sn_f3544 <int>,
    ## #   new_sn_f4554 <int>, new_sn_f5564 <int>, new_sn_f65 <int>,
    ## #   new_ep_m014 <int>, new_ep_m1524 <int>, new_ep_m2534 <int>,
    ## #   new_ep_m3544 <int>, new_ep_m4554 <int>, new_ep_m5564 <int>,
    ## #   new_ep_m65 <int>, new_ep_f014 <int>, new_ep_f1524 <int>,
    ## #   new_ep_f2534 <int>, new_ep_f3544 <int>, new_ep_f4554 <int>,
    ## #   new_ep_f5564 <int>, new_ep_f65 <int>, newrel_m014 <int>,
    ## #   newrel_m1524 <int>, newrel_m2534 <int>, newrel_m3544 <int>,
    ## #   newrel_m4554 <int>, newrel_m5564 <int>, newrel_m65 <int>,
    ## #   newrel_f014 <int>, newrel_f1524 <int>, newrel_f2534 <int>,
    ## #   newrel_f3544 <int>, newrel_f4554 <int>, newrel_f5564 <int>,
    ## #   newrel_f65 <int>

이 데이터셋은 전형적인 실 데이터이다. **열 중복, 가변 코드, 다수의 결측값**들이 포함되어 있다.

-   country, iso2와 iso3는 국가를 중복해서 지정하는 세 개의 변수이다.
-   year 또한 분명히 변수이다.
-   다른 모든 열을 알 수 없지만, 변수가 아니라 value같다.

``` r
# NA값 확인해보기
sapply(who, FUN = function(x) sum(is.na(x)))
```

    ##      country         iso2         iso3         year  new_sp_m014 
    ##            0            0            0            0         4067 
    ## new_sp_m1524 new_sp_m2534 new_sp_m3544 new_sp_m4554 new_sp_m5564 
    ##         4031         4034         4021         4017         4022 
    ##   new_sp_m65  new_sp_f014 new_sp_f1524 new_sp_f2534 new_sp_f3544 
    ##         4031         4066         4046         4040         4041 
    ## new_sp_f4554 new_sp_f5564   new_sp_f65  new_sn_m014 new_sn_m1524 
    ##         4036         4045         4043         6195         6210 
    ## new_sn_m2534 new_sn_m3544 new_sn_m4554 new_sn_m5564   new_sn_m65 
    ##         6218         6215         6213         6219         6220 
    ##  new_sn_f014 new_sn_f1524 new_sn_f2534 new_sn_f3544 new_sn_f4554 
    ##         6200         6218         6224         6220         6222 
    ## new_sn_f5564   new_sn_f65  new_ep_m014 new_ep_m1524 new_ep_m2534 
    ##         6223         6221         6202         6214         6220 
    ## new_ep_m3544 new_ep_m4554 new_ep_m5564   new_ep_m65  new_ep_f014 
    ##         6216         6220         6225         6222         6208 
    ## new_ep_f1524 new_ep_f2534 new_ep_f3544 new_ep_f4554 new_ep_f5564 
    ##         6219         6219         6219         6223         6223 
    ##   new_ep_f65  newrel_m014 newrel_m1524 newrel_m2534 newrel_m3544 
    ##         6226         7050         7058         7057         7056 
    ## newrel_m4554 newrel_m5564   newrel_m65  newrel_f014 newrel_f1524 
    ##         7056         7055         7058         7050         7056 
    ## newrel_f2534 newrel_f3544 newrel_f4554 newrel_f5564   newrel_f65 
    ##         7058         7057         7057         7057         7055

``` r
ncol(who)
```

    ## [1] 60

``` r
# gather를 통해서 열 모으기
who1 <- who %>% 
  gather(new_sp_m014:newrel_f65, key = "key", value = "cases")

# 잠시 범주형으로 전환
who1$key <- as.factor(who1$key)
# 레벨 파악하기
nlevels(who1$key)
```

    ## [1] 56

``` r
levels(who1$key)
```

    ##  [1] "new_ep_f014"  "new_ep_f1524" "new_ep_f2534" "new_ep_f3544"
    ##  [5] "new_ep_f4554" "new_ep_f5564" "new_ep_f65"   "new_ep_m014" 
    ##  [9] "new_ep_m1524" "new_ep_m2534" "new_ep_m3544" "new_ep_m4554"
    ## [13] "new_ep_m5564" "new_ep_m65"   "new_sn_f014"  "new_sn_f1524"
    ## [17] "new_sn_f2534" "new_sn_f3544" "new_sn_f4554" "new_sn_f5564"
    ## [21] "new_sn_f65"   "new_sn_m014"  "new_sn_m1524" "new_sn_m2534"
    ## [25] "new_sn_m3544" "new_sn_m4554" "new_sn_m5564" "new_sn_m65"  
    ## [29] "new_sp_f014"  "new_sp_f1524" "new_sp_f2534" "new_sp_f3544"
    ## [33] "new_sp_f4554" "new_sp_f5564" "new_sp_f65"   "new_sp_m014" 
    ## [37] "new_sp_m1524" "new_sp_m2534" "new_sp_m3544" "new_sp_m4554"
    ## [41] "new_sp_m5564" "new_sp_m65"   "newrel_f014"  "newrel_f1524"
    ## [45] "newrel_f2534" "newrel_f3544" "newrel_f4554" "newrel_f5564"
    ## [49] "newrel_f65"   "newrel_m014"  "newrel_m1524" "newrel_m2534"
    ## [53] "newrel_m3544" "newrel_m4554" "newrel_m5564" "newrel_m65"

데이터 사전을 사용할 수도 있다. 데이터 사전은 다음을 알려준다.

(1). 각 열의 처음 세 글자는 해당 열이 포함하는 결핵 사례가, 새로운 사례인지 과거 사례인지를 나타낸다. 이 데이터셋에서 각 열은 새로운 사례를 포함한다.

(2). 그 다음 두 글자는 다음의 결핵의 유형을 기술한다.

-   rel은 재발 사례를 의미한다.
-   ep는 폐외(extrapulmonary) 결핵 사례를 의미한다.
-   sn은 폐 얼룩으로 보이지 않는 폐결핵 사례를 의미한다.(smear negative)
-   sp는 폐 얼룩으로 보이는 폐결핵 사례를 의미한다.

(3). 여섯 번째 글자는 결핵 환자의 성별을 나타낸다. 남성(m)과 여성(f)으로 사례를 분류한다.

(4). 나머지 숫자는 연령대를 나타낸다.

**우선 열 이름의 형식을 수정해야 한다.** 열 이름이 new\_rel이 아니라 newrel이기 때문에 일관성이 없다.

``` r
who2 <- who1 %>% 
  mutate(key = str_replace(key, "newrel", "new_rel"))

table(who2$key)
```

    ## 
    ##   new_ep_f014  new_ep_f1524  new_ep_f2534  new_ep_f3544  new_ep_f4554 
    ##          7240          7240          7240          7240          7240 
    ##  new_ep_f5564    new_ep_f65   new_ep_m014  new_ep_m1524  new_ep_m2534 
    ##          7240          7240          7240          7240          7240 
    ##  new_ep_m3544  new_ep_m4554  new_ep_m5564    new_ep_m65  new_rel_f014 
    ##          7240          7240          7240          7240          7240 
    ## new_rel_f1524 new_rel_f2534 new_rel_f3544 new_rel_f4554 new_rel_f5564 
    ##          7240          7240          7240          7240          7240 
    ##   new_rel_f65  new_rel_m014 new_rel_m1524 new_rel_m2534 new_rel_m3544 
    ##          7240          7240          7240          7240          7240 
    ## new_rel_m4554 new_rel_m5564   new_rel_m65   new_sn_f014  new_sn_f1524 
    ##          7240          7240          7240          7240          7240 
    ##  new_sn_f2534  new_sn_f3544  new_sn_f4554  new_sn_f5564    new_sn_f65 
    ##          7240          7240          7240          7240          7240 
    ##   new_sn_m014  new_sn_m1524  new_sn_m2534  new_sn_m3544  new_sn_m4554 
    ##          7240          7240          7240          7240          7240 
    ##  new_sn_m5564    new_sn_m65   new_sp_f014  new_sp_f1524  new_sp_f2534 
    ##          7240          7240          7240          7240          7240 
    ##  new_sp_f3544  new_sp_f4554  new_sp_f5564    new_sp_f65   new_sp_m014 
    ##          7240          7240          7240          7240          7240 
    ##  new_sp_m1524  new_sp_m2534  new_sp_m3544  new_sp_m4554  new_sp_m5564 
    ##          7240          7240          7240          7240          7240 
    ##    new_sp_m65 
    ##          7240

``` r
who3 <- who2 %>% 
  separate(key, into = c("new","type","sexage"), sep = "_")

who3
```

    ## # A tibble: 405,440 x 8
    ##    country     iso2  iso3   year new   type  sexage cases
    ##    <chr>       <chr> <chr> <int> <chr> <chr> <chr>  <int>
    ##  1 Afghanistan AF    AFG    1980 new   sp    m014      NA
    ##  2 Afghanistan AF    AFG    1981 new   sp    m014      NA
    ##  3 Afghanistan AF    AFG    1982 new   sp    m014      NA
    ##  4 Afghanistan AF    AFG    1983 new   sp    m014      NA
    ##  5 Afghanistan AF    AFG    1984 new   sp    m014      NA
    ##  6 Afghanistan AF    AFG    1985 new   sp    m014      NA
    ##  7 Afghanistan AF    AFG    1986 new   sp    m014      NA
    ##  8 Afghanistan AF    AFG    1987 new   sp    m014      NA
    ##  9 Afghanistan AF    AFG    1988 new   sp    m014      NA
    ## 10 Afghanistan AF    AFG    1989 new   sp    m014      NA
    ## # ... with 405,430 more rows

이때, iso2, iso3, new는 모두 같으므로, 제거해도 될 것이다.

``` r
who4 <- who3 %>% 
  select(-iso2,-iso3,-new)

who4
```

    ## # A tibble: 405,440 x 5
    ##    country      year type  sexage cases
    ##    <chr>       <int> <chr> <chr>  <int>
    ##  1 Afghanistan  1980 sp    m014      NA
    ##  2 Afghanistan  1981 sp    m014      NA
    ##  3 Afghanistan  1982 sp    m014      NA
    ##  4 Afghanistan  1983 sp    m014      NA
    ##  5 Afghanistan  1984 sp    m014      NA
    ##  6 Afghanistan  1985 sp    m014      NA
    ##  7 Afghanistan  1986 sp    m014      NA
    ##  8 Afghanistan  1987 sp    m014      NA
    ##  9 Afghanistan  1988 sp    m014      NA
    ## 10 Afghanistan  1989 sp    m014      NA
    ## # ... with 405,430 more rows

다음으로는, sexage를 sex와 age로 분리할 것이다.

``` r
who5 <- who4 %>% 
  separate(sexage, into = c("sex", "age"), sep = 1)

who5
```

    ## # A tibble: 405,440 x 6
    ##    country      year type  sex   age   cases
    ##    <chr>       <int> <chr> <chr> <chr> <int>
    ##  1 Afghanistan  1980 sp    m     014      NA
    ##  2 Afghanistan  1981 sp    m     014      NA
    ##  3 Afghanistan  1982 sp    m     014      NA
    ##  4 Afghanistan  1983 sp    m     014      NA
    ##  5 Afghanistan  1984 sp    m     014      NA
    ##  6 Afghanistan  1985 sp    m     014      NA
    ##  7 Afghanistan  1986 sp    m     014      NA
    ##  8 Afghanistan  1987 sp    m     014      NA
    ##  9 Afghanistan  1988 sp    m     014      NA
    ## 10 Afghanistan  1989 sp    m     014      NA
    ## # ... with 405,430 more rows

``` r
# 각 country, year, sex에 대해 총 결핵 사례 조사하기
who5 %>% 
  group_by(country, year, sex) %>% 
  summarise(cases = sum(cases, na.rm = TRUE))
```

    ## # A tibble: 14,480 x 4
    ## # Groups:   country, year [7,240]
    ##    country      year sex   cases
    ##    <chr>       <int> <chr> <int>
    ##  1 Afghanistan  1980 f         0
    ##  2 Afghanistan  1980 m         0
    ##  3 Afghanistan  1981 f         0
    ##  4 Afghanistan  1981 m         0
    ##  5 Afghanistan  1982 f         0
    ##  6 Afghanistan  1982 m         0
    ##  7 Afghanistan  1983 f         0
    ##  8 Afghanistan  1983 m         0
    ##  9 Afghanistan  1984 f         0
    ## 10 Afghanistan  1984 m         0
    ## # ... with 14,470 more rows

이 모든 과정들을 **파이프 기능을 활용**한다면, 더 간단하게 표현할 수 있을 것이다.

타이디 하지 않은 데이터
=======================

타이디 데이터가 아닌, 다른 데이터 구조를 사용하는 데는 두 가지 이유가 있다.

-   다른 표현 방식이 성능상, 저장용량상 장점이 클 수 있다.
-   전문 분야에서 독자적으로 진화시킨 데이터 저장 규칙이 타이디 데이터의 규칙과는 다를 수 있다.

**이런 이유로 인해 티블(데이터프레임)이 아닌, 다른 것이 필요해진다.**
