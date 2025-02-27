LogCluster - A Data Clustering and Pattern Mining Algorithm for Event Logs

LogCluster - 事件日志的数据聚类和模式挖掘算法

Risto Vaarandi and Mauno Pihelgas

TUT 数字取证和网络安全中心
塔林科技大学
爱沙尼亚塔林
______

# 0 概要
Abstract—Modern IT systems often produce large volumes of event logs, and event pattern discovery is an important log management task. For this purpose, data mining methods have been suggested in many previous works. In this paper, we present the LogCluster algorithm which implements data clustering and line pattern mining for textual event logs. The paper also describes an open source implementation of LogCluster.

Keywords—event log analysis; mining patterns from event logs; event log clustering; data clustering; data mining

摘要——现代 IT 系统通常会产生大量的事件日志，而事件模式发现是一项重要的日志管理任务。 为此，在许多以前的工作中已经提出了数据挖掘方法。 在本文中，我们提出了 LogCluster 算法，该算法实现了文本事件日志的数据聚类和线型挖掘。 该论文还描述了 LogCluster 的开源实现。

关键词——事件日志分析； 从事件日志中挖掘模式； 事件日志集群； 数据聚类； 数据挖掘

# 1 介绍

During the last decade, data centers and computer networks have grown significantly in processing power, size, and complexity. As a result, organizations commonly have to handle many gigabytes of log data on a daily basis. For example, in our recent paper we have described a security log management system which receives nearly 100 million events each day [1]. In order to ease the management of log data, many research papers have suggested the use of data mining methods for discovering event patterns from event logs [2–20]. This knowledge can be employed for many different purposes like the development of event correlation rules [12–16], detection of system faults and network anomalies [6–9, 19], visualization of relevant event patterns [17, 18], identification and reporting of network traffic patterns [4, 20], and automated building of IDS alarm classifiers [5].

在过去十年中，数据中心和计算机网络在处理能力、规模和复杂性方面都显着增长。 因此，组织通常必须每天处理数 GB 的日志数据。 例如，在我们最近的论文中，我们描述了一个安全日志管理系统，它每天接收近 1 亿个事件 [1]。 为了简化日志数据的管理，许多研究论文建议使用数据挖掘方法从事件日志中发现事件模式[2-20]。 这些知识可以用于许多不同的目的，例如开发事件关联规则 [12-16]、检测系统故障和网络异常 [6-9、19]、可视化相关事件模式 [17、18]、识别和 报告网络流量模式 [4, 20]，以及自动构建 IDS 警报分类器 [5]。

In order to analyze large amounts of textual log data without well-defined structure, several data mining methods have been proposed in the past which focus on the detection of line patterns from textual event logs. Suggested algorithms have been mostly based on data clustering approaches [2, 6, 7, 8, 10, 11]. The algorithms assume that each event is described by a single line in the event log, and each line pattern represents a group of similar events. 

为了分析大量没有明确结构的文本日志数据，过去已经提出了几种数据挖掘方法，这些方法侧重于从文本事件日志中检测线条模式。 建议的算法主要基于数据聚类方法 [2, 6, 7, 8, 10, 11]。 算法假设每个事件在事件日志中由一行描述，每个行模式代表一组相似的事件。


In this paper, we propose a novel data clustering algorithm called LogCluster which discovers both frequently occurring line patterns and outlier events from textual event logs. The remainder of this paper is organized as follows – section II provides an overview of related work, section III presents the LogCluster algorithm, section IV describes the LogCluster prototype implementation and experiments for evaluating its performance, and section V concludes the paper.

在本文中，我们提出了一种新的数据聚类算法，称为 LogCluster，它可以从文本事件日志中发现频繁出现的线条模式和异常事件。 本文的其余部分组织如下——第二部分概述了相关工作，第三部分介绍了 LogCluster 算法，第四部分描述了 LogCluster 原型实现和评估其性能的实验，第五部分总结了本文。


# 2 相关工作
## 2.1 SLCT

One of the earliest event log clustering algorithms is SLCT that is designed for mining line patterns and outlier events from textual event logs [2]. During the clustering process, SLCT assigns event log lines that fit the same pattern (e.g., Interface * down) to the same cluster, and all detected clusters are reported to the user as line patterns. For finding clusters in log data, the user has to supply the support threshold value s to SLCT which defines the minimum number of lines in each cluster. SLCT begins the clustering with a pass over the input data set, in order to identify frequent words which occur at least in s lines (word delimiter is customizable and defaults to whitespace). Also, each word is considered with its position in the line. For example, if s=2 and the data set contains the lines

最早的事件日志聚类算法之一是 SLCT，它设计用于从文本事件日志中挖掘行模式和异常事件 [2]。 在聚类过程中，SLCT 将符合相同模式（例如，Interface * down）的事件日志行分配给同一个类簇，并将所有检测到的类簇作为行模式报告给用户。 为了在日志数据中查找类簇，用户必须向 SLCT 提供支持阈值 s，SLCT 定义了每个类簇中的最小行数。 SLCT 通过对输入数据集的传递开始聚类，以识别至少在 s 行中出现的频繁单词（单词分隔符是可定制的，默认为空格）。 此外，每个单词都考虑其在行中的位置。 例如，如果 s=2 并且数据集包含行

Interface eth0 down
Interface eth1 down
Interface eth2 up


then words (Interface,1) and (down,3) occur in three and two lines, respectively, and are thus identified as frequent words. SLCT will then make another pass over the data set and create cluster candidates. When a line is processed during the data pass, all frequent words from the line are joined into a set which will act as a candidate for this line. After the data pass, candidates generated for at least s lines are reported as clusters together with their supports (occurrence times). Outliers are identified during an optional data pass and written to a user-specified file. For example, if s=2 then two cluster candidates {(Interface,1), (down,3)} and {(Interface,1)} are detected with supports 2 and 1, respectively. Thus, {(Interface,1), (down,3)} is the only cluster and is reported to the user as a line pattern Interface * down (since there is no word associated with the second position, an asterisk is printed for denoting a wildcard).Reported cluster covers the first two lines, while the line Interface eth2 up is considered an outlier.

