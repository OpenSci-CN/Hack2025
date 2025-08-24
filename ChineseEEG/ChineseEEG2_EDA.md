# COSN-Hackathon2025
COSN-Hackathon2025: ChineseEEG-2

## 背景介绍
用于神经解码的脑电图（EEG）研究需要大规模的基准数据集，特别是能将大脑活动与大型语言模型（LLM）语义表征对齐的、跨越口语、听力和阅读模式的成对大脑-语言数据，然而这类数据集在非英语语言中极为罕见。为此，我们推出了ChineseEEG-2，一个高密度脑电图数据集，旨在为真实的中文语言任务中的神经解码模型提供基准，它在前作ChineseEEG默读数据集的基础上，增加了朗读和被动听力两种模式，通过同步采集四名参与者朗读时的脑电和语音，并将语音播放给另外八名参与者以获取听力脑电数据，实现了朗读和被动听力模式之间精确的时间和语义对齐。该数据集包含脑电信号、语音音频、来自预训练语言模型的对齐语义嵌入以及任务标签，与ChineseEEG相结合，共同支持跨越口语、听力和阅读的联合语义对齐学习，不仅能够为神经解码算法提供基准测试，也推动了中文多模式语言任务中大脑与大型语言模型的对齐，为下一代神经语义解码提供了重要资源。[1]

## 数据描述
该数据集严格遵循脑电图数据结构（EEG-BIDS）规范，该规范是标准BIDS框架在脑电图领域的扩展。数据集的根目录下，Reading aloud 和 Passive Listening 文件夹分别存放朗读任务和听力任务的脑电数据，每个参与者的数据都封装在一个**.zip压缩包中，内含标准的脑电文件格式（.vhdr**、.vmrk和**.eeg**）。脑电数据根据语义材料（《小王子》和《加内特之梦》）被组织到ses-littleprince和ses-garnettdream两个子文件夹中，并按章节划分，以便于处理和分析。为了便于分析，脑电记录被分段并编码为runXY（其中X表示运行序号，Y表示该运行中的章节序号）。此外，数据集的derivatives文件夹包含与原始数据结构一致的已处理脑电数据和滤波结果，其中filtered子文件夹存放了经过1-40 Hz带通滤波的数据，可用于分析与语言任务相关的不同神经振荡活动。preprocessed子文件夹包含经过预处理流程的脑电数据。materials&embeddings文件夹位于朗读任务文件夹内，存储实验中使用的文本刺激和生成的嵌入，原始小说以**.txt格式保存，而音频和文本材料的嵌入则以.npy**格式保存。整个数据集的组织方式严格遵循EEG-BIDS指南，以确保其可重复性、透明度，便于其他研究人员使用，并为基于脑电图的语言研究提供了坚实的基础。[1]

## 数据EDA
### 脑电EDA的必要性
**一句话要点：** EEG 的信噪比低、伪影多、个体差异大，任何建模或统计前不做数据探索（EDA），等同于把噪声当信号、把偶然当规律。  
> - 数据质量与风险控制  
> 基础完整性：采样率、通道数/名称、参考方式、缺失段（NaN/Inf）、饱和/截幅、时间轴是否连续。  
> 环境噪声：50/60 Hz 线噪、设备接地问题；EDA 能及早决定是否做陷波与重新参考。  
> 漂移与稳定性：长时趋势、温度/电极阻抗变化造成的低频抬升。  
> - 参数设定的“依据”  
> 滤波/分段：看到原始 PSD 后再定高通/低通与陷波频率；看事件锁定曲线再定 tmin/tmax 与基线窗。  
> 功率与试次数：试次数、每类试次的均衡性直接决定统计功效（power）与模型泛化。  
> - 伪影识别与可解释性  
> 瞬目/眼动（低频、幅度大、前额主导）、肌电（高频、局部骤变）、线噪（窄带峰）；  
> ICA 级别：哪几个成分最“跟着事件动”，哪些是伪影成分（便于剔除/回归）；  
> 没有这一步，很容易把眨眼当“激活”、把咀嚼当“语义处理”。  
> - 任务范式与事件对齐校验  
> onset/时长分布、抖动是否合理；0 s 附近是否有触发器伪影；  
> ERP 是否在预期时间窗出现（如 N1/P2 的 0–200 ms、语义相关的 300–500 ms）。  
> 这一步能及早暴露标注错位或音频-EEG 时钟偏移。  
> - 建模与评估的防“踩坑”  
> 类别不平衡、泄漏风险（例如把相邻时间窗同时放进训练与测试）；  
> 选择合适的指标（AUC、F1、balanced accuracy）、合适的划分（by trial / by block / by subject）。  
> - 可复现与审稿友好  
> 输出QC 报告（通道/成分保留率、线噪评分、试次剔除率、SNR 分布）；  
> 让滤波参数、基线窗、阈值都有“数据依据”，而不是拍脑袋。  

