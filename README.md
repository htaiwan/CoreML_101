# CoreML_101

## 	前言

#### 目前在apple developer官網和github上都可以找很多相關tutorial，但主要都侷限拿現成Core ML model，但如果我想要有自己的一個客製化model該怎麼做呢？所以我想要從頭開始學會打造一個自己的model。例如，我想要有一個model可以幫我區別不同的遊戲機圖片(PS4, XBOX, Will)那又該怎麼做？


## 	官方文件 - [Core ML](https://developer.apple.com/machine-learning/)

#### 在我看完官方文件跟範例後，可以發現到apple並沒有提到要如何自己做一個客製化model，只提到有一個python tool可以把目前流行的machine learning model(Caffe, Keras...)轉換Core ML model([Converting Trained Models to Core ML](https://developer.apple.com/documentation/coreml/converting_trained_models_to_core_ml))


## 	步驟

### 1. 建立可以產生Caffe model的環境 
#### 萬事起頭難，建立環境真的是最困難的一件事，這關我卡滿久的。

[在mac順利執行Digits](https://github.com/htaiwan/CoreML_101/blob/master/Compiler%20Caffe%20and%20use%20DIGITS%20on%20Mac%20Sierra%2010.12.5%20.md)

### 2. 訓練自己的Caffe model
#### 當環境建立完成，就可以透Digits訓練自己的model可以區分不同圖片，Digits是一個非常方便GUI tool，可以簡易的透過UI介面的操作完成訓練，當然這些操作也可以透過寫程式來執行，但在我選擇了digits來簡化入門的學習。

[Digits](https://github.com/NVIDIA/DIGITS)

[機器學習動手玩：給新手的教學](https://github.com/humphd/have-fun-with-machine-learning/blob/master/README_zh-tw.md)


### 3. 轉換(將Caffe轉換成CoreML)
####  如何透過apple提供pyton tool進行轉換([coremltools](https://pypi.python.org/pypi/coremltools))。

#### 在安裝coremltools，一直無法順利安裝，後來是參考這篇([RuntimeError: Unable to load Caffe](https://forums.developer.apple.com/thread/78826))才順利完成安裝。

#### 3.1 首先將下面三個檔案放在同個資料夾

```
coreml.py // 自行建立test.caffemodel // 透過digit訓練完成的caffmodel，可以由digit直接outputtest.prototxt // 透過digit訓練完成的caffmodel，可以由digit直接output

```

#### 3.2 coreml.py內容(可以參考文件的coreml tool文件上的api說明，我這邊只利用最基本的幾個api，能順利編譯出來CoreML model 

```
import coremltools

coreml_model = coremltools.converters.caffe.convert(
    ('test.caffemodel', 'test.prototxt'))
    
coreml_model.author = 'XXXX'
coreml_model.license = 'XXXX'
coreml_model.short_description = "XXXX"

coreml_model.input_description['data'] = 'Input image to be classified'
coreml_model.output_description['softmax'] = 'Most likely image category'

coreml_model.save('test.mlmodel')

```

#### 3.3 進入資料夾後，在terminal下執行 python coreml.py，如果一切順利應該就會在同個資料夾下看到自己產生的CoreML model了，如此一來就大功告成。


### 4. 整合
#### 我順利把自己產生的CoreML model放進Xcode8中，並產生對應的class，接下來就是後續的應用了。


## 	後續

#### 前前後後也差不多花了兩個禮拜再嘗試研究CoreML，除了剛開始的環境建立真的比較頭疼外，現在我仍在研究如何拿現成別人訓練完成Caffe model來進行fine tuning，訓練自己想要的辨識的圖片，這必須透過修改train_val.prototxt，但這檔案真的是博大精深，有很多東西需要再研究。