（1）那么单词 (Interface,1) 和 (down,3) 分别出现在三行和两行中，因此被识别为频繁词。
（2）然后，SLCT 将再次遍历数据集并创建候选类簇。当在数据传递期间处理一行时，该行中的所有频繁词都被加入到一个集合中，该集合将作为该行的候选词。
（3）数据通过后，为至少 s 行生成的候选与它们的支持（出现次数）一起报告为类簇。在可选数据传递期间识别异常值并将其写入用户指定的文件。例如，如果 s=2，则检测到两个候选簇 {(Interface,1), (down,3)} 和 {(Interface,1)}，分别支持 2 和 1。因此，{(Interface,1), (down,3)} 是唯一的簇，并以行模式 Interface * down 的形式报告给用户（因为没有与第二个位置关联的单词，所以打印一个星号表示通配符）。报告的类簇覆盖了前两行，而 Interface eth2 up 行被认为是异常值。

SLCT has several shortcomings which have been pointed out in some recent works. Firstly, it is not able to detect wildcards after the last word in a line pattern [11]. For instance, if s=3 for three example lines above, the cluster {(Interface,1)} is reported to the user as a line pattern Interface, although most users would prefer the pattern Interface * *. Secondly, since word positions are encoded into words, the algorithm is sensitive to shifts in word positions and delimiter noise [8]. For instance, the line Interface HQ Link down would not be assigned to the cluster Interface * down, but would rather generate a separate cluster candidate. Finally, low support thresholds can lead to overfitting when larger clusters are split and resulting patterns are too specific [2].


SLCT 有几个缺点，在最近的一些工作中已经指出。 首先，它无法检测行模式中最后一个单词之后的通配符[11]。 例如，如果上面三个示例行的 s=3，集群 {(Interface,1)} 将作为行模式 Interface 报告给用户，尽管大多数用户更喜欢模板 Interface * *。 其次，由于单词位置被编码为单词，因此该算法对单词位置的变化和分隔符噪声很敏感[8]。 例如，Line Interface HQ Link down 不会分配给类簇 Interface * down，而是生成一个单独的类簇候选。 最后，当更大的集群被分割并且结果模式过于具体时，低支持阈值会导致过度拟合[2]。

自动日志解析，分配符合**相同模式**的事件日志行到相同簇中，所有检测的簇都被作为行模式。用户需要提供支持阈值 s 以便SLCT定义每个簇中**最少行数量**。在日志处理期间，所有来自日志的频繁词放入集合作为此日志的候选。

1. 建立单词字典，对所有日志建立包含单词频率和坐标的字典；
2. 建立日志簇；
3. 生成日志模板。

## 2.2 SLCT的修改版本
Reidemeister, Jiang, Munawar and Ward [6, 7, 8] developed a methodology that addresses some of the above shortcomings. The methodology uses event log mining techniques for diagnosing recurrent faults in software systems. First, a modified version of SLCT is used for mining line patterns from labeled event logs. In order to handle clustering errors caused by shifts in word positions and delimiter noise, line patterns from SLCT are clustered with a single-linkage clustering algorithm which employs a variant of the Levenshtein distance function. After that, a common line pattern description is established for each cluster of line patterns. According to [8], single-linkage clustering and post-processing its results add minimal runtime overhead to the clustering by SLCT. The final results are converted into bit vectors and used for building decision-tree classifiers, in order to identify recurrent faults in future event logs.


Reidemeister、Jiang、Munawar 和 Ward [6, 7, 8] 开发了一种方法来解决上述一些缺点。 该方法使用事件日志挖掘技术来诊断软件系统中的经常性故障。 首先，SLCT 的修改版本用于从标记的事件日志中挖掘行模式。 为了处理由单词位置偏移和分隔符噪声引起的聚类错误，来自 SLCT 的线条模式使用单链接聚类算法进行聚类，该算法采用 Levenshtein 距离函数的变体。 之后，为每个线型集群建立一个共同的线型描述。 根据[8]，单链接聚类和后处理其结果为 SLCT 的聚类增加了最小的运行时开销。 最终结果被转换为位向量并用于构建决策树分类器，以识别未来事件日志中的重复故障。

## 2.3 IPLoM
Another clustering algorithm that mines line patterns from event logs is IPLoM by Makanju, Zincir-Heywood and Milios [10, 11]. Unlike SLCT, IPLoM is a hierarchical clustering algorithm which starts with the entire event log as a single partition, and splits partitions iteratively during three steps. Like SLCT, IPLoM considers words with their positions in event log lines, and is therefore sensitive to shifts in word positions. During the first step, the initial partition is split by assigning lines with the same number of words to the same partition. During the second step, each partition is divided further by identifying the word position with the least number of unique words, and splitting the partition by assigning lines with the same word to the same partition. During the third step, partitions are split based on associations between word pairs. At the final stage of the algorithm, a line pattern is derived for each partition. Due to its hierarchical nature, IPLoM does not need the support threshold, but takes several other parameters (such as partition support threshold and cluster goodness threshold) which impose fine-grained control over splitting of partitions [11]. As argued in [11], one advantage of IPLoM over SLCT is its ability to detect line patterns with wildcard tails (e.g., Interface * *), and the author has reported higher precision and recall for IPLoM.

另一种从事件日志中挖掘线型的聚类算法是 Makanju、Zincir-Heywood 和 Milios [10, 11] 的 IPLoM。与 SLCT 不同，IPLoM 是一种层次聚类算法，它从整个事件日志作为单个分区开始，并在三个步骤中迭代地拆分分区。与 SLCT 一样，IPLoM 考虑单词及其在事件日志行中的位置，因此对单词位置的变化很敏感。在第一步中，通过将具有相同字数的行分配给同一分区来拆分初始分区。在第二步中，通过识别具有最少唯一词数的词位置来进一步划分每个分区，并通过将具有相同单词的行分配给同一分区来划分分区。在第三步中，根据词对之间的关​​联划分分区。在算法的最后阶段，为每个分区导出一个线型。由于其分层性质，IPLoM 不需要支持阈值，而是采用其他几个参数（例如分区支持阈值和集群良好度阈值），这些参数对分区的拆分进行了细粒度控制 [11]。正如 [11] 中所述，IPLoM 优于 SLCT 的一个优势是它能够检测带有通配符尾部的线条模式（例如，接口 * *），并且作者报告了 IPLoM 的更高精度和召回率。


