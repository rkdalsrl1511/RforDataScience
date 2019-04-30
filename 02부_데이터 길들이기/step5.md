8장 : readr로 하는 데이터 불러오기
================
huimin
2019년 4월 29일

기본 설정하기
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

일반 텍스트 직사각형 파일을 R로 불러오는 방법<br> readr패키지의 함수들은 기존의 read.csv()함수에 비해서 10배 이상 빠르다.

|      함수     |                설명                |
|:-------------:|:----------------------------------:|
|  read\_csv()  |    쉼표로 구분된 파일을 읽는다.    |
|  read\_csv2() | 세미콜론으로 구분된 파일을 읽는다. |
|  read\_tsv()  |    탭으로 구분된 파일을 읽는다.    |
| read\_delim() |  임의의 구분자로 된 파일을 읽는다. |

``` r
# read_csv
read_csv("a,b,c
         1,2,3
         4,5,6")
```

    ## # A tibble: 2 x 3
    ##       a     b     c
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
# read_csv 옵션 : skip 기능. 첫 n줄을 건너 뛴다.
read_csv("안녕하세요
         안녕하세요
         a,b,c
         1,2,3
         4,5,6", skip = 2)
```

    ## # A tibble: 2 x 3
    ##       a     b     c
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
# read_csv 옵션 : comment 기능. #으로 시작하는 줄을 생략한다.
read_csv("#안녕하세요
         a,b,c
         1,2,3
         4,5,6", comment = "#")
```

    ## # A tibble: 2 x 3
    ##       a     b     c
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
# read_csv 옵션 : col_names 기능.
read_csv("1,2,3
         4,5,6", col_names = FALSE)
```

    ## # A tibble: 2 x 3
    ##      X1    X2    X3
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
# read_csv 옵션 : col_names 기능2.
read_csv("1,2,3
         4,5,6", col_names = c("x","y","z"))
```

    ## # A tibble: 2 x 3
    ##       x     y     z
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
# read_csv 옵션 : NA기능. 결측값을 넣는데에 사용한다.
read_csv("1,2,3
         4,5,.", na = ".")
```

    ## # A tibble: 1 x 3
    ##     `1`   `2` `3`  
    ##   <dbl> <dbl> <lgl>
    ## 1     4     5 NA

벡터 파싱하기
=============

readr이 디스크에서 파일을 읽는 방법에 알아보기 전에 parse\_\*() 함수에 대해 알아볼 필요가 있다.<br> 이 함수들은 문자형 벡터를 입력으로 하여 논리형, 정수형, 날짜형과 같은 특수화된 벡터를 반환한다.

이 함수들은parse\_\*(문자형 벡터, na = "")형태를 공통적으로 가지고 있다. 첫번째 인수는 파싱할 문자형 벡터이며, na인수는 결측으로 처리되어야 하는 문자열을 지정한다.

|        함수        |                          설명                          |
|:------------------:|:------------------------------------------------------:|
|  parse\_logical()  |                   논리형 파싱을 한다.                  |
|  parse\_integer()  |                   정수형 파싱을 한다.                  |
|   parse\_double()  | 엄격한 수치형 파서. 나라마다 방식이 다르므로 복잡하다. |
|   parse\_number()  | 유연한 수치형 파서. 나라마다 방식이 다르므로 복잡하다. |
| parse\_character() |     문자 인코딩에 대해서 생각하기 때문에 복잡하다.     |
|   parse\_factor()  |                   팩터형을 생성한다.                   |
|  parse\_datetime() |                       날짜와 시간                      |
|    parse\_date()   |                       날짜와 시간                      |
|    parse\_time()   |                       날짜와 시간                      |

``` r
# 파싱하기
x <- parse_integer(c("123","345","asbc"))
```

    ## Warning: 1 parsing failure.
    ## row col   expected actual
    ##   3  -- an integer   asbc

``` r
# 출력해보기
print(x)
```

    ## [1] 123 345  NA
    ## attr(,"problems")
    ## # A tibble: 1 x 4
    ##     row   col expected   actual
    ##   <int> <int> <chr>      <chr> 
    ## 1     3    NA an integer asbc

``` r
# 문제점 확인하기
# problems는 티블로 반환하면, dplyr로 작업할 수 있다.
problems(x)
```

    ## # A tibble: 1 x 4
    ##     row   col expected   actual
    ##   <int> <int> <chr>      <chr> 
    ## 1     3    NA an integer asbc

숫자 파싱
---------

생각해야 할 것들이 있다.

-   세계 여러 지역에서 숫자를 다르게 쓴다.
-   숫자는 $, %와 같은 단위를 나타내는 문자가 붙어있을 때가 많다.
-   숫자는 1,000,000와 같이 그룹화 문자가 포함되어 있는 경우가 있다.

**첫번째 문제**를 해결하기 위해서 **지역마다 다른 방식의 파싱 옵션을 지정**하기 위해서 **로케일(locale)**이라는 개념을 사용한다. 숫자를 파싱할 때 가장 중요한 옵션은 소수점으로 사용하는 문자이다. 새로운 로케일을 생성하고 **decimal\_mark 인수를 설정**하여 기본값인 .를 다른 값으로 재정의할 수 있다.

readr의 기본 로케일은 미국이다. **두번째 문제를 처리하는 parse\_number()**는 숫자 앞뒤의 **비수치 문자를 무시**한다. 통화 및 백분율에 유용하지만, 텍스트에 포함된 숫자를 추출하는 데도 효과적이다.

**마지막 문제**는 **parse\_number()와 로케일을 조합**하여 parse\_number()가 그룹화 마크를 무시하도록 함으로써 해결할 수 있다.

``` r
# 숫자 파싱
parse_double("1.23")
```

    ## [1] 1.23

``` r
# decimal_mark 인수를 설정하여 기본값 바꾸기
parse_double("1,23", locale = locale(decimal_mark = ","))
```

    ## [1] 1.23

``` r
# 비수치 문자를 무시하는 파싱
parse_number("$100")
```

    ## [1] 100

``` r
parse_number("20%")
```

    ## [1] 20

``` r
# parse_number와 로케일을 조합하여 그룹화 마크 무시하여 파싱
parse_number("123.456.789", locale = locale(grouping_mark = "."))
```

    ## [1] 123456789

``` r
parse_number("$123,456,789", locale = locale(grouping_mark = ","))
```

    ## [1] 123456789

문자열
------

같은 문자열을 나타내는 방법은 여러 가지이다.<br> **CharToRaw()**를 사용하여 문자열의 기본 표현을 볼 수 있다.<br> 각 16진수 값은 정보 바이트를 나타낸다. 예를 들면 48은 H를 나타내고, 61은 a를 나타낸다. **16진수 수치를 문자로 매핑하는 것을 인코딩이라고 하며**, 앞의 인코딩은 **ASCII**라고 한다.

**ASCII는 정보 교환을 위한 미국 표준 코드**의 줄임말이며, 영문자를 잘 표현한다. 오늘날에 거의 모든 곳에서 지원되는 하나의 표준은 **UTF-8**이 있다.

**readr은 모든 곳에서 UTF-8을 사용한다.** 이를 인식하지 못하는 시스템에서 생성된 데이터에 사용할 수 없다. 따라서 이 문제를 해결하려면 parse\_character()에서 **인코딩을 지정**해야 한다.

readr에서 제공하는 **guess\_encoding()**을 통해서 사용자가 데이터의 인코딩을 확인할 수 있다.

``` r
x1 <- charToRaw("Hadley")
x1
```

    ## [1] 48 61 64 6c 65 79

``` r
guess_encoding(x1)
```

    ## # A tibble: 1 x 2
    ##   encoding confidence
    ##   <chr>         <dbl>
    ## 1 ASCII             1

``` r
x1 <- "Hadley"
x1 <- x1 %>% iconv(from = "EUC-KR", to = "UTF-8")
```

팩터형
------

R은 팩터형을 사용하여, 가질 수 있는 값을 미리 알고 범주형 변수로 나타낸다. 예상치 못한 값이 있을 때마다 경고를 생성하려면 parse\_factor()에 **가질 수 있는 레벨의 벡터를 제공**하면 된다.

``` r
fruit <- c("apple","banana")
parse_factor(c("apple","banana","bananana"), levels = fruit)
```

    ## Warning: 1 parsing failure.
    ## row col           expected   actual
    ##   3  -- value in level set bananana

    ## [1] apple  banana <NA>  
    ## attr(,"problems")
    ## # A tibble: 1 x 4
    ##     row   col expected           actual  
    ##   <int> <int> <chr>              <chr>   
    ## 1     3    NA value in level set bananana
    ## Levels: apple banana

데이트형, 데이트-타임형, 타임형
-------------------------------

세 가지 파서 중에서 선택하면 된다.<br> 자신만의 날짜-시간 형식을 만들어 쓸 수도 있다.

**연** - %Y(4자리) %y(2자리, 00-69 = 2000-2069)<br> **월** - %m(2자리) %b(jan과 같이 축약된 명칭) %B(전체명칭 january)<br> **일** - %d(2자리) %e(선택적 선행 공백)<br> **시간** - %H(0-23 시간 형식) %I(0-12 %p와 함께사용) %p(a.m/p.m)<br> %M(분) %S(초) %OS(실수 초) %Z(시간대 - America/Chicago) %z(UTC 와의 오프셋, 예:+0800)<br> **숫자가 아닌 문자** - %.(숫자가 아닌 문자 하나를 건너뛴다.) %\*(모두)

비영어권의 월 이름에 %b 또는 %B를 사용하는 경우, locale()의 lang 인수를 설정해야 한다. date\_names\_langs()에 내장된 언어 목록을 보라. 자신의 언어가 아직 포함되어 있지 않았으면 date\_names()를 사용하여 생성하라.

``` r
# ISO8601(국제 표준) 년 월 일 시 분 초
parse_datetime("2010-10-01T2010")
```

    ## [1] "2010-10-01 20:10:00 UTC"

``` r
# 연도, -또는/, 월, -또는/, 일을 입력한다.
parse_date("2010-10-01")
```

    ## [1] "2010-10-01"

``` r
# 시, :, 분 그리고 선택적으로 :, 초, 선택적 a.m/p.m 표시를 입력
parse_time("01:10 am")
```

    ## 01:10:00

``` r
parse_time("20:10:01")
```

    ## 20:10:01

``` r
# 포맷 만들기
parse_date("01/02/15", "%m/%d/%y")
```

    ## [1] "2015-01-02"

``` r
parse_date("01/02/15", "%d/%m/%y")
```

    ## [1] "2015-02-01"

``` r
parse_date("01/02/15", "%y/%m/%d")
```

    ## [1] "2001-02-15"

``` r
# 비영어권에서 %B %b 사용 시
parse_date("1 janvier 2015", "%d %B %Y", locale = locale("fr"))
```

    ## [1] "2015-01-01"

파일 파싱하기
=============

개별 벡터를 파싱하는 방법을 배웠으므로, readr이 파일을 파싱하는 방법을 알아볼 차례이다.

-   readr이 각 열의 유형을 자동으로 추측하는 방법
-   기본 사양을 재정의하는 방법

전략
----

readr은 휴리스틱 방법을 사용하여 각 열의 유형을 파악한다. 첫 번째 1000행을 읽고 휴리스틱 방법을 사용하여 각 열의 유형을 찾는다.

guess\_parser()와 parse\_guess()를 사용하여 문자형 벡터에 이 과정을 재현해볼 수 있다.

``` r
guess_parser("2010-10-01")
```

    ## [1] "date"

``` r
guess_parser("15:01")
```

    ## [1] "time"

``` r
guess_parser(c("TRUE","FALSE"))
```

    ## [1] "logical"

``` r
guess_parser(c("1","5","9"))
```

    ## [1] "double"

``` r
guess_parser(c("12,352,561"))
```

    ## [1] "number"

``` r
str(parse_guess("2010-10-01"))
```

    ##  Date[1:1], format: "2010-10-01"

이 휴리스틱 방법은 각각의 형식들에 일치하는 항목을 찾으면 멈춘다. 규칙 중 어느 것도 적용되지 않으면 해당 열은 문자열 벡터로 남긴다.

**문제점**

-   처음 1000행이 특수한 경우이어서 readr이 충분히 일반적이지 않은 유형으로 추측할 수 있다. 예를 들어서 1000행까지 정수만 있는 더블형 열
-   열에 결측값이 많이 있을 수 있다. 첫 번째 1000개의 행에 NA만 있는 경우 readr이 문자형 벡터로 추측했지만, 좀 더 구체적으로 파싱하고 싶을 수 있다.

``` r
# 1000개의 행을 보고 생성된 열 상세 내용과 첫 5개 행의 파싱 오류 발생
challenge <- read_csv(readr_example("challenge.csv"))
```

    ## Parsed with column specification:
    ## cols(
    ##   x = col_double(),
    ##   y = col_logical()
    ## )

    ## Warning: 1000 parsing failures.
    ##  row col           expected     actual                                               file
    ## 1001   y 1/0/T/F/TRUE/FALSE 2015-01-16 'C:/R/R-3.6.0/library/readr/extdata/challenge.csv'
    ## 1002   y 1/0/T/F/TRUE/FALSE 2018-05-18 'C:/R/R-3.6.0/library/readr/extdata/challenge.csv'
    ## 1003   y 1/0/T/F/TRUE/FALSE 2015-09-05 'C:/R/R-3.6.0/library/readr/extdata/challenge.csv'
    ## 1004   y 1/0/T/F/TRUE/FALSE 2012-11-28 'C:/R/R-3.6.0/library/readr/extdata/challenge.csv'
    ## 1005   y 1/0/T/F/TRUE/FALSE 2020-01-13 'C:/R/R-3.6.0/library/readr/extdata/challenge.csv'
    ## .... ... .................. .......... ..................................................
    ## See problems(...) for more details.

``` r
challenge[1:10,]
```

    ## # A tibble: 10 x 2
    ##        x y    
    ##    <dbl> <lgl>
    ##  1   404 NA   
    ##  2  4172 NA   
    ##  3  3004 NA   
    ##  4   787 NA   
    ##  5    37 NA   
    ##  6  2332 NA   
    ##  7  2489 NA   
    ##  8  1449 NA   
    ##  9  3665 NA   
    ## 10  3863 NA

``` r
challenge[1001:1010,]
```

    ## # A tibble: 10 x 2
    ##        x y    
    ##    <dbl> <lgl>
    ##  1 0.238 NA   
    ##  2 0.412 NA   
    ##  3 0.746 NA   
    ##  4 0.723 NA   
    ##  5 0.615 NA   
    ##  6 0.474 NA   
    ##  7 0.578 NA   
    ##  8 0.242 NA   
    ##  9 0.114 NA   
    ## 10 0.298 NA

``` r
problems(challenge)
```

    ## # A tibble: 1,000 x 5
    ##      row col   expected        actual    file                              
    ##    <int> <chr> <chr>           <chr>     <chr>                             
    ##  1  1001 y     1/0/T/F/TRUE/F~ 2015-01-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  2  1002 y     1/0/T/F/TRUE/F~ 2018-05-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  3  1003 y     1/0/T/F/TRUE/F~ 2015-09-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  4  1004 y     1/0/T/F/TRUE/F~ 2012-11-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  5  1005 y     1/0/T/F/TRUE/F~ 2020-01-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  6  1006 y     1/0/T/F/TRUE/F~ 2016-04-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  7  1007 y     1/0/T/F/TRUE/F~ 2011-05-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  8  1008 y     1/0/T/F/TRUE/F~ 2020-07-~ 'C:/R/R-3.6.0/library/readr/extda~
    ##  9  1009 y     1/0/T/F/TRUE/F~ 2011-04-~ 'C:/R/R-3.6.0/library/readr/extda~
    ## 10  1010 y     1/0/T/F/TRUE/F~ 2010-05-~ 'C:/R/R-3.6.0/library/readr/extda~
    ## # ... with 990 more rows

``` r
# 열단위 작업을 통한 파싱 문제 해결
challenge <- read_csv(readr_example("challenge.csv"),
                      col_types = cols(x = col_double(),
                                       y = col_date()))

head(challenge)
```

    ## # A tibble: 6 x 2
    ##       x y         
    ##   <dbl> <date>    
    ## 1   404 NA        
    ## 2  4172 NA        
    ## 3  3004 NA        
    ## 4   787 NA        
    ## 5    37 NA        
    ## 6  2332 NA

``` r
tail(challenge)
```

    ## # A tibble: 6 x 2
    ##       x y         
    ##   <dbl> <date>    
    ## 1 0.805 2019-11-21
    ## 2 0.164 2018-03-29
    ## 3 0.472 2014-08-04
    ## 4 0.718 2015-08-16
    ## 5 0.270 2020-02-04
    ## 6 0.608 2019-01-06

모든 parse\_xyz()함수는 해당하는 col\_xyz()함수를 가지고 있다. **col\_types를 항상 설정**하여 readr이 생성하는 출력물로부터 만들어 나가는 것을 추천한다.

기타 전략
---------

**더 많은 행을 살펴보기**

``` r
# 1행만 더 살펴본다면, 파싱에 문제가 있었음을 알 수 있다.
challenge <- readr::read_csv(readr_example("challenge.csv"),
                             guess_max = 1001)
```

    ## Parsed with column specification:
    ## cols(
    ##   x = col_double(),
    ##   y = col_date(format = "")
    ## )

**모든 열을 문자형 벡터로 읽으면 문제를 쉽게 진달할 수 있다.**

``` r
challenge <- readr::read_csv(readr_example("challenge.csv"),
                             col_types =cols(.default=col_character()))


#이 방법은 type_convert()와 사용하면 유용하다. 휴리스틱 파싱 방법을 데이터 프레임의 문자형 열에 적용한다.
type_convert(challenge)
```

    ## Parsed with column specification:
    ## cols(
    ##   x = col_double(),
    ##   y = col_date(format = "")
    ## )

    ## # A tibble: 2,000 x 2
    ##        x y         
    ##    <dbl> <date>    
    ##  1   404 NA        
    ##  2  4172 NA        
    ##  3  3004 NA        
    ##  4   787 NA        
    ##  5    37 NA        
    ##  6  2332 NA        
    ##  7  2489 NA        
    ##  8  1449 NA        
    ##  9  3665 NA        
    ## 10  3863 NA        
    ## # ... with 1,990 more rows

이외에도,

-   매우 큰 파일을 읽는 경우, n\_max를 10,000 또는 100,000과 같이 작은 숫자로 설정할 수 있다. 이렇게 하면 일반적인 문제를 해결하는 동시에 반복작업을 가속화할 수 있다.
-   파싱에 중대한 문제가 있는 경우에 read\_lines()을 이용하여 라인으로 이루어진 문자형 벡터로 읽거나 read\_file()을 이용하여 길이가 1인 문자형 벡터로 읽는 것이 더 쉬울 수 있다. 그런 다음 나중에 배울 문자열 파싱 방법을 사용하여 좀 더 특이한 포맷을 파싱하면 된다.

과같은 전략들이 있다.

파일에 쓰기
===========

readr 패키지에는 디스크에 데이터를 다시 기록하는 데 유용한 함수, **write\_csv()와 write\_tsv()**가 있다. 이 두 함수는 다음과 같은 특징이 있다.

-   항상 UTF-8로 문자열을 인코딩한다.
-   데이트형과 데이터-타임형을 ISO 8601 형식으로 저장하여 어디에서든 쉽게 파싱될 수 있게 한다.

CSV 파일을 엑셀로 내보내려면 write\_excel\_csv() 함수를 사용한다. 대표적인 인수로는 **x(저장할 데이터프레임)와 path**가 있다.

참고로 CSV로 저장을 한 후, 다시 read\_csv()로 불러올 때에는 **기존에 저장되어 있던 유형 정보가 초기화된다.** 예를들어서, 팩터형 변수는 저장되면서 다시 문자형으로 초기화된다. 따라서 두 가지 대안을 사용한다.

-   **write\_rds()와 read\_rds()**는 베이스 함수인 readRDS() saveRDS()의 래퍼 함수들이다. 이들은 RDS라는 R의 커스텀 바이너리 형식으로 데이터를 저장한다.
-   **feather 패키지**의 **wirte\_feather()와 read\_feather()**는 다른 프로그래밍 언어와 공유할 수 있는 빠른 바이너리 파일 형식을 구현한다.

feather는 RDS보다 빠르며 R 외부에서도 사용할 수 있다.

기타 데이터 유형
================

다른 유형의 데이터를 R로 불러오려면 다음에 나열된 tidyverse 패키지로 시작하는 것이 좋다.

-   haven은 SPSS, Stata, SAS 파일을 읽을 수 있다.
-   readxl은 엑셀 파일(.xls와 .xlsx)을 읽을 수 있다.
-   DBI를 데이터베이스 특화 백엔드와 함께 사용하면 데이터베이스에 대해 SQL 쿼리를 실행하고 데이터프레임을 반환할 수 있다.

JSON에는 jsonlite 패키지를 사용하고 XML에는 xml2를 사용하면 된다.
