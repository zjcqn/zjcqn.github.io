---
title: Kaldi实践 Aishell v1
author: 陈钱牛
date: 2020-09-23 16:20:00 +0800
layout: post
categories: Implementation
tags: [Study, ASV, Kaldi]
typora-root-url: ..
math: true
banner:
  video:
  loop: true
  volume: 0.8
  start_at: 8.5
  image: /assets/images/default/gnu.jpeg
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
---


# 总流程

1. 数据准备
2. 提取MFCC
3. 训练GMM-UBM
4. 训练和提取ivector
5. 训练PLDA模型
6. 划分训练集（test）为enroll（注册集）和eval（测试集）
7. 提取enroll和eval的ivector并计算PLDA结果 

## 目录结构

```shell
|—— cmd.sh              # 修改集群多线程版本或本地运行版本
|—— conf                 
|   |—— mfcc.conf   	# MFCC参数配置文件
|    —— vad.conf    	# VAD(语音激活检测)激活配置
|—— data                # 下载的数据集存放路径，run.sh中修改新建的数据下载路径
|   |—— data_aishell.tgz     
|    —— resource_aishell.tgz
|—— local               #本地脚本文件,一般不可直接移植
|   |—— aishell_data_prep.sh
|   |—— download_and_untar.sh
|   |——produce_trials.py
|    —— split_data_enroll_eval.py
|—— path.sh              # 设置环境变量
|—— README.txt      
|—— run.sh		 #主脚本，调用cmd.sh和path.sh
|—— sid -> ../../sre08/v1/sid	# sid,steps,utils中的脚本较为通用，一般可以移植
|—— steps -> ../../wsj/s5/steps
 —— utils -> ../../wsj/s5/utils
```

## 主函数脚本run.sh

```shell
|-data/data_url   	#配置数据集存放路径和下载路径
|-cmd.sh/path.sh  	#线程设置及环境变量初始化
|-download		#下载数据集
|   `-local/download_and_untar.sh  	#下载数据集脚本
|   `-local/download_and_untar.sh   	#下载字典信息文件脚本
|-数据准备阶段
|   `-local/aishell_data_prep.sh
|   `-MFCC特征提取部分
|       `-steps/make_mfcc.sh MFCC脚本
|       `-sid/compute_vad_decision.sh 	#计算语音激活检测(端点检测)
|       `-utils/fix_data_dir.sh 	#数据修复脚本
|-训练diag UBM
|   `-train_diag_ubm.sh 训练ubm脚本
|-训练full UBM
|   `-train_full_ubm.sh
|-训练ivector模型
|   `-train_ivector_extractor.sh 
|-提取ivector
|   `-extract_ivectors.sh
|-训练PLDA(概率线性判别分析)模型
|   `-$train_cmd
|-测试集数据注册和测试
|   `-mkdir/cp/cp  #创建注册/测试的文件路径，将数据文件放到新的路径下
|   `-local/split_data_enroll_eval.py 		#对测试数据路径下的文件进行注册/测试操作
|   `-local/produce_trials.py  		 	#对测试者的语音数据进行测试 
|   `utils/fix_data_dir.sh 			#对enroll/eval数据进行数据修复操作
|-提取注册人的ivector
|   `-sid/extract_ivectors.sh
|_提取测试者的ivector
|   `-extract_ivector.sh
|-计算plda的打分
|   `-ivector-plda-scoring
|-计算eer
```

# AISHELL数据集介绍

aishell是包含178小时的中文语料集，包括两个压缩包: 

- ``data_aishell`` ：包括音频文件和对应的字幕，分为train（340人）、dev（40人，没用到）、test（20人）三个文件夹
- ``resource_aishell``：包括词典，在本项目中应用不到

其中有train（340人）、dev（40人）（好像没用到）、test（20人）三个文件夹，

# 1. 数据准备

```shell
# Data Preparation
local/aishell_data_prep.sh $data/data_aishell/wav $data/data_aishell/transcript
```

## 实现功能

- 数据下载检验解压
- 创建kaldi音频处理所必须的必要数据utt2spk、wav.scp、text等，这样就不需要我们自己手动输入了。在./data/local下创建train、dev、test文件夹，各有一份上述文件。

## 生成文件

- ``utt2spk``：语句和说话人的对应关系
- ``wav.scp``：语句和音频的对应关系
- ``test``：语句和字幕的对应关系
- ``spk2utt``：说话人和语句的对应关系

# 2. 提取MFCC

```shell
# Now make MFCC  features.
# mfccdir should be some place with a largish disk where you
# want to store MFCC features.
mfccdir=mfcc
for x in train test; do
  steps/make_mfcc.sh --cmd "$train_cmd" --nj 10 data/$x exp/make_mfcc/$x $mfccdir
  sid/compute_vad_decision.sh --nj 10 --cmd "$train_cmd" data/$x exp/make_mfcc/$x $mfccdir
  utils/fix_data_dir.sh data/$x