# 3 LogCluster 算法
$L = [l_1, l_2, ..., l_n]$是文本事件日志，由 n 行组成，每行 $l_i (1<=i<=n)$ 是事件的完全表征，i 是行唯一标识；

每行 $l_i$ 是 k 个词的序列，$l_i = (w_{i,1}, w_{i,2}, ..., w_{i,k_i})$。

LogCluster 使用支持阈值 $s(1<=s<=n)$ 作为输入参数，将日志划分到 $C_1, C_2, ..., C_m$ 簇中，每簇至少 s 条日志，O是离群簇。

LogCluster 将日志聚类问题视作模式挖掘问题，每簇 $C_j$ 通过模式 $p_j$ 标识为唯一的，类簇内所有行与之匹配。
为检测簇，LogCluster 从日志中挖掘模式 $p_j$。
模式 $p_j$ 和簇 $C_j$ 的支持值定义为 $C_j$ 中日志的数量，每种模式由词（wrods）和通配符（wildcards）组成。
例如：通配符$*\{1, 3\}$ 表示匹配1到3个单词。


## 3.1 构建频繁词
为找到达到支持阈值的模式，每种模式的所有词至少要发生在 s 条事件日志中。

LogCluster 考虑日志中的每个词但是**不包括位置信息**。$I_w$ 是包含单词 w 的行标识的集合。如果 $I_w$ 大于等于阈值 s ，则 w 是频繁词，所有频繁词的集合使用 F 表示。

LogCluster 使用一个 h 大小的框架计数器。在预先处理事件日志时，每条事件日志行的去重词散列到 0 到 h-1 的整数，增加对应的计数数量。(构建词表，统计词频)。

设想的实现方式：每行日志分词后，词汇去重，统计所有词汇的词频，超过阈值 s 的为频繁词

## 3.2 生成候选簇
频繁词集合构建后，LogCluster 产生簇的候选。
对事件日志中的每行，LogCluster 从日志提取所有频繁词，将词处理为元组，保留原始行中原始位置，元组会作为候选簇的标识，所在行会被归为对应的候选。

如果给定的候选不存在，则初始化并将支持计数设为1，从行中创建其行模式。如果候选存在，其支持计数增加，行模式调整以覆盖当前行。

Log Cluster不记录分配给候选簇的日志。

举例：事件日志“Interface DMZ-link down at node router2”，频繁词是“Interface, down, at, node”，该行被分配给识别的候选元组(Interface, down, at, node)。如果候选不存在，则设置行模式初始化为“Interface *{1,1} down at node *{1,1}”，计数设为 1，通配符 *{1，1}可匹配任何单一词汇。如果下一行产生同样的候选标识“Interface HQ link down at node router2”，候选支持计数增加到 2。行模式设置为“Interface *{1,2} down at node *{1,1}”，为使模式匹配，至少一个但不超过2个在 interface和down之间。

设想的实现方式：根据每条日志中的频繁词(保持其原有词序)，将有相同频繁词的日志归并，并提取对应的模式，提取后模式数量小于 s 的模式删除，之后即可获得模式提取结果。

```c
Procedure : Generate_Candidates
Input   : event log L = {l1, l2, l3, ..., ln}     事件日志
        set of frequent words F                 频繁词
Output  : set of cluster candidates X


X := Null
for (id = 1; id <= n; ++id) do
    tuple := ()
    vars := ()
    i := 0; v := 0
    for each w in (w_{id,1},…,w_{id,kid}) do
        if (w \in F) then
            tuple[i] := w           # 第i个频繁词
            vars[i] := v            # 第i个频繁词前有几个非频繁词
            ++i; v := 0
        else
            ++v
        fi
    done

    vars[i] := v
    k := # of elements in tuple
    if (k > 0) then
        if (\exists Y \in X, Y.tuple == tuple) then
            # 此行的频繁词列表已经存在候选簇中，则支持度加一
            ++Y.support
            for (i := 0; i < k+1; ++i) do
                if (Y.varmin[i] > vars[i]) then
                    Y.varmin[i] := vars[i]
                fi
                if (Y.varmax[i] < vars[i]) then
                    Y.varmax[i] := vars[i]
                fi
            done
        else
            # 初始化候选簇
            initialize new candidate Y
            Y.tuple := tuple
            Y.support := 1
            for (i := 0; i < k+1; ++i) do
                Y.varmin[i] := vars[i]
                Y.varmax[i] := vars[i]
            done
            X := X $\cup$ { Y }
        fi
        Y.pattern = ()
        j: = 0
        for (i := 0; i < k; ++i) do
            if (Y.varmax[i] > 0) then
                min := Y.varmin[i]
                max := Y.varmax[i]
                Y.pattern[j] := “*{min,max}”
                ++j
            fi
            Y.pattern[j] := tuple[i]
            ++j
        done

        if (Y.varmax[k] > 0) then
            min := Y.varmin[k]
            max := Y.varmax[k]
            Y.pattern[j] := "*{min,max}"
        fi
    fi
    done
    return X
```


## 3.3 优化方法

通过所有数据完成簇候选构建后，LogCluster 将所有支持计数小于支持阈值 s 的候选排除，保留剩余的。
当模式挖掘使用较小的支持阈值执行时，LogCluster 与 SLCT 相似，倾向于过拟合，即较大的簇可能会被划分为较小的簇，有过于详细的行模式。

### 3.3.1 Aggregate_support
When pattern mining is conducted with lower support threshold values, LogCluster is (similarly to SLCT) prone to overfitting – larger clusters might be split into smaller clusters with too specific line patterns. For example, the cluster with a pattern Interface *{1,1} down could be split into clusters with patterns Interface *{1,1} down, Interface eth1 down, and Interface eth2 down. Furthermore, meaningful generic patterns (e.g., Interface *{1,1} down) might disappear during cluster splitting. In order to address the overfitting problem, LogCluster employs two optional heuristics for increasing the support of more generic cluster candidates and for joining clusters. The first heuristic is called Aggregate_Supports and is applied after the candidate generation procedure has been completed, immediately before clusters are selected. The heuristic involves finding candidates with more specific line patterns for each candidate, and adding supports of such candidates to the support of the given candidate. For instance, if candidates User bob login from 10.1.1.1, User *{1,1} login from 10.1.1.1, and User *{1,1} login from *{1,1} have supports 5, 10, and 100, respectively, the support of the candidate User *{1,1} login from *{1,1} will be increased to 115. In other words, this heuristic allows clusters to overlap.

