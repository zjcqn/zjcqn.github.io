---
title: Kaldi实践 工程文件目录结构
author: 陈钱牛
date: 2020-09-24 12:00:00 +0800
categories: [Implementation]
tags: [Kaldi, ASV, Study]
math: true
layout: post
---

## data的文件目录结构

```shell
data 
├── local
│   ├── dev
│   ├── test
│   └── train
├── test
│   ├── aishell_speaker_ver.lst
│   ├── conf
│   ├── enroll
│   ├── eval
│   ├── feats.scp
│   ├── frame_shift
│   ├── spk2utt
│   ├── split10
│   ├── text
│   ├── utt2dur
│   ├── utt2num_frames
│   ├── utt2spk
│   ├── vad.scp
│   └── wav.scp
└── train
    ├── conf
    ├── feats.scp
    ├── frame_shift
    ├── spk2utt
    ├── split10
    ├── split12
    ├── text
    ├── utt2dur
    ├── utt2num_frames
    ├── utt2spk
    ├── vad.scp
    └── wav.scp
```

所有的解压语料包产生的原始数据记录的链接信息都存放在``data/local``下

所有信息被分成train和test两部分以后，链接信息被分别存放在``data/train``和``data/test``下，两部分下的目录结构基本一致（除了test部分做测试时产生的个别文件不同），常规的也不用赘述，只看一部分即可，以``data/train``为例。

```shell
└── train
    ├── conf		
    ├── feats.scp		#<utt> <raw_mfcc_addr>
    ├── frame_shift
    ├── spk2utt
    ├── split10			#存放10批次运算时的相关table关系，内部结构与./data/train类似
    ├── split12			#存放12批次运算时的相关table关系
    ├── text
    ├── utt2dur			#<utt> <during time>语句对应的时间
    ├── utt2num_frames	#<utt> <num of frames>语句对应的帧框数，和时间基本成正比
    ├── utt2spk			
    ├── vad.scp			#<utt> <vad_mfcc>
    └── wav.scp
```

## exp的文件目录

```shell
|-- diag_ubm_1024			#files about diag_UBM
|-- extractor_1024			#files about ivector extractor
|-- full_ubm_1024			#files about full_UBM
|-- ivector_enroll_1024		#ivector files
|-- ivector_eval_1024
|-- ivector_train_1024		
|-- make_mfcc				#mfcc log files
-- trials_out				#the result of plda_score
```