done
```

## 实现功能

- 用``steps/make_mfcc.sh`` 来提取MFCC特征
- 通过``sid/compute_vad_decision.sh`` 进行数据预处理，执行语音激活检测算法，用于判断什么时候有语音输出，什么时候是静音状态。通过使用VAD，我们可以找到有效语音段，剔除静音段，在语音识别等过程中可以大大减少要处理的数据量。

## 生成数据

- ``mfccdir``: 保存了提取的特征
- ``exp/make_mfcc``: 保存了日志，即log 文件

# 3. 训练GMM-UBM

```shell
# train diag ubm
sid/train_diag_ubm.sh --nj 10 --cmd "$train_cmd" --num-threads 16 \
  data/train 1024 exp/diag_ubm_1024

#train full ubm
sid/train_full_ubm.sh --nj 10 --cmd "$train_cmd" data/train \
  exp/diag_ubm_1024 exp/full_ubm_1024
```



分成两部分 训练diag_UBM和训练full_UBM，用先训练的diag_UBM来训练完整的UBM，可以加快运算速度同时提高效果

## 生成文件

第一步产生文件：``exp/diag_ubm_1024/final.dubm``，转换成文本：

```shell
gmm-global-copy –binary=false final.dubm final_dubm.txt
```

第二步产生文件： ``exp/full_ubm_1024/final.ubm``，转换成文本：

```shell
fgmm-global-copy –binary=false final.ubm final_ubm.txt
```

``final.dubm``文件内容：

```shell
GCONSTS 		#这个值为一个1024维向量，是便于计算用的常量
WEIGHT 			#权重
INV_VARS 		#方差矩阵的逆矩阵
MEANS_INVVARS  	#均值向量的转置*方差矩阵的逆矩阵
final.ubm		#文件内容和上一个相同，其中INV_VARS为1024个60×60的下三角矩阵
```




# 4. 训练ivector

```shell
#train ivector
sid/train_ivector_extractor.sh --cmd "$train_cmd --mem 10G" \
  --num-iters 5 exp/full_ubm_1024/final.ubm data/train \
  exp/extractor_1024
#extract ivector
sid/extract_ivectors.sh --cmd "$train_cmd" --nj 10 \
exp/extractor_1024 data/train exp/ivector_train_1024
```

**训练ivector实际是训练T矩阵的过程**，其整体流程如下：

1.  由``final.ubm``得到``final.dubm``
2.  用``final.ubm``初始化``ivector``，即T矩阵
3.  `` final.dubm``选择前20个高斯分量，并输出后验概率
4.  迭代更新5次，得到最终的T矩阵

## 生成文件

``exp/extractor_1024/final.ie``转成文本:

```shell
ivector-extractor-copy –binary=false final.ie final_ie.txt
```

文件内容：

```shell
w_vec 		#ubm高斯分量的权重
M_2014 		#T矩阵，1024·60*400维矩阵
SigmaInv 	#ubm的协方差矩阵 1024个60*60的下三角矩阵
```

# 5.提取ivector 

这一步才是真正的提取ivector，将**训练好的T**矩阵带入计算公式，从而获得每句话的ivector。
介绍extract_ivectors.sh中的几个可执行文件。

```shell
gmm-gselect #本实验中一共采用了1024个高斯，在计算充分统计量时需要用到类条件概率，但实际上有前若干个类条件概率较大的高斯就能描述该帧的概率分布情况，这里取了前30个。
fgmm-global-gselect-to-post #选定了30个高斯之后输出每一帧的后验概率
ivector-extract #提取每句话的ivector, 生成ivector.JOB.scp JOB根据自己的电脑配置来设置。
ivector-normalize-length #对每一句话的ivector进行正则化处理
ivector-mean #在ivector的理论中，假设每句话代表一个说话人，所以最后要对同一说话人的不同语音进行ivector平均化，从而得到每个人的ivector.最后再进行一次对每个人的ivetor的正则化处理
```

# 6.训练PLDA模型

```shell
#train plda
$train_cmd exp/ivector_train_1024/log/plda.log \
	ivector-compute-plda ark:data/train/spk2utt \
	'ark:ivector-normalize-length scp:exp/ivector_train_1024/ivector.scp  ark:-     |' \
	exp/ivector_train_1024/plda