当以较低的支持阈值进行模式挖掘时，LogCluster（类似于 SLCT）容易过度拟合——较大的集群可能会被分割成具有过于具体的线条模式的较小集群。例如，模式 Interface *{1,1} down 的集群可以拆分为模式 Interface *{1,1} down、Interface eth1 down 和 Interface eth2 down 的集群。此外，有意义的通用模式（例如，Interface *{1,1} down）可能会在集群分裂期间消失。为了解决过拟合问题，LogCluster 采用了两种可选的启发式方法来增加对更通用集群候选者的支持和加入集群。第一个启发式称为 Aggregate_Supports 并在候选生成过程完成后立即在选择集群之前应用。启发式方法涉及为每个候选者找到具有更具体线型的候选者，并将这些候选者的支持添加到给定候选者的支持中。例如，如果候选人用户 bob 从 10.1.1.1 登录，用户 *{1,1} 从 10.1.1.1 登录，用户 *{1,1} 从 *{1,1} 登录，则支持 5、10 和 100 ，分别从*{1,1}登录候选用户*{1,1}的支持将增加到115。换句话说，这种启发式允许集群重叠。

第一种减少过拟合的启发式策略叫 Aggregate_Support ，在候选簇生成后，簇选择前使用。这种启发涉及发现对每种候选有更详细行模式的候选，增加在给定候选中的支持。此种模式可以重叠。

例如，三个候选簇
“User bob login from 10.1.1.1”,
“User *{1,1} login from 10.1.1.1”, and
“User *{1,1} login from *{1,1}”，
支持度分别为 5，10，100，候选簇 “User *{1,1} login from *{1,1}”的支持度可合并为 115；Aggregate_Support 允许簇重合。


### 3.3.2 Join_Cluster

The second heuristic is called Join_Clusters and is applied after clusters have been selected from candidates. For each frequent word $w \in F$, we define the set $C_w$ as follows: $\$C_w = {f | f \in F, I_w \cap I_f \neq \phi}\}$ (i.e., $C_w$ contains all frequent words that co-occur with w in event log lines). If $w’ \in C_w$ (i.e., w’ co-occurs with w), we define dependency from w to w’ as $dep(w, w’) = \frac {|I_w \cap I_{w’}|} {|Iw|}$. In other words, dep(w, w’) reflects how frequently w’ occurs in lines which contain w. Also, note that $0 < dep(w, w’) ≤ 1$. If $w_1,…,w_k$ are frequent words of a line pattern (i.e., the corresponding cluster is identified by the tuple ($w_1,…,w_k$)), the weight of the word wi in this pattern is calculated as follows: $weight(w_i) = \Sigma_{j=1}^k dep(w_j, w_i) / k$. Note that since dep(wi, wi) = 1, then 1/k ≤ weight(wi) ≤ 1. Intuitively, the weight of the word indicates how strongly correlated the word is with other words in the pattern. For example, suppose the line pattern is Daemon testd killed, and words Daemon and killed always appear together, while the word testd never occurs without Daemon and killed. Thus, weight(Daemon) and weight(killed) are both 1. Also, if only 2.5% of lines that contain both Daemon and killed also contain testd, then $weight(testd) = (1 + 0.025 + 0.025) / 3 = 0.35$. (We plan to implement more weight functions in the future versions of the LogCluster prototype.)

第二个启发式称为 Join_Clusters，在从候选中选择集群后应用。对于每个频繁词$w \in F$，我们定义集合$C_w$如下： 
$\{C_w = {f | f \in F, I_w \cap I_f \neq \phi}\}$
（即，$C_w$ 包含在事件日志行中与 w 共同出现的所有频繁词）。如果 $w' \in C_w$ （即 w' 与 w 共现），我们定义从 w 到 w' 的依赖关系为 
$dep(w, w') = \frac {|I_w \cap I_{w'} |} {|Iw|}$
。换句话说，dep(w, w') 反映了 w' 在包含 w 的行中出现的频率。另外，请注意 $0 < dep(w, w') ≤ 1$（不共现的不在考虑范围内）。如果$w_1,...,w_k$是一个行模式的频繁词（即对应的簇由元组($w_1,...,w_k$)标识），那么这个模式中词$w_i$的权重计算如下:
$weight(w_i) = \Sigma_{j=1}^k dep(w_j, w_i) / k$
。请注意，由于 $dep(w_i, w_i) = 1$，因此 $\frac 1 k ≤ weight(w_i) ≤ 1$。直观地说，单词的权重表示该单词与模式中其他单词的相关性有多强。例如，假设行模式是 `Daemon testd killed`，单词 Daemon 和 kill 总是一起出现，而单词 testd 永远不会在没有 Daemon 和 Killed 的情况下出现。因此，weight(Daemon) 和 weight(killed) 都是 1。此外，如果只有 2.5% 的行同时包含 Daemon 和 killed 也包含 testd，那么
$weight(testd) = (1 + 0.025 + 0.025) / 3 = 0.35$
。（我们计划在 LogCluster 原型的未来版本中实现更多的权重函数。）



总结：第二种启发称为 Join_Cluster，在簇已经从候选中选择后使用。$C_w$ 包含所有高频词共现的词汇。
$$
dep(w, w') = \frac {|I_w \cap I_{w'}|} {|I_w|}
$$
代表 w' 在含有 w 的日志行中发生的频繁度。换言之，$dep(w,w')$ 代表出现 $w$ 的日志里出现 $w'$ 的频率。

词 $w_i$ 在该模式中的权重计算公式：
$$
weights(w_i) = \frac {\Sigma_{j=1}^k dep(w_j, w_i)}  k
$$
直觉上说，在这个pattern中，这个词与其他词的相关性有多强。词的权重代表了词与模式中其他词之间关联的强度。

