# GCNv2 SLAM
## original GCNv2 SLAM
https://github.com/jiexiong2016/GCNv2_SLAM
### Dependencies
C++11 or C++0x Compiler
Pytorch
Pangolin
OpenCV
Eigen3
DBoW2
g2o
### ビルドの方法
#### 環境
自分の環境です。ここでは主にWindowsでのビルド方法を説明します。
Windows10
Visual Studio 2019
CMake 3.23.1
cuda toolkit 10.1
cudnn 7.6.5
libtorch1.7.0+cu101
##### libtorch
libtorchはpipでダウンロードしたものを使用する。私はcudaの都合上1.7.0を使用している。
```
pip install torch==1.7.0+cu101 -f  https://download.pytorch.org/whl/torch_stable.html
```
ダウンロードが完了すると、torchというフォルダが作成されている。その中に、libとincludeというフォルダがあるので、場所が分かりやすい別のフォルダーにコピーしておく。
##### OpenCV
私は、OpenCVは3.2.0で実行できました。また、OpenCVのバージョンが高すぎると、```opencv/cv.h```が使用できない。
##### GCNv2 SLAMのダウンロード
下のコマンドでクローンするか、https://github.com/jiexiong2016/GCNv2_SLAMでダウンロードしてください。
```
git clone https://github.com/jiexiong2016/GCNv2_SLAM.git
```
##### Pangolinとeigenのダウンロード
Pangolinとeigenは別のリポジトリから取ります。例えばhttps://github.com/phdsky/ORBSLAM24Windowsで、入手できます。
##### Thirdpartyのビルド
DBoW2、g2o、PangolinをCMakeでソリューションファイルを生成し、ビルドします。全体的に注意することは、ランタイムライブラリの種類を統一することと.libにすることとReleaseに変更することです。一つ一つプロパティを開いて変更してください。
##### GCNv2 SLAMのCMakeLists.txtを変更
そのまま、CMakeでconfigurarionするとエラーが出ると思います。なぜなら、pytorchの設定やOpenCVのバージョンが原因だからです。よって、CMakeLists.txtのtorchに関する項目を削除したり、OpenCVのバージョンを自分のものに合わせたりしてください。その後にCMakeを使用してください。
##### GCNv2 SLAMのプロパティを変更
まずは、Releaseにすることが先です。その後に、インクルードファイルやリンカーの設定をする。pytorchのlibファイルは多いので、ある程度楽します。pytorchのlibファイルをコピーしたファイル内で、```dir /b *.lib >libs.txt ```をコマンドプロンプトなどで実行することで、libs.txtは静的ライブラリの名前が抽出されています。その中身をコピーしてリンカーのライブラリにペーストしてください。次に、リンカーのその他のオプションを開いて、```/INCLUDE:?warp_size@cuda@at@@YAHXZ ```を追加してください。もし、pytorchのバージョンが1.8.0以降なら```/INCLUDE:?searchsorted_cuda@native@at@@YA?AVTensor@2@AEBV32@0_N1@Z```も追加しておく。
##### 外部ファイルの変更
実行するには、まだ必要なものがある。テスト用データセットとptファイルの変更です。まずは、確認のためにpipでダウンロードしたtorchを使って、下のプログラムを実行してください。
```
import torch

model = torch.jit.load('C:\\Program Files (x86)\\Microsoft Visual Studio\\MyProjects\\GCNv2\\models\\back\\gcn2_320x240.pt')
```
gcn2_320x240.ptのパスは適宜変更してください。実行するとエラーが出るので、gcn2_320x240.ptをWinRAR（https://www.winrarjapan.com/download）をダウンロードして変更する。基本的には、```_32 = torch.squeeze(torch.grid_sampler(input, grid, 0, 0))```の部分を```_32 = torch.squeeze(torch.grid_sampler(input, grid, 0, 0, True))```に変更することで、エラーが出なくなります。
##### プロジェクトファイルの変更
GCNextractor.ccとGCNextractor.hを変更します。
変更前GCNextractor.h
```
std::shared_ptr<torch::jit::script::Module> module;
```
変更後GCNextractor.h
```
torch::jit::Module module;
```
変更前GCNextractor.cc
```
auto output = module->forward(inputs).toTuple();
```
変更後GCNextractor.cc
```
auto output = module.forward(inputs).toTuple();
```
また、```const char *net_fn = "D:\\gcn2_320x240.pt";```ではgcn2_320x240.ptのパスを適宜変更してください。