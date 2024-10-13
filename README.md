> 初步构想，有待完善...

## Description
🤖「能不能好好说话？」拼音首字母缩写查词翻译工具

https://github.com/itorr/nbnhhsh

## Sample
```
[{
	'original': 'DL是ML的一个分支。',
	'translation': ['深度学习是机器学习的一个分支。', 'deadline是机器学习的一个分支。'],
	'dictionary': [{
		'word': 'dl',
		'definition': ['大佬', '毒瘤', '决斗链接', '刀郎', '邓伦', '懂了', '代练', '消逝的光芒', '达赖', '多练', 'dancing line', '毒龙', 'Dua Lipa', '顶流', 'download', 'dlsite', '大龙', '对立', '跳舞的线', '代聊(论坛)', '大林', '丁磊', 'deadline', '深度学习', '独立', '朵拉', 'doctor love （明日方舟）', '地理', '汪东城x炎亚纶', '呆驴', '下载']
	}, {
		'word': 'ml',
		'definition': ['Make Love', '马鹿', '摸了', '梅林固件', '偶像大师百万现场', 'M League', '码了', '马龙', '机器学习', 'Machine Learning', 'master love', '梅莉（东方Project人物）', '马列（马克思列宁）', "Mao's legacy（游戏名称）", '梅林传奇', 'Making Lovers(游戏名称)', '毫升（单位）', '米兰', '免流', '麻了', '马丽(演员)', '马琳', '米洛（up主）', '冥龙（光遇）', 'Margin Left(HTML)']
	}]
}]
```

## Note

> 删除了两年前（2022年）错漏百出的拙劣实现，待合适时机再考虑重写一个demo。
> 原有思路：借助语言模型的惊异度和困惑度指标，为候选词列表排序，并选择信息量最低的高确定性选项。
> AI发展迅速，语言模型的能力自ChatGPT发布以来已显著增强。为该设想提供更强支撑的同时，甚至能辅助完成这个设想的具体实现。