例如，假设模式是 “Daemon testd killed”，Daemon 和 killed 总是同时出现，testd从未和 Daemon和killed一起出现，Daemon和killed的权重是 1，如果只有 2.5%的日志同时含有 Daemon和killed及testd，testd 的权重为 (1 + 0.025 + 0.025) / 3 = 0.35

The Join_Clusters heuristic takes the user supplied word weight threshold t as its input parameter (0 < t ≤ 1). For each cluster, a secondary identifier is created and initialized to the cluster’s regular identifier tuple. Also, words with weights smaller than t are identified in the cluster’s line pattern, and each such word is replaced with a special token in the secondary identifier. Finally, clusters with identical secondary identifiers are joined. When two or more clusters are joined, the support of the joint cluster is set to the sum of supports of original clusters, and the line pattern of the joint cluster is adjusted to represent the lines in all original clusters.

Join_Clusters 启发式将用户提供的词权阈值 t 作为其输入参数 (0 < t ≤ 1)。 对于每个类簇，都会创建一个辅助标识符并将其初始化为集群的常规标识符元组。 此外，权重小于 t 的单词在集群的线条模式中被识别，并且每个这样的单词在辅助标识符中被替换为一个特殊的标记。 最后，加入具有相同辅助标识符的集群。 当两个或多个簇连接时，将联合簇的支持度设置为原始簇的支持度之和，调整联合簇的线型以表示所有原始簇中的线。



For example, if two clusters have patterns Interface *{1,1} down at node router1 and Interface *{2,3} down at node router2, and words router and router2 have insufficient weights, the clusters are joined into a new cluster with the line pattern Interface *{1,3} down at node (router1|router2). Fig. 2 describes the details of the Join_Clusters heuristic. Since the line pattern of a joint cluster consists of strongly correlated words, it is less likely to suffer from overfitting. Also, words with insufficient weights are incorporated into the line pattern as lists of alternatives, representing the knowledge from original patterns in a compact way without data loss. Finally, joining clusters will reduce their number and will thus make cluster reviewing easier for the human expert.

例如，如果两个集群在节点 router1 具有模式 
`Interface *{1,1} down at node router1`和 
`Interface *{2,3} down at node router2`，并且单词 router 和 router2 的权重不足，则将集群加入一个新集群 节点 (router1|router2) 的线型接口 *{1,3} 向下。 图 2 描述了 Join_Clusters 启发式的细节。 由于联合簇的线条模式由高度相关的单词组成，因此不太可能遭受过拟合。 此外，将权重不足的单词作为替代列表合并到线条模式中，以紧凑的方式表示来自原始模式的知识，而不会丢失数据。 最后，加入集群将减少它们的数量，从而使人类专家更容易进行集群审查。



流程图：
![](flow.png)

可使用规则表达式过滤日志，移除过滤的内容。

在挖掘过程中，现有的挖掘模式将词作为原子处理，不尝试发现词内部的潜在结构。

为解决上述问题，Log Cluster 遮盖特定词，创建词类。

如果一个词时非频繁的，但其所属词类时频繁的，在挖掘过程中词类替代词，并视之为频繁词

```
Procedure: Join_Clusters
Input: set of clusters C = {C1,…,Cp}
    word weight threshold t
    word weight function W()
Output: set of clusters C’ = {C’1,…,C’m}, m ≤ p
C’ := {}
for (j = 1; j <= p; ++j) do
    tuple := Cj.tuple
    k := # of elements in tuple
    for (i := 0; i < k; ++i) do
        if (W(tuple, i) < t) then
            tuple[i] := TOKEN
        fi
    done
    if (\exists Y \in C’, Y.tuple == tuple) then
        # 处理后如果有相同的pattern，则进行合并
        Y.support := Y.support + Cj.support
        for (i := 0; i < k+1; ++i) do
            if (Y.varmin[i] > Cj.varmin[i]) then
                Y.varmin[i] := Cj.varmin[i]
            fi
            if (Y.varmax[i] < Cj.varmax[i]) then
                Y.varmax[i] := Cj.varmax[i]
            fi
        done
    else
        initialize new cluster Y
        Y.tuple := tuple
        Y.support := Cj.support
        for (i := 0; i < k+1; ++i) do
            Y.varmin[i] := Cj.varmin[i]
            Y.varmax[i] := Cj.varmax[i]
            if (i < k AND Y.tuple[i] == TOKEN) then
                Y.wordlist[i] := \varnothing
            fi
        done
        C’ := C’ \cup { Y }
    fi
    Y.pattern := ()
    j: = 0
    for (i := 0; i < k; ++i) do
        if (Y.varmax[i] > 0) then
            min := Y.varmin[i]
            max := Y.varmax[i]
            Y.pattern[j] := “*{min,max}”
            ++j
        fi
        if (Y.tuple[i] == TOKEN) then
            if (Cj.tuple[i]  Y.wordlist[i]) then
                Y.wordlist[i] := Y.wordlist[i]  { Cj.tuple[i] }
            fi
            Y.pattern[j] := “( elements of Y.wordlist[i] separated by | )”
        else
            Y.pattern[j] := Y.tuple[i]
        fi
        ++j
    done
    if (Y.varmax[k] > 0) then
        min := Y.varmin[k]
        max := Y.varmax[k]
        Y.pattern[j] := “*{min,max}”
    fi
    done
    return C’
```

## 3.4 LogCluster整个流程
```
Procedure: LogCluster
Input:
    event log L = {l1,…,ln}
    support threshold s
    word sketch size h (optional)
    word weight threshold t (optional)
    word weight function W() (optional)
    boolean for invoking Aggregate_Supports
    procedure A (optional)
    file of outliers ofile (optional)
Output:
    set of clusters C = {C1,…,Cm} the cluster of outliers O (optional)

1. if (defined(h)) then 
    make a pass over L and build the word sketch
    of size h for filtering out infrequent words
    at step 2
2. make a pass over L and find the set of
    frequent words: F := {w | |Iw| ≥ s}
3. if (defined(t)) then
    make a pass over L and find dependencies for
    frequent words: {dep(w, w’) | w \in F, w’ \in Cw}
4. make a pass over L and find the set of cluster
    candidates X: X := Generate_Candidates(L, F)
5. if (defined(A) AND A == TRUE) then 
    invoke Aggregate_Supports() procedure
6. find the set of clusters C
    C := {Y \in X | supp(Y) ≥ s}
7. if (defined(t)) then
    join clusters: C := Join_Clusters(C, t, W)
8. report line patterns and their supports
    for clusters from set C
9. if (defined(ofile)) then
    make a pass over L and write outliers to ofile
```

