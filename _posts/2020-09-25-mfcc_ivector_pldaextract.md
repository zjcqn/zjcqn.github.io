---
title: Kaldi | Kaldi 提取MFCC、IVECTOR和PLDA得分
author: 陈钱牛
date: 2020-09-25 12:33:00 +0800
categories: [Study,SRS,Kaldi]
tags: [Kaldi]
math: true
---


需要输出

- 对应音频的mfcc
- 从测试集数据集到plda得分的全过程步骤
- plda score

# 提取MFCC

```shell
cd $mfccdir
```

``mfccdir`` 目录下有8类文件：

- raw_mfcc_test.n.ark	raw_mfcc_train.n.ark
- raw_mfcc_test.n.scp	raw_mfcc_train.n.scp
- vad_mfcc_test.n.ark	vad_mfcc_train.n.ark
- vad_mfcc_test.n.scp	vad_mfcc_train.n.scp

显然，ark中存放音频，scp中存放对应关系。与raw相比，vad对mfcc进行了有效语音的检测，取哪个作为待修改的样本有待考量通过以下命令可以将.ark文件转化为txt文件，供外部调用。

```shell
copy-feats ark:foo.ark ark,t:foo.txt
```

在 `` data/train/feats.scp``中存有语音和存有mfcc.ark文件中首字节数一一对应起来（但是我们不知道.ark二进制文件的存放格式，所以这个办法有待考证）在``data/utt2num_frames``中存有语音对应的帧数，也就是对应的mfcc的个数，可以通过较为复杂的文本运算来提取mfcc。

幸运的是python模块``kaldiio``为我们提供了极为方便的API来提取ark文件中的数据并直接转换成numpy数组。

使用``kaldiio``提取mfcc的方式如下：

```python
from kaldiio import ReadHelper
import numpy
train_feats_scp_path="scp:/home/cqn/kaldi/egs/aishell/v1demo/data/train/feats.scp"
# eval_feats_scp_path="scp:/home/cqn/kaldi/egs/aishell/v1demo/data/test/eval/feats.scp"
# enroll_feats_scp_path="scp:/home/cqn/kaldi/egs/aishell/v1demo/data/test/enroll/feats.scp"
def get_utt2mfcc(feats_scp_path):
    utt2mfcc={}
    with ReadHelper(feats_scp_path) as reader:
        for key, numpy_array in reader:
            utt2mfcc[key]=numpy_array
    return utt2mfcc

utt2mfcc=get_utt2mfcc(feats_scp_path)
```

通过函数``get_utt2mfcc``最后得到的是一个 <UTT> : <MFCC Maxtrix>的字典。<MFCC Maxtrix>是一个帧数*mfcc维数的矩阵。

某一段音频的mfcc与ark中mfcc数据的对应关系存放在 `` data/train/feats. scp``和 `` data/train/feats.scp``中，

提取train的mfcc耗时很久，资源占用也很多，但是在攻击中并没有用到train的mfcc，使用的是测试集的小量mfcc。

# 提取ivector

同理，获取ivector的方法如下代码块，最终得到一个400维的ivector向量。值得注意的是ivector分为句子级的ivector和说话人级的ivector，这里两个都可以提取，操作上的区别在于.scp文件名称略有区别，在``spk_ivector.scp``中，``.ark``的索引地址使用的是相对路径，所以代码要放在kaldi工程工作路径下运行才行。

```python
ivector_path="scp:/home/cqn/kaldi/egs/aishell/v1demo/exp/ivector_enroll_1024/spk_ivector.scp"

def get_utt2ivector(ivector_path):
    utt2ivec={}
    with ReadHelper(ivector_path) as reader:
        for key, numpy_array in reader:
            utt2ivec[key]=numpy_array
    return utt2ivec
utt2ivec=get_utt2ivector(ivector_path)
print(utt2ivec['BAC009S0764W0149'])
```

# 提取PLDA打分

```shell
#compute plda score
trials=data/test/aishell_speaker_ver.lst
local/produce_trials.py data/test/eval/utt2spk $trials
ivector-plda-scoring --num-utts=ark:exp/ivector_enroll_1024/num_utts.ark \
	exp/ivector_train_1024/plda \
	ark:exp/ivector_enroll_1024/spk_ivector.ark \
	"ark:ivector-normalize-length scp:exp/ivector_eval_1024/ivector.scp ark:- |" \
	"cat '$trials' | awk '{print \\\$2, \\\$1}' |" exp/trials_out
```

PLDA得分保存在trials_out文件中，每一句话都和每一位说话人进行匹配打分。保存格式如下：

```text
S0764 BAC009S0764W0276 21.00846
S0765 BAC009S0764W0276 -49.95869
S0766 BAC009S0764W0276 -31.04892
S0767 BAC009S0764W0276 -53.95539
S0768 BAC009S0764W0276 -69.1451
S0769 BAC009S0764W0276 -61.76085
S0770 BAC009S0764W0276 -54.75211
S0901 BAC009S0764W0276 -173.1366
S0902 BAC009S0764W0276 -113.1527
S0903 BAC009S0764W0276 -125.7623
S0904 BAC009S0764W0276 -121.6964
S0905 BAC009S0764W0276 -106.0765
S0906 BAC009S0764W0276 -116.6665
S0907 BAC009S0764W0276 -124.6596
S0908 BAC009S0764W0276 -147.4091
```

通过以下命令可以得到更加完整的列表

```shell
paste -d " " ./exp/trials_out ./data/test/aishell_speaker_ver.lst| awk '{print $1,$2,$3,$6}' |sed 's/nontarget/0/'|sed 's/target/1/' > foo
```

``foo``保存的格式如下，方便python处理

```text
S0764 BAC009S0764W0276 21.00846 1 
S0765 BAC009S0764W0276 -49.95869 0
S0766 BAC009S0764W0276 -31.04892 0
S0767 BAC009S0764W0276 -53.95539 0
S0768 BAC009S0764W0276 -69.1451 0
S0769 BAC009S0764W0276 -61.76085 0
S0770 BAC009S0764W0276 -54.75211 0
S0901 BAC009S0764W0276 -173.1366 0
S0902 BAC009S0764W0276 -113.1527 0
S0903 BAC009S0764W0276 -125.7623 0
S0904 BAC009S0764W0276 -121.6964 0
S0905 BAC009S0764W0276 -106.0765 0
S0906 BAC009S0764W0276 -116.6665 0
```







