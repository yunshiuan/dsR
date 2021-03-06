---
title       : Introducing Data Science with R
subtitle    : Lab Session - Classifier
author      : Yu-Yun Chang
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---



## Supervised and unsupervised

> - supervised: given data with labels
> - unsupervised: given data without labels



--- 

## Supervised and unsupervised

> - supervised: classifier
> - unsupervised: cluster



--- 


## Classifiers

- We're going to introduce the usage of a classifier
- There are a lot of classfiers in machine learning:
  - Bayes
  - Decision Tree
  - Neural Network
  - Support Vector Machine
  - there is more ...


---


## Classifier: SVM

- We're going to use Support Vector Machine (SVM) in this lab session

---

## Load the library and data

- install packages <span style="color:blue">e1071</span> and <span style="color:blue">MLmetrics</span>
- these are packages for machine learning and evaluation



```r
require(e1071)
require(MLmetrics)
```

---

## Classifier: SVM


- load in the file: [download here](https://ceiba.ntu.edu.tw/course/582551/data.txt)
- our target: 從 "交往經驗" 和 "被暗戀人數" 推測有機會成為網紅?


```r
df <- read.table('data.txt', sep='\t', header=T)
head(df, 4)
```

```
##   編號   系級    年級      學號   姓名 身高 體重 交往經驗 告白失敗次數
## 1    1 校外生  不知道  es_20227 林帛箴  100   60        7            1
## 2    2 中文系  一年級 t05101308 于佳杏  100   61        8            1
## 3    3 歷史系  四年級 b00203057 簡韻真  101   62        9            1
## 4    4 哲學系  四年級 b02104031 高嘉苓  102   63       10            1
##   被暗戀人數 是否為網紅 身價
## 1          4          y  100
## 2          5          y  100
## 3          6          y  100
## 4          7          y  100
```



---

## SVM Procedure

 - select the <span style="color:blue">features</span> (columns)
 - split the data into <span style="color:blue">trainset</span> (70-80%) and <span style="color:blue">testset</span> (20-30%)
 - feed the <span style="color:blue">features</span> and <span style="color:blue">trainset</span> into the machine
 - evaluate the performance of the machine with <span style="color:blue">testset</span> and <span style="color:blue">F-score</span> 

<br>
> 
<img src="fscore.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="300px" style="display: block; margin: auto;" />



---
## SVM Step by Step

### our target: 從 "交往經驗" 和 "被暗戀人數" 推測有機會成為網紅?

- features: "交往經驗" 和 "被暗戀人數"
- correct answers: "是否為網紅" (*Note that the data type should be a factor)

- 1) extract the features


```r
feature <- df[, c(8, 10, 11)]
head(feature, 4)
```

```
##   交往經驗 被暗戀人數 是否為網紅
## 1        7          4          y
## 2        8          5          y
## 3        9          6          y
## 4       10          7          y
```


--- 
## SVM Step by Step

- 2) split the data into <span style="color:blue">trainset</span> (70-80%) and <span style="color:blue">testset</span> (20-30%)


```r
index <- 1:nrow(df)
testindex <- sample(index, trunc(length(index)*30/100))
trainset <- feature[-testindex,]
testset <- feature[testindex,]
```


--- 
## SVM Step by Step

- 3) feed the <span style="color:blue">features</span> and <span style="color:blue">trainset</span> into the machine


```r
# tune the svm to find out the best cost and gamma
(tuned <- tune.svm(是否為網紅~., data = trainset, cost=10^(-1:2), gamma=c(.5,1,2)))
```

```
## 
## Parameter tuning of 'svm':
## 
## - sampling method: 10-fold cross validation 
## 
## - best parameters:
##  gamma cost
##      1    1
## 
## - best performance: 0.2809524
```


--- 
## SVM Step by Step

- 3) feed the <span style="color:blue">features</span> and <span style="color:blue">trainset</span> into the machine


```r
# train the svm model
model <- svm(是否為網紅~., data = trainset, kernel='linear', cost = 1, gamma = 1)
```

--- 
## SVM Step by Step

- 4) test the performance of the machine with <span style="color:blue">testset</span>


```r
# delete the column with correct answers
# use the trained model to predict the testset
prediction <- predict(model, testset[,-3])
prediction
```

```
## 45 86 87 15 40 22 16 24 13 14 55  4 19 36 56  9 66 76 73 48 20  1 61 67 33 
##  n  n  n  n  y  n  n  n  n  n  n  n  n  y  n  n  n  n  n  n  n  n  n  n  n 
## 75 62 
##  n  n 
## Levels: n y
```



--- 
## SVM Step by Step

- 5) construct a confusion matrix


```r
ConfusionMatrix(prediction, testset[,3])
```

```
##       y_pred
## y_true  n  y
##      n 16  0
##      y  9  2
```



--- 
## SVM Step by Step

- 6) compute the F-score



```r
F1_Score(prediction, testset[,3])
```

```
## [1] 0.7804878
```




---
## Your turn  :D

- 透過哪些 features 可以預測 F-score 最高的身價 svm 分類器

- 請先將 "身價" 分為 2 類:
  - 0~9999
  - 10000~5000000
  

---
## Tips


> - 你可能會想要試著調整以下:
>   - 挑選不同 features 組合
>   - trainset 和 testset 的分配比例
>   - 更改分類器 svm() 內的參數, 例如: kernels
  
<br>   
> - 給不甘於上述, 想要晉升高階大平台者
>   - 查詢 e1071 套件內 tune.svm() 的使用方式調整 gamma 和 cost 值
>   - 對於資料進行 10-fold cross-validation (可使用 caret 套件)
  

