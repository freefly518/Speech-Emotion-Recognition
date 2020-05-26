# Speech Emotion Recognition 

用 SVM、MLP、LSTM 进行语音情感识别，Keras 实现。

改进了特征提取方式，识别准确率提高到了 80% 左右。原来的版本的存档在 [First-Version 分支](https://github.com/Renovamen/Speech-Emotion-Recognition/tree/First-Version)。

[English Document](README_EN.md) | 中文文档

&nbsp;

## Environment

Python 3.6.7

Keras 2.2.4

&nbsp;

## Structure

```
├── models                 // 模型实现
│   ├── common.py          // 所有模型的通用部分（即所有模型都会继承这个类）
│   ├── dnn.py             // LSTM
│   └── ml.py              // SVM & MLP
├── extract_feats          // 特征提取
│   ├── librosa.py         // librosa 提取特征
│   └── opensmile.py       // Opensmile 提取特征
├── misc
│   ├── files.py           // 用于整理数据集（分类、批量重命名）
│   ├── opts.py            // 使用 argparse 从命令行读入参数
│   └── utils.py           // 加载模型、绘图（雷达图、频谱图、波形图）
├── files
│   ├── features           // 存储提取好的特征
│   └── models             // 存储训练好的模型
├── config.py              // 配置参数
├── train.py               // 训练模型
├── predict.py             // 用训练好的模型预测指定音频的情感
├── preprocess.py          // 数据预处理（提取数据集中音频的特征并保存）
└── example.sh             // 命令行输入样例
```

&nbsp;

## Requirments

### Python

- [scikit-learn](https://github.com/scikit-learn/scikit-learn)：SVM & MLP 模型，划分训练集和测试集
- [Keras](https://github.com/keras-team/keras)：LSTM
- [TensorFlow](https://github.com/tensorflow/tensorflow)：Keras 的后端
- [librosa](https://github.com/librosa/librosa)：提取特征、波形图
- [SciPy](https://github.com/scipy/scipy)：频谱图
- [pandas](https://github.com/pandas-dev/pandas)：加载特征
- [Matplotlib](https://github.com/matplotlib/matplotlib)：绘图
- [numpy](github.com/numpy/numpy)

### Tools

- [Opensmile](https://github.com/naxingyu/opensmile)：提取特征

&nbsp;

## Datasets

1. [RAVDESS](https://zenodo.org/record/1188976)

   英文，24 个人（12 名男性，12 名女性）的大约 1500 个音频，表达了 8 种不同的情绪（第三位数字表示情绪类别）：01 = neutral，02 = calm，03 = happy，04 = sad，05 = angry，06 = fearful，07 = disgust，08 = surprised。

2. [SAVEE](http://kahlan.eps.surrey.ac.uk/savee/Download.html)

   英文，4 个人（男性）的大约 500 个音频，表达了 7 种不同的情绪（第一个字母表示情绪类别）：a = anger，d = disgust，f = fear，h = happiness，n = neutral，sa = sadness，su = surprise。

3. [EMO-DB](http://www.emodb.bilderbar.info/download/)

   德语，10 个人（5 名男性，5 名女性）的大约 500 个音频，表达了 7 种不同的情绪（倒数第二个字母表示情绪类别）：N = neutral，W = angry，A = fear，F = happy，T = sad，E = disgust，L = boredom。

4. CASIA

   汉语，4 个人（2 名男性，2 名女性）的大约 1200 个音频，表达了 6 种不同的情绪：neutral，happy，sad，angry，fearful，surprised。

&nbsp;

## Usage

### Prepare

安装依赖：

```python
pip install -r requirements.txt
```

安装 [Opensmile](https://github.com/naxingyu/opensmile)。

&nbsp;

### Configuration

在 [`config.py`](config.py) 中配置参数。

其中 Opensmile 标准特征集目前只支持：

- `IS09_emotion`：[The INTERSPEECH 2009 Emotion Challenge](http://mediatum.ub.tum.de/doc/980035/292947.pdf)，384 个特征；
- `IS10_paraling`：[The INTERSPEECH 2010 Paralinguistic Challenge](https://sail.usc.edu/publications/files/schuller2010_interspeech.pdf)，1582 个特征；
- `IS11_speaker_state`：[The INTERSPEECH 2011 Speaker State Challenge](https://www.phonetik.uni-muenchen.de/forschung/publikationen/Schuller-IS2011.pdf)，4368 个特征；
- `IS12_speaker_trait`：[The INTERSPEECH 2012 Speaker Trait Challenge](http://www5.informatik.uni-erlangen.de/Forschung/Publikationen/2012/Schuller12-TI2.pdf)，6125 个特征；
- `IS13_ComParE`：[The INTERSPEECH 2013 ComParE Challenge](http://www.dcs.gla.ac.uk/~vincia/papers/compare.pdf)，6373 个特征；
- `ComParE_2016`：[The INTERSPEECH 2016 Computational Paralinguistics Challenge](http://www.tangsoo.de/documents/Publications/Schuller16-TI2.pdf)，6373 个特征。

如果需要用其他特征集，可以自行修改 `FEATURE_NUM` 参数。

&nbsp;

### Preprocess

首先需要提取数据集中音频的特征并保存到本地。Opensmile 提取的特征会被保存在 `.csv` 文件中，librosa 提取的特征会被保存在 `.p` 文件中。

| Long option    | Option | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| `--feature`    | `-f`   | 提取特征的方式 [ `o`：Opensmile / `l`：librosa ] [ 默认：`o` ] |

例子：

```python
python preprocess.py -f 'o'
```
[`example.sh`](example.sh) 中有更多的例子。

&nbsp;

### Train

数据集路径可以在 `config.py` 中配置，相同情感的音频放在同一个文件夹里（可以考虑使用 `misc/files.py` 整理数据），如：

```
└── datasets
    ├── angry
    ├── happy
    ├── sad
    ...
```

&nbsp;
命令行参数：

| Long option    | Option | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| `--model_type` | `-mt`  | 模型种类 [ `svm` / `mlp` / `lstm` ] [ 默认：`svm` ]          |
| `--model_name` | `-mn`  | 要保存或加载的模型文件名 [ 默认：`default` ]                 |
| `--feature`    | `-f`   | 提取特征的方式 [ `o`：Opensmile / `l`：librosa ] [ 默认：`o` ] |


例子：

```python
python train.py -mt 'svm' -mn 'SVM' -f 'o'
```
[`example.sh`](example.sh) 中有更多的例子。

&nbsp;

如果不想命令行输入，可以：

```python
from train import train

'''
input params:
	model_name: 模型名称（svm / mlp / lstm）
	save_model_name: 保存模型的文件名
	feature_method: 提取特征的方法（'o': Opensmile / 'l': librosa）
'''
train(model_name = "lstm", save_model_name = "LSTM", feature_method = 'l')
```

&nbsp;

### Predict

用训练好的模型来预测给定音频的情感：

| Long option    | Option | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| `--model_type` | `-mt`  | 模型种类 [ `svm` / `mlp` / `lstm` ] [ 默认：`svm` ]          |
| `--model_name` | `-mn`  | 要保存或加载的模型文件名 [ 默认：`default` ]                 |
| `--feature`    | `-f`   | 提取特征的方式 [ `o`：Opensmile / `l`：librosa ] [ 默认：`o` ] |
| `--audio`      | `-a`   | 要预测的音频的路径 [ 默认：`default.wav` ]                   |

例子：

```python
python predict.py -mt 'svm' -mn 'SVM' -f 'o' -a 'test/happy.wav'
```
[`example.sh`](example.sh) 中有更多的例子。

&nbsp;

如果不想命令行输入，可以：

```python
from misc.utils import load_model
from predict import predict

'''
input params:
	load_model_name: 要加载的模型的文件名
	model_name: 模型名称（svm / mlp / lstm）
return:
	model: 一个模型
'''
model = load_model(load_model_name = "LSTM", model_name = "lstm")

'''
input params:
	model: 加载的模型
	model_name: 模型名称（svm / mlp / lstm）
	file_path: 要预测的文件路径
	feature_method: 提取特征的方法（'o': Opensmile / 'l': librosa）
return: 
	预测结果和概率
'''
predict(model, model_name = "lstm", file_path = 'test/angry.wav', feature_method = 'l')
```

&nbsp;

### Functions
#### Radar Chart

画出预测概率的雷达图。

来源：[Radar](https://github.com/Zhaofan-Su/SpeechEmotionRecognition/blob/master/leidatu.py)

```python
from misc.utils import Radar
'''
输入:
    data_prob: 概率数组
'''
Radar(result_prob)
```

&nbsp;

#### Play Audio

播放一段音频

```python
from misc.utils import playAudio
playAudio(file_path)
```

&nbsp;

#### Plot Curve

画训练过程的准确率曲线和损失曲线。

```python
from misc.utils import plotCurve
'''
输入:
    train(list): 训练集损失值或准确率数组
    val(list): 验证集损失值或准确率数组
    title(str): 图像标题
    y_label(str): y 轴标签
'''
plotCurve(train, val, title, y_label)
```

&nbsp;

#### Waveform

画出音频的波形图。

```python
from misc.utils import Waveform
Waveform(file_path)
```

&nbsp;

#### Spectrogram

画出音频的频谱图。

```python
from misc.utils import Spectrogram
Spectrogram(file_path)
```

&nbsp;

## Other Contributors

- [@Zhaofan-Su](https://github.com/Zhaofan-Su)
- [@Guo Hui](https://github.com/guohui15661353950)