**选取以下数据进行EDA**   
ChineseEEG-2/PassiveListening/derivatives/preprocessed/sub-01.zip[2]  
sub-01 ses-littleprince task-lis run-110 ica_components.npy  
SIZE: 21.32 MB  
Trigger如下：  
<img width="818" height="213" alt="Trigger" src="https://github.com/user-attachments/assets/d53a2c42-2291-4c1b-8728-b9b20bcb16ed" />

### ERP_mean

<img width="1200" height="540" alt="erp_mean_ROWS" src="https://github.com/user-attachments/assets/e6d889d0-d747-44a5-ba3c-496ab358cb18" />
<img width="1200" height="540" alt="erp_mean_ROWE" src="https://github.com/user-attachments/assets/7e598e70-490b-4824-94a7-fdbc91539ba0" />

**Heatmap**
<img width="1200" height="750" alt="erp_heatmap_ROWS" src="https://github.com/user-attachments/assets/d96e39e8-2e38-4bd6-9e14-1b3f939ddf19" />
<img width="1200" height="750" alt="erp_heatmap_ROWE" src="https://github.com/user-attachments/assets/9a56e084-eb1f-4d01-b942-f64fc43eda5a" />

**ERP_top_evoked_abs_mean**
<img width="1200" height="540" alt="ERP_ROWS_top_evoked_abs_mean_comp6_with_trials" src="https://github.com/user-attachments/assets/c5d28a11-0529-4034-ac40-7db213eee48e" />
<img width="1200" height="540" alt="ERP_ROWE_top_evoked_abs_mean_comp6_with_trials" src="https://github.com/user-attachments/assets/5034cd4c-8653-44ae-9ff1-b631fa77811f" />

**ERP_top_snr_with_trials**
<img width="1200" height="540" alt="ERP_ROWS_top_snr_comp6_with_trials" src="https://github.com/user-attachments/assets/d0a87c8e-10e1-41c0-9b96-4335bc7f674c" />
<img width="1200" height="540" alt="ERP_ROWE_top_snr_comp6_with_trials" src="https://github.com/user-attachments/assets/2da2ddca-dc94-4510-8a33-67dd78cfcd19" />


## 脑电数据可视化
TODO

## References
[1] Chen, S., Li, B., He, C., Li, D., Wu, M., Shen, X., Wang, S., Wei, X., Wang, X., Wu, H., & Liu, Q. (2025). ChineseEEG-2: An EEG Dataset for Multimodal Semantic Alignment and Neural Decoding during Reading and Listening (No. arXiv:2508.04240). arXiv. https://doi.org/10.48550/arXiv.2508.04240  
[2] Sitong Chen, Beiqianyi Li, Cuilin He, Dongyang Li, Mingyang Wu, Xinke Shen, Wang, S., Xuetao Wei, Xindi Wang, Haiyan Wu, & Quanying Liu. (2025). ChineseEEG-2:An EEG Dataset for Multimodal Semantic Alignment and Neural Decoding during Reading and ListeningChineseEEG-2:An EEG Dataset for Multimodal Semantic Alignment and Neural Decoding during Reading and Listening (Version V1, p. 110037613972 bytes, 61 files) [Dataset]. Science Data Bank. https://doi.org/10.57760/SCIENCEDB.20611  

