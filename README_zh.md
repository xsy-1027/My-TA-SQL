# Before Generation, Align it! A Novel and Effective Strategy for Mitigating Hallucinations in Text-to-SQL Generation

[![Data Link](https://img.shields.io/badge/BIRD-benchmark-green.svg)](https://bird-bench.github.io/)
[![Leaderboard](https://img.shields.io/badge/SPIDER-benchmark-pink.svg)](https://yale-lily.github.io/spider)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/release/python-380/)
[![OpenAI 0.27.6](https://img.shields.io/badge/OpenAI-0.27.6-orange.svg)](https://pypi.org/project/openai/)


这是论文 "生成前先对齐！一种新颖有效的减少文本到SQL生成中幻觉现象的策略" 的官方代码库，该论文已被2024年ACL Findings接收。 ["Before Generation, Align it! A Novel and Effective Strategy for Mitigating Hallucinations in Text-to-SQL Generation"](http://arxiv.org/abs/2405.15307)。

## 概述

在这项工作中，我们首先研究并得出当前文本到SQL框架中出现的幻觉现象，并将其归类为两大类：基于**模式的幻觉**和**基于逻辑**的幻觉。

<img src="./assets/hallucinations.png" align="middle" width="95%"> <br />

然后我们介绍了一种新策略，**任务对齐（TA）**，旨在在每个阶段减少幻觉现象。我们进一步提出了一个名为**TA-SQL的文本到SQL框架**，它由**任务对齐的模式链接（TASL）模块**和**任务对齐的逻辑合成（TALOG）模块**组成。这个代码库包含了使用GPT-4作为后端，在BIRD开发集上实现和评估TA-SQL的所有代码，如我们论文中所述。

<img src="./assets/main_figure.png" align="middle" width="95%">

## 环境搭建

• 使用以下命令配置本地环境:

   ```bash
    $ conda create -n tasql python=3.8.16
    $ conda activate tasql
    $ pip3 install -r requirements.txt
   ```

• 为Azure OpenAI API设置环境变量或在`./src/llm.py`中修改您自己的OpenAI配置
   ```bash
   export OPENAI_API_BASE="YOUR_OPENAI_API_BASE"
   export OPENAI_API_VERSION="YOUR_OPENAI_API_VERSION"
   export OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
   ```

## 数据准备

本文中使用的BIRD开发集可以直接从BIRD排行榜:[BIRD Leaderboard](https://bird-bench.github.io/) 下载。下载成功后，请将解压后的文件夹放在./data/下。./data/dev_databases/下的数据库集应包含以下资源：
- `database`: Each database folder should contain 
  - `database_description`: The csv files are manufactured to describe database schema and its values for models to explore or references.
  - `sqlite`: The database contents in BIRD.
- `dev_tables.json`: The file contains related information for each database, including `db_id`, `table_names_originial`, etc,. 
- `dev.json`: The file contains text-to-SQL paired with the oracle knowledge evidence. 

## 收集结果

要运行此项目，您可以直接执行命令行，按照说明（您可能需要根据个人喜好调整参数和路径）：
   ```bash
    $ sh ./run.sh
   ```
在此脚本中，如附录A.1所述，我们首先为每个列生成简洁的描述，作为显示数据库模式信息的组件。然后，我们运行TA-SQL工作流程，为每个自然语言问题生成SQL。这两个步骤的输出存储在`./outputs/`中。

### column_meaning.json
`./outputs/column_meaning.json`存储了一个Python字典，其中键由`database_id|table_name|column_name`组成，值是从原始CSV中总结的每个列及其值的关键信息。您可以直接使用此文件，或者通过运行`./src/conclude_meaning.py`生成简洁描述。

为了保持相同的评估设置，我们还为训练集（进行中）和测试集生成了`column_meaning.json`。如果您想在您的BIRD提交中使用它，请在提交邮件中提及。

## 评估
要运行评估，您需要先将真实的SQL文件`dev_gold.sql`放在`./data/`中。然后您可以使用以下命令行评估结果：

   ```bash
    $ sh ./run_evaluation.sh
   ```

## BIRD开发集上的EX性能
| MODEL            | SIM.  | MOD.  | CHALL. | TOTAL |
|------------------|-------|-------|--------|-------|
| **Closed-Source LLM**  |       |       |        |       |
| GPT4             | 54.35 | 34.64 | 31.70  | 46.35 |
| **+TA-SQL**      | **63.14** | **48.60** | **36.11**  | **56.19** |
| GPT4-turbo       | 59.35 | 38.92 | 27.78  | 50.19 |
| **+TA-SQL**      | **60.54** | **40.86** | **38.19**  | **52.48** |
| Claude           | 51.34 | 30.07 | 23.24  | 42.47 |
| **+TA-SQL**      | **56.97** | **39.78** | **27.78**  | **48.89** |
| ChatGPT          | 47.60 | 22.44 | 18.31  | 37.22 |
| **+TA-SQL**      | **51.57** | **33.76** | **25.69**  | **43.74** |
| **Open-Source weaker LLM** |       |       |        |       |
| DeepSeek         | 51.68 | 29.03 | 18.06  | 41.66 |
| **+TA-SQL**      | **53.41** | **32.04** | **19.44**  | **43.74** |
| CodeLlama        | 34.81 | 15.48 | 11.11  | 26.73 |
| **+TA-SQL**      | **37.30** | **13.33** | **11.11**  | **27.57** |

## BIRD Mini-Dev上的性能
为了促进高效且成本效益的开发周期，我们提供了一个名为**Mini-Dev**的BIRD开发数据集的轻量级版本。这个数据集是根据社区反馈编制的，包含了从原始开发数据集中派生的500对高质量的文本到SQL配对。为了进一步提高BIRD系统的实用性，我们不仅在SQLite中，还在MySQL和PostgreSQL中提供了Mini-Dev数据集。我们还为Mini-Dev数据集引入了两个新的评估指标：基于奖励的有效性得分（R-VES）和软F1得分，以呈现更全面和稳定的性能评估。关于Mini-Dev的更多细节可以在Mini-Dev介绍页面找到。

这里，我们展示了TA在Mini-Dev上的性能
| MODEL            | EX  | R-VES  | Soft F1 |
|------------------|-------|-------|--------|
| **Closed-Source LLM**  |       |       |        
| GPT4-turbo       | 45.80 | 44.79 | 50.08  | 
| **+TA-SQL**      | **58.00** | **56.44** | **62.40**  |        
| GPT35-turbo      | 38.00 | 37.33 | 41.84  | 
| **+TA-SQL**      | **41.60** | **40.59** | **44.25**  | 
| **Open-Source LLM** |       |       |           
| Llama3-70b        | 40.08 | 39.02 | 44.38  | 
| **+TA-SQL**      | **42.80** | **41.37** | **46.66**  | 

## 项目结构

```txt
├─data/
|  ├─dev_databases # data of BIRD dev databases
├─src/
|  ├─conclude_meaning.py  # Generate suffcient descriptions for columns in the dataset
|  ├─llm.py               # OpenAI API call
|  ├─modules.py           # Modules of TA-SQL framework
|  ├─prompt_bank.py       # prompt templates used in each module of TA-SQL framework
|  ├─utils.py             # utils functions
├─evaluation/
|  ├─evaluation.py        # EX evaluation script
|  ├─evaluation_ves.py    # VES evaluation script
├─outputs/
├─README.md
├─requirements.txt
├─run_evaluation.sh
├─run.py 
├─run.sh 
```

## 引用
如果您认为我们的工作对您有帮助，请引用此代码库。
```text
@article{qu2024before,
  title={Before Generation, Align it! A Novel and Effective Strategy for Mitigating Hallucinations in Text-to-SQL Generation},
  author={Qu, Ge and Li, Jinyang and Li, Bowen and Qin, Bowen and Huo, Nan and Ma, Chenhao and Cheng, Reynold},
  journal={arXiv preprint arXiv:2405.15307},
  year={2024}
}
```