1. 如果定义了h，创建一个词表，过滤非频繁词。
2. 找到大于s的频繁词集合：$F = \{w | |I_w| >= s\}$
3. 如果定义了t，找到频繁词之间的相关关系。${dep(w, w’) | w \in F, w’ \in Cw}$
4. 生成候选类簇。$X := Generate_Candidates(L, F)$
5. 如果A == TRUE，调用 Aggregate_Supports()
6. 生成类族C，类簇支持度大于s。 $C := {Y \in X | supp(Y) ≥ s}$
7. 如果定义了t，$C := Join_Clusters(C, t, W)$
8. 汇总行pattern和它们的支持度。
9. 如果定义了ofile，则输出到文件。


# 4 LogCluster 实现和性能
```bash
--lfiters       过滤特定行
--template      
--separator     正则表达式，匹配上的话作为单词的定界符
--wfilter
--wsearch


```

For assessing the performance of the LogCluster algorithm, we have created its publicly available GNU GPLv2 licensed prototype implementation in Perl. The implementation is a UNIX command line tool that can be downloaded from http://ristov.github.io/logcluster. Apart from its clustering capabilities, the LogCluster tool supports a number of data preprocessing options which are summarized below. In order to focus on specific lines during pattern mining, a regular expression filter can be defined with the --lfilter command line option. For instance, with --lfilter=’sshd\[\d+\]:’ patterns are detected for sshd syslog messages (e.g., May 10 11:07:12 myhost sshd[4711]: Connection from 10.1.1.1 port 5662).

为了评估 LogCluster 算法的性能，我们在 Perl 中创建了其公开可用的 GNU GPLv2 许可原型实现。 该实现是一个 UNIX 命令行工具，可以从 http://ristov.github.io/logcluster 下载。 除了集群功能外，LogCluster 工具还支持许多数据预处理选项，总结如下。 为了在模式挖掘期间关注特定行，可以使用 --lfilter 命令行选项定义正则表达式过滤器。 例如，使用 --lfilter='sshd\[\d+\]:' 模式检测 sshd 系统日志消息（例如，5 月 10 日 11:07:12 myhost sshd[4711]: Connection from 10.1.1.1 port 5662）。

If a template string is given with the --template option, match variables set by the regular expression of the --lfilter option are substituted in the template string, and the resulting string replaces the original event log line during the mining. For example, with the use of --lfilter=’(sshd\[\d+\]: .+)’ and --template=’$1’ options, timestamps and hostnames are removed from sshd syslog messages before any other processing. If a regular expression is given with the --separator option, any sequence of characters that matches this expression is treated as a word delimiter (word delimiter defaults to whitespace).

如果使用--template 选项给出模板字符串，则--lfilter 选项的正则表达式设置的匹配变量将替换在模板字符串中，并且生成的字符串替换挖掘期间的原始事件日志行。 例如，使用 --lfilter='(sshd\[\d+\]: .+)' 和 --template='$1' 选项，时间戳和主机名会在任何其他处理之前从 sshd syslog 消息中删除。 如果使用 --separator 选项给出正则表达式，则与该表达式匹配的任何字符序列都将被视为单词分隔符（单词分隔符默认为空格）。


Existing line pattern mining tools treat words as atoms during the mining process, and make no attempt to discover potential structure inside words (the only exception is SLCT which includes a simple post-processing option for detecting constant heads and tails for wildcards). In order to address this shortcoming, LogCluster implements several options for masking specific word parts and creating word classes. If a word matches the regular expression given with the --wfilter option, a word class is created for the word by searching it for substrings that match another regular expression provided with the --wsearch option. All matching substrings are then replaced with the string specified with the --wreplace option. For example, with the use of --wfilter=’=’, --wsearch=’=.+’, and --wreplace=’=VALUE’ options, word classes are created for words which contain the equal sign (=) by replacing the characters after the equal sign with the string VALUE. Thus, for words pid=12763 and user=bob, classes pid=VALUE and user=VALUE are created. If a word is infrequent but its word class is frequent, the word class replaces the word during the mining process and will be treated like a frequent word. Since classes can represent many infrequent words, their presence in line patterns provides valuable information about regularities in word structure that would not be detected otherwise.

现有的线型挖掘工具在挖掘过程中将单词视为原子，并且不会尝试发现单词内部的潜在结构（唯一的例外是 SLCT，它包括一个简单的后处理选项，用于检测通配符的恒定头部和尾部）。为了解决这个缺点，LogCluster 实现了几个用于屏蔽特定单词部分和创建单词类的选项。如果一个词与 --wfilter 选项提供的正则表达式匹配，则通过搜索与 --wsearch 选项提供的另一个正则表达式匹配的子字符串来为该词创建一个词类。然后将所有匹配的子字符串替换为 --wreplace 选项指定的字符串。例如，通过使用 --wfilter='='、--wsearch='=.+' 和 --wreplace='=VALUE' 选项，将为包含等号 (=) 的单词创建单词类通过将等号后面的字符替换为字符串 VALUE。因此，对于单词 pid=12763 和 user=bob，创建了类 pid=VALUE 和 user=VALUE。如果一个词不频繁但它的词类是频繁的，则词类在挖掘过程中替换了这个词，将被视为一个频繁词。由于类可以表示许多不常见的单词，它们在线条模式中的存在提供了关于单词结构规律性的有价值的信息，否则这些信息不会被检测到。

