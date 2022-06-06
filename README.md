> 待完善...

# Description
🤖「能不能好好说话？」辅以语言模型的构想 https://github.com/itorr/nbnhhsh

# Sample
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

# Demo
加载语言模型：
```
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

def load_model(path):
    tokenizer = AutoTokenizer.from_pretrained(path)
    model = AutoModelForCausalLM.from_pretrained(path).to(device)
    model.eval()
    return tokenizer, model

tokenizer, model = load_model("uer/gpt2-chinese-cluecorpussmall")
```
加载相关函数：
```
import math, requests, json, re, itertools, time

def query(text):
    # 提交到nbnhhsh项目查询，提交内容仅包含字母及数字序列，剔除顺序、大小写等信息，保证隐私安全
    query = ",".join(list(set(re.findall("[A-Za-z0-9]{2,10}", text)))).lower()
    if not query: return {}
    response = requests.post("https://lab.magiconch.com/api/nbnhhsh/guess", headers={"Content-Type":"application/json"}, data=json.dumps({"text":query}))
    results = json.loads(response.text)
    dictionary = {}
    for data in results:
        if "trans" in data: # 释义（用户上传）
            dictionary[data["name"]] = data["trans"]
        elif "inputting" in data: # 猜测（程序推测）（无用户提交数据时，会作为拼音首字母、词组首字母查询）
            dictionary[data["name"]] = data["inputting"]
    return dictionary

def get_perplexity_score(sentence):
    tokenize_input = tokenizer.tokenize(sentence)
    tensor_input = torch.tensor([tokenizer.convert_tokens_to_ids(tokenize_input)]).to(device)
    if tensor_input.size()[1] <= 0: return None
    predictions = model(tensor_input, labels=tensor_input)
    loss = predictions[0]
    return math.exp(loss)

def get_perplexity_ranking(sentences):
    result = [[sentence, get_perplexity_score(sentence)] for sentence in sentences]
    result.sort(key=lambda data: data[1])
    return result

def get_translation_data(text, dictionary):
    # 翻译标记数据，例：[[DL,深度学习,下载,地理],是,[ML,机器学习,毫升],的一个分支]
    data = [str(text)]
    for key in reversed(sorted(dictionary.keys(), key=len)):
        if len(dictionary[key]) < 1: continue
        definition = dictionary[key]
        definition = [re.sub("[\(（].*?[\)）]", "", _) for _ in definition] # 清洗词典释义
        definition = [key] + definition
        index = 0
        while index < len(data):
            if not isinstance(data[index], str): continue # 忽略已标记条目
            text = data.pop(index)
            texts = text.lower().split(key.lower()) # 忽略大小写
            dat = list(itertools.chain.from_iterable([_, definition] for _ in texts))[0:-1]
            for i in range(len(dat)): data.insert(index + i, dat[i])
            index += len(dat)
            index += 1
    return data

def get_translation_sentences(data): # 根据翻译标记数据生成所有组合
    data = [([_] if isinstance(_, str) else _[1:]) for _ in data]
    sentences = ["".join(_) for _ in itertools.product(*data)]
    return sentences

def get_estimated_time(sentences): # 推理一次以保守估计整体耗时
    test = list(sorted(sentences, key=len))[-1]
    start = time.perf_counter()
    get_perplexity_ranking([test])
    return (time.perf_counter() - start) * len(sentences)

def translate(text): # 翻译接口函数，返回格式化结果
    if text == "" or not isinstance(text, str): return []
    dictionary = query(text)
    regex = re.compile("[\n。！？；]+")
    sents = regex.split(text)
    puncs = regex.findall(text)
    puncs.append("")
    texts = list(map(lambda sent, punc: sent + punc, sents, puncs))
    texts = [text for text in texts if text]
    # 预估耗时并作限制
    sents = [get_translation_sentences(get_translation_data(texts[index], dictionary)) for index in range(len(texts))]
    etime = get_estimated_time(sum(sents, []))
    if etime > 60: return [[{ "original": text, "translation": "难以计算，请缩短输入或升级机器。（当前设备：" + str(device) + "，预计耗时：" + str(round(etime)) + "s）", "dictionary": dictionary }]]
    # ---------------
    results = []
    for index in range(len(texts)):
        text = texts[index]
        data = get_translation_data(text, dictionary)
        sentences = get_translation_sentences(data)
        translation = []
        ranking = get_perplexity_ranking(sentences)
        maximum = 3 # 最大输出数量
        score = min(ranking[0][1] + 50, ranking[0][1] * 2) # 困惑度阈值，以最优选为基准计算得出
        for i in range(len(ranking)):
            if i >= maximum or ranking[i][1] > score: break
            translation.append(ranking[i][0])
        dic = []
        for item in data:
            if isinstance(item, str): continue
            word = item[0]
            entry = { "word": word, "definition": dictionary[word] }
            if entry in dic: continue
            dic.append(entry)
        results.append({ "original": text, "translation": translation, "dictionary": dic })
    return results
```
运行测试：
```
print(translate('你要同我做AI吗？NLP还是CV？'))
print(translate('jmm冲啊这个nc真的无敌好喝到翘jiojio！'))
```