```

ivector是全局差异因子，同时包含了说话人之间的差异和信道差异，于是我们用PLDA来做一个优化处理，**剥离掉其中的信道差异**。使用PLDA来作一个映射，将每一帧的PLDA是一个生成式模型，用EM算法来估计参数，我们定义第 i 个人的第 j 语音为 $x_{ij}$其模型的生成式可表示为：
$$
x_{ij}=\mu+Fh_i+Gw_{ij}+\epsilon_{ij}
$$

其中，

$\mu$ 表示全体训练数据的均值；
$F$可以看做是身份空间，包含了可以用来表示各种说话人的信息；
$h_i$ 就可以看做是具体的一个说话人的身份(或者是说话人在身份空间中的位置)；
$G$ 可以看做是误差空间，包含了可以用来表示同一说话人不同语音变化的信息；
$w_{ij}$ 表示的是在G空间中的位置；
$ϵ_{ij}$ 是最后的残留噪声项，用来表示尚未解释的东西。该项为零均高斯分布，方差为Σ。
这个过程就是训练$F$、$Σ$并计算$\mu$。

## 生成文件

``exp/ivector_train_1024/plda``，将文件转换成文本格式：

```shell
ivector-copy-plda –-binary=false plda plda.txt
```

文件内容：
（1） 所有语音ivector的均值 400维向量
（2） 说话人差异矩阵F 400*400维矩阵
（3） 协方差对角阵$Σ$ 400维向量形式

# 7. 划分训练集（test）并提取ivector

```shell
#split the test to enroll and eval
mkdir -p data/test/enroll data/test/eval
cp data/test/{spk2utt,feats.scp,vad.scp} data/test/enroll
cp data/test/{spk2utt,feats.scp,vad.scp} data/test/eval
local/split_data_enroll_eval.py data/test/utt2spk  data/test/enroll/utt2spk  data/test/eval/utt2spk
trials=data/test/aishell_speaker_ver.lst
local/produce_trials.py data/test/eval/utt2spk $trials
utils/fix_data_dir.sh data/test/enroll
utils/fix_data_dir.sh data/test/eval
#extract enroll ivector
sid/extract_ivectors.sh --cmd "$train_cmd" --nj 10 \
  exp/extractor_1024 data/test/enroll  exp/ivector_enroll_1024
#extract eval ivector
sid/extract_ivectors.sh --cmd "$train_cmd" --nj 10 \
  exp/extractor_1024 data/test/eval  exp/ivector_eval_1024
```

得到两个集的ivector``exp/ivector_enroll_1024``和``exp/ivector_eval_1024`` ，用于PLDA计算判别

# 8. 计算结果

```shell
#compute plda score
$train_cmd exp/ivector_eval_1024/log/plda_score.log \
  ivector-plda-scoring --num-utts=ark:exp/ivector_enroll_1024/num_utts.ark \
  	exp/ivector_train_1024/plda \
  	ark:exp/ivector_enroll_1024/spk_ivector.ark \
  	"ark:ivector-normalize-length scp:exp/ivector_eval_1024/ivector.scp ark:- |" \
  	"cat '$trials' | awk '{print \\\$2, \\\$1}' |" exp/trials_out
```



这个过程的可执行文件为``ivector-plda-scoring``

二进制文件``kaldi/src/ivectorbin/ivector-plda-scoring``的用法

```shell
Usage: ivector-plda-scoring <plda> <train-ivector-rspecifier> <test-ivector-rspecifier> <trials-rxfilename> <scores-wxfilename>
e.g.: ivector-plda-scoring --num-utts=ark:exp/train/num_utts.ark plda ark:exp/train/spk_ivectors.ark ark:exp/test/ivectors.ark trials scores
```

参数部分:

- 第一个是plda模型，
- 第二个是注册集中每个人的ivector，
- 第三个是经过归一化处理的评估集中每条语音的ivector，
- 第四个参数是说话人和测试语音之间的待PLDA测试的配对关系
- 第五个参数则是分数存储的文件。

选项部分:是读取每个说话人的语音的数量

具体的打分过程是：程序首先读取输入的注册集和测试集文件，然后按行读取trials的内容，比如说上述trials文件的第一行，以<注册集说话人id> <测试语音id> 的方式读取，（egs:S0764，BAC009S0764207)然后分别去两个输入文件中寻找对应的i-vector，计算PLDA得分，将<注册集说话人id> <测试语音id><得分> 写入到trials_out中，将二者带入PLDA的公式中，则可计算出得分。最后输出到trials_out中，格式为<语音><说话人><匹配得分>。



## 输出结果

```shell
#compute eer
awk '{print $3}' exp/trials_out | paste - $trials | awk '{print $1, $4}' | compute-eer -
```