For evaluating the performance of LogCluster and comparing it with other algorithms, we conducted a number of experiments with larger event logs. For the sake of fair comparison, we re-implemented the public C-based version of SLCT in Perl. Since the implementations of IPLoM and the algorithm by Reidemeister et al. are not publicly available, we were unable to study their source code for creating their exact prototypes. However, because the algorithm by Reidemeister et al. uses SLCT and has a similar time complexity (see section II), its runtimes are closely approximated by results for SLCT. During our experiments, we used 6 logs from a large institution of a national critical information infrastructure of an EU state. The logs cover 24 hour timespan (May 8, 2015), and originate from a wide range of sources, including database systems, web proxies, mail servers, firewalls, and network devices. We also used an availability monitoring system event log from the NATO CCD COE Locked Shields 2015 cyber defense exercise which covers the entire two-day exercise and contains Nagios events. During the experiments, we clustered each log file three times with support thresholds set to 1%, 0.5% and 0.1% of lines in the log. We also used the word sketch of 100,000 counters (parameter h in Fig. 3) for both LogCluster and SLCT, and did not employ Aggregate_Supports and Join_Clusters heuristics. Therefore, both LogCluster and SLCT were configured to make three passes over the data set, in order to build the word sketch during the first pass, detect frequent words during the second pass, and generate cluster candidates during the third pass. All experiments were conducted on a Linux virtual server with Intel Xeon E5-2680 CPU and 64GB of memory, and Table I outlines the results. Since LogCluster and SLCT implementations are both single-threaded and their CPU utilization was 100% according to Linux time utility during all 21 experiments, each runtime in Table I closely matches the consumed CPU time.

为了评估 LogCluster 的性能并将其与其他算法进行比较，我们使用更大的事件日志进行了许多实验。为了公平比较，我们在 Perl 中重新实现了基于 C 的公共版本的 SLCT。由于 IPLoM 和 Reidemeister 等人的算法的实现不公开，我们无法研究他们的源代码来创建他们的确切原型。然而，因为 Reidemeister 等人的算法使用 SLCT 并具有相似的时间复杂度（参见第二部分），其运行时间与 SLCT 的结果非常接近。在我们的实验中，我们使用了来自欧盟国家的国家关键信息基础设施的大型机构的 6 条日志。日志涵盖 24 小时时间跨度（2015 年 5 月 8 日），来源广泛，包括数据库系统、Web 代理、邮件服务器、防火墙和网络设备。我们还使用了来自 NATO CCD COE Locked Shields 2015 网络防御演习的可用性监控系统事件日志，该日志涵盖了整个为期两天的演习并包含 Nagios 事件。在实验期间，我们将每个日志文件聚类 3 次，支持阈值设置为日志中行的 1%、0.5% 和 0.1%。我们还为 LogCluster 和 SLCT 使用了 100,000 个计数器的单词草图（图 3 中的参数 h），并且没有使用 Aggregate_Supports 和 Join_Clusters 启发式。因此，LogCluster 和 SLCT 都被配置为对数据集进行三遍，以便在第一遍中构建单词草图，在第二遍中检测常用词，并在第三遍中生成候选聚类。所有实验均在配备 Intel Xeon E5-2680 CPU 和 64GB 内存的 Linux 虚拟服务器上进行，表 I 概述了结果。由于 LogCluster 和 SLCT 实现都是单线程的，并且在所有 21 个实验中，根据 Linux 时间实用程序，它们的 CPU 利用率都是 100%，因此表 I 中的每个运行时都与消耗的 CPU 时间非常匹配。

TABLE I. PERFORMANCE OF LOGCLUSTER AND SLCT
![](table1.png)

```
May 8 *{1,1} myserver dhcpd: DHCPREQUEST for *{1,2} from *{1,2} via *{1,4}

May 8 *{3,3} Note: no *{1,3} sensors

May 8 *{3,3} RT_IPSEC: %USER-3-RT_IPSEC_REPLAY: Replay packet detected on IPSec tunnel on *{1,1} with tunnel ID *{1,1} From *{1,1} to *{1,1} ESP, SPI *{1,1} SEQ *{1,1}

May 8 *{1,1} myserver httpd: client *{1,1} request GET *{1,1} HTTP/1.1 referer *{1,1} User-agent Mozilla/5.0 *{3,4} rv:37.0) Gecko/20100101 
Firefox/37.0 *{0,1}

May 8 *{1,1} myserver httpd: client *{1,1} request GET *{1,1} HTTP/1.1 referer *{1,1} User-agent Mozilla/5.0 (Windows NT *{1,3} AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Safari/537.36
```
Fig. 4. Sample clusters detected by LogCluster (for the reasons of privacy, sensitive data have been obfuscated).

As results indicate, SLCT was 1.28–1.62 times faster than LogCluster. This is due to the simpler candidate generation procedure of SLCT – when processing individual event log lines, SLCT does not have to check the line patterns of candidates and adjust them if needed. However, both algorithms require considerable amount of time for clustering very large log files. For example, for processing the largest event log of 16.3GB (rows 13-15 in Table I), SLCT needed about 1.5 hours, while for LogCluster the runtime exceeded 2 hours. In contrast, the C-based version of SLCT accomplishes the same three tasks in 18-19 minutes. Therefore, we expect a C implementation of LogCluster to be significantly faster.

结果表明，SLCT 比 LogCluster 快 1.28-1.62 倍。 这是由于 SLCT 的候选生成过程更简单——在处理单个事件日志行时，SLCT 不必检查候选的行模式并在需要时对其进行调整。 但是，这两种算法都需要大量时间来聚类非常大的日志文件。 例如，为了处理 16.3GB 的最大事件日志（表 I 中的第 13-15 行），SLCT 需要大约 1.5 小时，而对于 LogCluster，运行时间超过 2 小时。 相比之下，基于 C 的 SLCT 版本在 18-19 分钟内完成了相同的三项任务。 因此，我们预计 LogCluster 的 C 实现会明显更快。

According to Table I, LogCluster finds less clusters than SLCT during all experiments (some clusters are depicted in Fig. 4). The reviewing of detected clusters revealed that unlike SLCT, LogCluster was able to discover a single cluster for lines where frequent words were separated with a variable number of infrequent words. For example, the first cluster in Fig. 4 properly captures all DHCP request events. In contrast, SLCT discovered two clusters `May 8 * myserver dhcpd: DHCPREQUEST for * from * * via` and `May 8 * myserver dhcpd: DHCPREQUEST for * * from * * via` which still do not cover all possible event formats. Also, the last two clusters in Fig. 4 represent all HTTP requests originating from the latest stable versions of Firefox browser on all OS platforms and Chrome browser on all Windows platforms, respectively (all OS platform strings are matched by `*{3,4}` for Firefox, while `Windows NT *{1,3}` matches all Windows platform strings for Chrome). Like in the previous case, SLCT was unable to discover equivalent two clusters that would concisely capture HTTP request events for these two browser types.

