---
author: Connie
categories:
- R
- package
- Highcharts
date: "2021-09-22"
draft: false
excerpt: This theme has a form-to-email feature built in, thanks to the simple Formspree
  integration. All you need to activate the form is a valid recipient email address
  saved in the form front matter.
layout: single
links:
- icon: door-open
  icon_pack: fas
  name: website
  url: https://bakeoff.netlify.com/
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/apreshill/bakeoff
subtitle: Highcharter package in R tutorials
tags:
- hugo-site
title: Highcharts for R users
---

## Highcharts Exploration with Highcharter package in R

---


### What is Highcharter Package 

This is the package for Highcharts JavaScript library.It can be used similarly with ggplot2. It can be used free for personal or non-profit purposes but it should be purchased for commercial or public use. 

> Highcharter is a R wrapper for Highcharts javascript library and its modules.

![](https://wp-assets.highcharts.com/svg/highcharts-logo.png)

There are two main functions, which are highcharter() and hchart(). It can be used similar to ggplot2. Firstly, you can make a main chart with those two function and then add other elements with layering. 


two functions:
- highchart() : This function creates a Highchart chart using htmlwidgets. The widget can be rendered on HTML pages generated from R Markdown, Shiny, or other applications.

- hchart() : hchart uses highchart to draw a particular plot for an object of a particular class in a single command. This defines the S3 generic that other classes and packages can extend.




### Introduction 


Reference:
- https://www.highcharts.com/snippets/ 
- https://www.kaggle.com/nulldata/beginners-guide-to-highchart-visual-in-r 
- https://www.kaggle.com/henry090/beautiful-structured-exploration-with-highcharts 
- https://jkunst.com/highcharter/index.html 




Libraries:
```r
library(data.table)
library(stringr)
library(dplyr)
library(kableExtra)
library(highcharter)
library(tidytext)
library(countrycode)
library(purrr)
```



Sample dataset: 
2020 Kaggle ML & Data Science Survey

```r
df<- fread("kaggle_survey_2020_responses.csv", encoding="UTF-8")
df %>% head()
```


```{r}
df_answer<- data.table::fread("kaggle_survey_2020_responses.csv", header = FALSE, skip=2, na.strings = '', encoding="UTF-8")

questions<- names(data.table::fread("kaggle_survey_2020_responses.csv", encoding="UTF-8", skip = 1, nrow=1))

```


sapply : 벡터, 리스트, 표현식, 데이터 프레임 등에 함수를 적용하고 그 결과를 벡터 또는 행렬로 반환한다.
sapply(
  X,    # 벡터, 리스트, 표현식 또는 데이터 프레임
  FUN,  # 적용할 함수
  ...,  # 추가 인자. 이 인자들은 FUN에 전달된다.
)


### question/answer 구분하기 


```{r}
length(questions) #355
seq_len(length(questions)) #숫자형 벡터 반환

pattern=".*\\?"
dupl_q<- sapply(seq_len(length(questions)), function(x) str_extract(questions[x], pattern))

```


### Overview questions 

반복 질문 제거하기
대체적으로 어떤 질문이 survey에서 활용되고 있는가 살펴보기 

```{r}
quiz<- data.frame(Index= 1:length(dupl_q %>% unique()%>% .[-1]), Questions=dupl_q %>% unique() %>% .[-1])



frequency <- as.data.frame(table(dupl_q %>%.[-1]))

frequency

quiz<- inner_join(quiz, frequency, by=c("Questions" = "Var1"))

quiz

quiz<- quiz %>% mutate(Question_type = ifelse(Freq==1, "Single-choice", "Multiple-choice"))


quiz$Question_type<- cell_spec(quiz$Question_type, color = ifelse(quiz$Question_type=="Single-choice", "orange", "green"))



kableExtra::kbl(quiz, escape=F) %>%
    kableExtra::kable_paper() %>% 
    kableExtra::scroll_box(width="700px", height="300px") %>%
    kableExtra::kable_styling(position = "center", full_width = T, font_size = 13)


```


##How to use Highcharter




### bar chart

기본 차트
```{r}
df_answer %>%
  select("V3") %>%
  count(V3) %>%
  hchart("bar", hcaes(x=V3, y=n),
         color=c("orange"))

aes(x=,y)

df_answer %>%
  select("V3") %>%
  count(V3) -> V3

#   
hchart(V3, "bar", hcaes(x="V3", y="n"))
# 


```

가로세로 바꿔보기

```{r}
df_answer %>%
  select("V3") %>%
  count(V3) %>%
  hchart("column", hcaes(x=V3, y=n),
         color=c("orange"))


```



테마 적용해보기
```{r}

# hc_theme() Highcharts is very flexible so you can modify every element of the chart. There are some exiting themes so you can apply style to charts with few lines of code. 
# hc_add_theme()


df_answer %>%
  select("V3") %>%
  count(V3) %>%
  hchart("bar", hcaes(x=V3, y=n),
         name= "The number of respondents",
         color=c("orange")) %>%
  hc_add_theme(hc_theme_bloom()) %>%
  hc_xAxis(title=list(text="Gender"))



```



```{r}

df_answer %>%
  select("V3") %>%
  count(V3) %>%
  hchart("bar", hcaes(x=V3, y=n),
         name= "The number of respondents",
         color=c("orange")) %>%
  hc_add_theme(hc_theme_elementary()) %>%
  hc_xAxis(title=list(text="Gender"))




```




### word cloud


```{r}

#Q27 Which of the following big data products (relational databases, data warehouses, data lakes, or similar) do you use on a regular basis? 


df_answer %>% 
  select(c(V156:V174)) %>% 
  tidyr::pivot_longer(V156:V174,names_to="column", values_to="products",
                      values_drop_na=TRUE) %>%
  count(products) -> list_products


hchart(list_products, "wordcloud", hcaes(name= products, weight=n)) %>%
  hc_add_theme(hc_theme_elementary())



# <word cloud with questions data>
#
#
# text<- data.frame(questions)
# 
# text<- text %>% 
#     unnest_tokens(word, questions)
# 
# 
# text <- text %>% 
#     count(word) %>% 
#     arrange(desc(n)) %>%
#     filter(n<150, n>20)
# 
# 
# hchart(text, "wordcloud", hcaes(name= word, weight=n))



```



### Treemap

```{r}

#Q16 Which of the following machine learning frameworks do you use on a regular basis?


df_answer %>%
  select(c(V67:V82)) %>%
  tidyr::pivot_longer(V67:V82, names_to="column", values_to = "ML", 
                      values_drop_na = TRUE) %>%
  count(ML) %>%
  hchart("treemap",
         hcaes(x=ML, value=n, color=n)) %>%
  hc_add_theme(hc_theme_elementary())
  


```





### sunburst chart


```{r}
sunburst_fun = function(df_answer, col1, col2, col3, cols_names){
  data<- df_answer %>%
    mutate(V4 = ifelse(V4 == "United States of America", "USA", paste(V4))) %>%
    select(!!rlang::sym(col1), !!rlang::sym(col2), rlang::sym(col3))
  
  data$Count = 1
  rename_vars = c(cols_names, "Count")
  names(data) = rename_vars
  count_name = rename_vars[4]
  
  countries = data %>%
    count(!!rlang::sym(cols_names[1]), sort=TRUE) %>% .[1:3,] %>% .[[cols_names[1]]]
  
  
  data2 = data %>%
    filter(!!rlang::sym(cols_names[1]) %in% countries)
  
  
  dout<- data_to_hierarchical(data2, c(!!rlang::sym(cols_names[1]),
                                       !!rlang::sym(cols_names[2]),
                                       !!rlang::sym(cols_names[3])), !!rlang::sym(count_name))
}



dout = sunburst_fun(df_answer, "V4", "V3", "V2", c("Country", "Gender", "Age"))

countries = dout[[1]]
dout = dout[[2]]

hchart(dout, type="sunburst") %>%
  hc_title(text=glue::glue("Kaggle survey TOP 3 countries are"), useHTML=T) %>%
  hc_size(height=600)




data2 = df_answer %>%
  filter(V4 %in% c('United States of America', 'India', 'Other'))





```




### Icons Plot

```{r}

df_answer %>% 
  select(V3) %>%
  unique()




df_answer %>% 
  select(V3) %>%
  filter(V3 == c("Man","Woman")) %>%
  count(V3) %>%
  mutate(percent= round((n/sum(n))*100)) -> gender_icons


# hciconarray(c("Man", "Woman"), gender_icons$percent, icons = c('child')) #hciconarray 더이상 사용되지않음

hciconarray(c("Man", "Woman"), gender_icons$percent, icons = c('male','female'))

hchar("item")

```




### Area chart - backgraound image   

차트 만드는데에 필요한 테이블 만들기

- my_map: 설문조사 응답자의 국가 분포 
- my_map2: 설문조사 응답자의 position 별 분포  
```{r}

my_map <- df_answer[,c(4,6)] %>%
  count(V4, sort=TRUE) %>% #If TRUE, will show the largest groups at the top.
  rename(country=V4, Total_respondents = n)

my_map

my_map2<- df_answer[,c(4,6)] %>%
  count(V4, V6, sort=TRUE) %>%
  rename(country=V4) %>%
  tidyr::spread(.,V6,n) %>%
  left_join(my_map) %>%
  select(-"<NA>")
my_map2



my_map2[is.na(my_map2)]<- 0


```



국가코드를 사용하여(iso2C) 각 국가랑 매칭되는 flag 이미지 삽입할 준비하기. 
(countrycode package 사용)


```{r}

#원래 flag 이미지 링크: 
urlicon<- "url(https://raw.githubusercontent.com/tugmaks/flags/2d15d1870266cf5baefb912378ecfba418826a79/flags/flags-iso/flat/24/%s.png)"


my_map2<- my_map2 %>%
  mutate(countrycode = 
           countrycode(country, origin="country.name", destination="iso2c")) 

my_map2


my_map2<- my_map2%>% #countrycode package
  mutate(marker=sprintf(urlicon, countrycode),
         marker = map(marker, function(x) list(symbol = x)),
         flagicon = sprintf(urlicon, countrycode),
         flagicon = str_replace_all(flagicon, "url\\(|\\)", "")) %>% #Use C-style String Formatting Commands
  arrange(Total_respondents) %>%
  filter(Total_respondents>250) %>%
  mutate(country=ifelse(str_detect(country,"United Kingdom"), "UK", paste(country)),
         country=ifelse(str_detect(country,"United States"), "USA", paste(country)))




#이미지파일 커서 실패: urlicon<- "url(https://raw.githubusercontent.com/hampusborgos/country-flags/main/png250px/%s.png)" 
#/%.png 붙여줘야 함

# my_map2$countrycode<- tolower(my_map2$countrycode)




```


tooltip table 만들어주기 

```{r}

ttvars <- colnames(my_map2)[c(2:15)]

ttvars


tt<- tooltip_table(
  ttvars,
  sprintf("{point.%s}", ttvars), img = tags$img(src="{point.flagicon}", style="text-align: center;")
) #Helper for make table in tooltips(highcharter::)

tt

#Basic-> hchart(my_map2, "areaspline", hcaes(x=country, y=Total_respondents, name=country), name="Nations")


```


areachart 만들기

```{r}

urlimg <- "https://cdn.theatlantic.com/thumbor/_ChTd-1YYOiMmlKu2z6AMwxeFpk=/499x0:2500x2001/1080x1080/media/img/mt/2015/09/intern-2/original.jpg"


hchart(my_map2, "areaspline", 
       hcaes(x=country, y=Total_respondents, name=country), name="Nations") %>% #name=tooltip name 바꿔주는것
  hc_colors(hex_to_rgba("pink", 0.8)) %>% #area color 지정
  hc_chart(divBackgroundImage = urlimg ) %>% #??hc_chart
  hc_title(text="Number of respondents for each Job Title<br> Top 18 countries",
           align="left",
           style=list(color="white")) %>%
  hc_subtitle(text="2020 Kaggle ML & DS Survey",
              align="left",
              style=list(color="white")) %>%
  hc_xAxis(title=list(text="Country", style=list(color="white")),
           labels = list(style = list(color = "pink"))) %>%
  hc_yAxis(title=list(text="N", style=list(color="pink")),
           gridLineWidth= 0,
           labels= list(style = list(color = "white"))) %>% #RGB컬러 지정도 가능 
  hc_tooltip(
    pointFormat=tt,
    headerFormat=as.character(tags$h4("{point.key}", tags$br())),
    positioner = JS("function () { return { x: this.chart.plotLeft + 15, y: this.chart.plotTop + 0 }; }"),
    style = list(color = "grey", fontSize = "0.6em", fontWeight = "normal"),
    # backgroundColor="transparent",
    useHTML=TRUE, #html형태로 
    shape="round"
  ) %>%
  hc_size(height=650) %>%
  hc_credits(
    enabled=TRUE,
    text="Figure from https://www.eclecticpop.com/2015/11/movie-review-intern-2015.html",
    href="https://www.eclecticpop.com/2015/11/movie-review-intern-2015.html",
    styles=list(color="white")
  )
  # hc_add_theme(hc_theme_google())

```



```