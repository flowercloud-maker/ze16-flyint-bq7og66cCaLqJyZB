**阅读目录**

* [一、思维导图展示](#_label0)
* [二、OpenCompass简介](#_label1)
* [三、OpenCompass安装](#_label2)
* [四、OpenCompass在线测评(可选)](#_label3)
* [五、加载本地测试数据集](#_label4)
* [六、配置本地Qwen模型](#_label5)
* [七、编写本地测评脚本](#_label6)
* [八、启动本地测评](#_label7)
* [九、测评参数解释](#_label8)
* [十、测评结果](#_label9)

### 一、思维导图展示


　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119173341727-826355175.png)


 


### 二、OpenCompass简介


　　OpenCompass是一个大模型测评体系，开源、高效。同时集成CompassKit测评工具、CompassHub测评集社区，CompassRank测评榜单。


　　官网地址：https://opencompass.org.cn/home


:[veee加速器](https://liuyunzhuge.com)### 三、OpenCompass安装


　　3\.1 创建虚拟环境




```
conda create --name opencompass python=3.10 -y
conda activate opencompass
```


　　3\.2 通过pip安装OpenCompass




```
# 支持绝大多数数据集及模型
pip install -U opencompass

# 完整安装（支持更多数据集）
# pip install "opencompass[full]"

# 模型推理后端，由于这些推理后端通常存在依赖冲突，建议使用不同的虚拟环境来管理它们。
# pip install "opencompass[lmdeploy]"
# pip install "opencompass[vllm]"

# API 测试（例如 OpenAI、Qwen）
# pip install "opencompass[api]"
```


　　3\.3 基于源码安装OpenCompass




```
git clone https://github.com/open-compass/opencompass opencompass
cd opencompass
pip install -e .
# pip install -e ".[full]"
# pip install -e ".[vllm]"
```


　　3\.4 下载系统数据集(可选)


　　　　因为我们使用自己下载的数据集，所以系统的数据集，不是必要的，但是为了原始程序的健壮性，还是推荐下载，因为我没有验证不下载的情况。




```
# 下载数据集到 data/ 处
wget https://github.com/open-compass/opencompass/releases/download/0.2.2.rc1/OpenCompassData-core-20240207.zip
# 解压之后是个文件夹叫data(后面会告诉这个dataf放在那里，先记住这个data文件夹)unzip OpenCompassData-core-20240207.zip
```


　　3\.5 使用ModelScope自动下载模型和数据(可选)


　　　　因为我们也是使用本地的模型，不需要程序中自己下载，如果做在线测试的，可以配置一下




```
pip install modelscope
export DATASET_SOURCE=ModelScope
```


　　3\.6 在线测评(可选)


　　　　至此如果，你具备FQ的条件的话，就可以直接进行在线测试了。


### 四、OpenCompass在线测评(可选)


　　因为在线测评很多模型是从huggingface上直接下载，然后测评的，需要FQ，我这里就不演示了，直接把官网上的测试过程拿过来展示，如果不需要在线测试的可以直接跳过。


　　4\.1 首次测评


　　　　OpenCompass 支持通过命令行界面 (CLI) 或 Python 脚本来设置配置。对于简单的评估设置，我们推荐使用 CLI；而对于更复杂的评估，则建议使用脚本方式。你可以在configs文件夹下找到更多脚本示例。




```
# 命令行界面 (CLI)
opencompass --models hf_internlm2_5_1_8b_chat --datasets demo_gsm8k_chat_gen

# Python 脚本
opencompass ./configs/eval_chat_demo.py
```


　　　　可以在 configs 文件夹下找到更多的脚本示例。


　　4\.2 API测评


　　　　OpenCompass 在设计上并不区分开源模型与 API 模型。您可以以相同的方式或甚至在同一设置中评估这两种类型的模型。




```
export OPENAI_API_KEY="YOUR_OPEN_API_KEY"
# 命令行界面 (CLI)
opencompass --models gpt_4o_2024_05_13 --datasets demo_gsm8k_chat_gen

# Python 脚本
opencompass  ./configs/eval_api_demo.py

# 现已支持 o1_mini_2024_09_12/o1_preview_2024_09_12  模型, 默认情况下 max_completion_tokens=8192.
```


　　4\.3 后端推理


　　　　如果您想使用除 HuggingFace 之外的推理后端来进行加速评估，比如 LMDeploy 或 vLLM，可以通过以下命令进行。




```
opencompass --models hf_internlm2_5_1_8b_chat --datasets demo_gsm8k_chat_gen -a lmdeploy
```


### 五、加载本地测试数据集


　　5\.1 通过 git 下载我们要使用的LawBench数据集到本地




```
git clone https://gitee.com/ljn20001229/LawBench.git　　
```


　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119162652464-2081438781.png)


　　说明：1处 OpenCompassData\-core\-20231110\.zip是通过git下载的数据集压缩包，需要将其解压到同级的标记2处的 data 中。


　　说明：3处 LawBench 是 OpenCompassData\-core\-20231110\.zip 解压后的文件。


　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119163505124-48289623.png)


　　至此我们自定义本地数据就就下载并放好了。


### 六、配置本地Qwen模型


　　6\.1 模型下载。直接在modelscope上下载即可。


![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119164404358-660499927.png)


 　　6\.2 将下载好的模型，加入到项目目录中


　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119165506604-937471823.png)


### 七、编写本地测评脚本


　　7\.1 在根目录的configs文件夹中创建 eval\_local\_qwen\_1\_8b\_chat.py 用作我们的Qwen1\.8B模型的测评启动脚本，代码如下：




```
 1 # eval_local_qwen_1_8b_chat.py
 2 
 3 from mmengine.config import read_base
 4 
 5 with read_base():
 6     # 导入数据集
 7     from .datasets.lawbench.lawbench_zero_shot_gen_002588 import lawbench_datasets as zero
 8     from .datasets.lawbench.lawbench_one_shot_gen_002588  import lawbench_datasets as one
 9     # 导入模型
10     from  opencompass.configs.models.qwen.local_qwen_1_8b_chat import models
11 datasets = [*zero, *one]
```


　　7\.2 修改 from .datasets.lawbench.lawbench\_zero\_shot\_gen\_002588 import lawbench\_datasets as zero 中的  lawbench\_zero\_shot\_gen\_002588 文件：


　　　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119170429487-823048361.png)


　　7\.3 同样修改 from .datasets.lawbench.lawbench\_one\_shot\_gen\_002588 import lawbench\_datasets as one 中的 lawbench\_one\_shot\_gen\_002588 文件


　　　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119170656479-126910921.png)


　　7\.4 创建  from  opencompass.configs.models.qwen.local\_qwen\_1\_8b\_chat import models 中的 qwen.local\_qwen\_1\_8b\_chat 文件


　　　　![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119171233794-1178506022.png)


### 八、启动本地测评


　　本地测评直接使用python run.py 执行我们创建的 configs/eval\_local\_qwen\_1\_8b\_chat.py 文件，在加上参数即可




```
 python run.py configs/eval_local_qwen_1_8b_chat.py  --debug
```


### 九、测评参数解释


* \-\-debug：调试模式，会有日志信息在控制台输出
* \-\-dry\-run：该次测试只加载数据集，但是不会再测评中使用。
* \-\-accelerator vllm：vllm加速，适用于本地部署大模型
* \-\-reuse：是否重用历史结果
* \-\-work\-dir：结果储存路径，默认是再outputs/default中
* \-\-max\-num\-worker：用于数据并行


### 十、测评结果


![](https://img2024.cnblogs.com/blog/751754/202411/751754-20241119172618493-1381197172.png)


 　　至此使用OpenCompass通过本地数据集LawBench测评本地模型Qwen1\.8B\_chat模型记录完毕，谢谢各位看官姥爷，看了这么久！笔芯！！！