根据表 I，LogCluster 在所有实验中发现的集群少于 SLCT（一些集群如图 4 所示）。对检测到的集群的审查表明，与 SLCT 不同，LogCluster 能够发现单个集群，其中频繁词与可变数量的不常用词分开。例如，图 4 中的第一个集群正确地捕获了所有 DHCP 请求事件。相比之下，SLCT 在 `May 8 * myserver dhcpd: DHCPREQUEST for * from * * via`    和 
`May 8 * myserver dhcpd: DHCPREQUEST for * * from * * via`
中发现了两个集群，它们仍然没有涵盖所有可能的事件格式。此外，图 4 中的最后两个集群分别代表来自所有 OS 平台上最新稳定版本的 Firefox 浏览器和所有 Windows 平台上 Chrome 浏览器的所有 HTTP 请求（所有 OS 平台字符串由 `*{3,4}` 匹配对于 Firefox，而 `Windows NT *{1,3}` 匹配 Chrome 的所有 Windows 平台字符串）。与前面的案例一样，SLCT 无法发现等效的两个集群，它们可以简明地捕获这两种浏览器类型的 HTTP 请求事件。

When evaluating the Join_Clusters heuristic, we found that word weight thresholds (parameter t in Fig. 3) between 0.5 and 0.8 produced the best joint clusters. Fig. 5 displays three sample joint clusters which were detected from the mail server and Nagios logs (rows 16-21 in Table I). Fig. 5 also illustrates data preprocessing capabilities of the LogCluster tool. For the mail server log, a word class is created for each word which contains punctuation marks, so that all sequences of non-punctuation characters which are not followed by the equal sign (=) or opening square bracket ([) are replaced with a single X character. For the Nagios log, word classes are employed for masking blue team numbers in host names, and also, trailing timestamps are removed from each event log line with --lfilter and --template options. The first two clusters in Fig. 5 are both created by joining three clusters, while the last cluster is the union of twelve clusters which represent Nagios SSH service check events for 192 servers.

在评估 Join_Clusters 启发式时，我们发现在 0.5 和 0.8 之间的词权阈值（图 3 中的参数 t）产生了最好的联合集群。图 5 显示了从邮件服务器和 Nagios 日志中检测到的三个示例联合集群（表 I 中的第 16-21 行）。图 5 还说明了 LogCluster 工具的数据预处理能力。对于邮件服务器日志，为每个包含标点符号的单词创建一个单词类，以便将所有后跟等号 (=) 或左方括号 ([) 的非标点字符序列替换为单个 X 字符。对于 Nagios 日志，使用单词类来屏蔽主机名中的蓝队编号，并且使用 --lfilter 和 --template 选项从每个事件日志行中删除尾随时间戳。图 5 中的前两个集群都是通过加入三个集群创建的，而最后一个集群是十二个集群的联合，代表 192 个服务器的 Nagios SSH 服务检查事件。


# 5 结论

In this paper, we have described the LogCluster algorithm for mining patterns from event logs. For future work, we plan to explore hierarchical event log clustering techniques. We also plan to implement the LogCluster algorithm in C, and use LogCluster for automated building of user behavior profiles.

在本文中，我们描述了从事件日志中挖掘模式的 LogCluster 算法。 对于未来的工作，我们计划探索分层事件日志聚类技术。 我们还计划在 C 中实现 LogCluster 算法，并使用 LogCluster 自动构建用户行为档案。


# 6 开源实现
## 6.1 logparser
python部分代码流程：
1. 生成调用perl的命令行，包括参数，并且指定perl的临时输入输出文件路径。
2. 生成日志格式的正则表达式。
```python
headers, regex = self.generate_logformat_regex(self.log_format)
```
3. 处理后的日志放入临时输入文件中。
4. 调用 perl 程序，输出文件中为生成的行模板。
```bash
perl ../logparser/LogCluster/logcluster.pl --input LogCluster_result.log_10000.-0.1-/logcluster_input.log -rsupport 0.1 -aggrsup > LogCluster_result.log_10000.-0.1-/logcluster_output.txt
```
5. 根据临时输出文件，得到事件模板、日志情况聚类等。

# 7 个人理解
1. 什么样的模板是好的？模板由频繁词+通配符构成，频繁词尽量多，通配符（匹配字符内容）尽量少，但是匹配的日志尽量多。
2. 频繁词能全找出来吗？假如匹配日志大于s行的才认为是一个类，则构成这个模板的频繁词数量必然也都大于s。如果一个类有某个关键词w，另一个类也有w，则w出现次数大于2s。
3. 簇能全找出来吗？找出来的模板不一定形成簇，但是能形成簇的模板能全部找出来。全部日志不区位置查找频繁词，由频繁词构成模板，虽然模板可能不是能匹配S行以上的簇（例如频繁词出现s次，但是在多个模板中出现），但是只要是能构成簇的模板也会全部找出来。
4. aggregrate_support ？用在生成候选簇之后、簇被选定前。如果一个模板A能匹配另一个模板B，即B有A的全部频繁词，A有更多通配符，则A合并B，这样有更多未形成簇的候选可以符合大于s的条件，也就会增多簇。
5. join_cluster ? 如2中所说，由频繁词构成的模板匹配日志未必大于s，由于 某个频繁词总量大于s，但是分散于多个模板中，而模板匹配日志数量却未必大于s。因此，需要计算一个模板内频繁词在所有日志的共现频率，然后再计算频繁词在模板中权重。


# 参考资料

1. [Log Cluster：日志数据聚类和模式挖掘算法](https://blog.csdn.net/MarkAustralia/article/details/122242966)
2. [https://github.com/ristov/logcluster](https://github.com/ristov/logcluster)
3. [https://github.com/logpai/logparser](https://github.com/logpai/logparser.git)
4. [https://github.com/zhugegy/LogClusterC](https://github.com/zhugegy/LogClusterC)

