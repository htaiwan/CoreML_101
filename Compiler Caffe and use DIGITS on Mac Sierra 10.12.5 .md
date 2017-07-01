# Compiler Caffe and use DIGITS on Mac Sierra 10.12.5 

## 參考資料
### 主要參考了下列資料，但實作上，還是要因個人的系統設定，會有不同狀況產生。
* [Installing Caffe on Mac 10.11.5 and later in the 10.11 series, and 10.12.1 and later in the 10.12 series](https://gist.github.com/doctorpangloss/f8463bddce2a91b949639522ea1dcbe4)
* [在 macOS Sierra 10.12.2 上編譯 Caffe 並使用 DIGITS](https://blog.birkhoff.me/macos-sierra-10-12-2-build-caffe/)
* [Building DIGITS](https://github.com/NVIDIA/DIGITS/blob/digits-3.0/docs/BuildDigits.md)
* [Mac Operating System Support in CUDA 8.0](http://docs.nvidia.com/cuda/cuda-installation-guide-mac-os-x/index.html#system-requirements)

## 步驟
### 1. 先Clone the caffe & DIGITS repo
```
cd ~/Documents
git clone https://github.com/BVLC/caffe.git
git clone https://github.com/NVIDIA/DIGITS.git
```
完成後，會在Documents資料夾中，看到caffe跟DIGITS兩個資料夾。

### 2. 確認透過 homebrew 安裝的 Python 已經刪除
```
brew uninstall python  
```
因為mac已經有內建Python，我們直接用系統提供的即可。

### 3. 安裝相關的dependency
```
brew install -vd snappy leveldb gflags glog szip lmdb
brew install hdf5 opencv
brew upgrade libpng
brew tap homebrew/versions

// 編譯並安裝 protobuf
brew install --build-from-source --with-python -vd protobuf
// 編譯並安裝 boost
brew install --build-from-source -vd boost159 boost-python159
brew link --force boost159  
```

### 4. 編譯caffe
#### 4.1 修改Caffe中提供的config.example

```
cd caffe 
cp Makefile.config.example Makefile.config
```
##### 打開 Makefile.config，進行下列修改

* 變更 BLAS_INCLUDE

```
BLAS_INCLUDE := /Applications/Xcode.app/Contents/Developer/Platforms
MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/System/Library/Frameworks/
Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers

```

* 變更 BLAS_LIB

```
BLAS_LIB := /System/Library/Frameworks/Accelerate.framework/Versions/A/
Frameworks/vecLib.framework/Versions/A
```

* 變更 PYTHON_INCLUDE

```
PYTHON_INCLUDE := /usr/include/python2.7 \  
     /Library/Python/2.7/site-packages/numpy/core/include 
```

* 將 WITH_PYTHON_LAYER := 1 那一行的 "#" 移除
* 將 USE_LEVELDB := 0 那一行的 "#" 移除
* 將 CPU_ONLY := 1 那一行的 "#" 移除 (如果出現"check failed: error = cudaSuccess (35 vs 0) CUDA driver version is insufficient for CUDA runtime version."，就把這行comment out，因為GPU並未支援CUDA driver，就單純靠自己的CPU)

#### 4.2 安裝 Xcode Command Line Tools 7.3
安裝完成後，執行

```
sudo xcode-select --switch /Library/Developer/CommandLineTools  
```

#### 4.3 開始編譯caffe

```
make -j8 all
sudo -H easy_install pip
pip install --user -r python/requirements.txt
make -j8 pytest
make -j8 test  
make -j8 runtest  
make -j8 pycaffe  
make -j8 pytest 

make distribute

cp -r distribute/python/caffe ~/Library/Python/2.7/lib/python/site-packages/

cp distribute/lib/libcaffe.so.1.0.0-rc3 ~/Library/Python/2.7/lib/python/site-packages/
caffe/libcaffe.so.1.0.0-rc3

install_name_tool -change @rpath/libcaffe.so.1.0.0 ~/Library/Python/2.7/lib/python/site-packages/caffe/libcaffe.so.1.0.0 
~/Library/Python/2.7/lib/python/site-packages/caffe/_caffe.so

```

#### 4.4 驗證編譯完成caffe是否OK
執行下列指令，若沒出現錯誤，就表示已經順利完成Caffe的編譯

```
python -c 'import caffe'

```

### 5. 執行DIGITS

#### 5.1 設定變數
在.bashrc，加入這兩行，若沒有這個.bashrc，就建立一個(我是放在/Users/XXX/)

```
export DIGITS_ROOT="~/Documents/DIGITS"  
export CAFFE_ROOT='~/Documents/caffe'

```
重啟shell，讓系統載入剛剛的設定

```
source ~/.bashrc

```

#### 5.2 修改digits-devserver
進入Digits目錄下，執行vim ./digits-devserver，把最後一行python 改為 python2.7

#### 5.3 執行

```
./digits-devserver

```

### 5.4 驗證
接著開啟 http://localhost:5000/ 就可以看到 DIGITS 首頁


