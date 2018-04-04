+++
date = "30 Mar 2018"
draft = true
title = "Class 9: Adversarial Malware Detection"
author = "Team Nematode"
slug = "class9"
+++


### Introduction

### Malware History - Machine Learning Origins

### PDF Malware Classifiers

## Hidost: A Static Machine-Learning-Based Detector of Malicious Files
There has been a substantial amount of work on the detection of no-executable malware which includes static, dynamic and combined methods. Although static methods perform in orders of magnitude faster, their applicability has been limited to only specific file formats. Hidost introduces the static machine-learning-based malware detection system to operate multiple file formats like pdf or swf having hierarchical  document structure. 

### Hierarchically structured file formats
File formats are developed as a mean to store the physical representation of certain information but all of them do not have logical structure. For example, some formats like text files do not have any logical structure but others e.g.; HTML files represents a logical relationships between html elements. Following two figures shows the hierarchical structure of pdf and  swf files respectively.

<p align="center">
<img src="/images/pdf_structure.png" width="400" >
<br> <b>Figure:</b> Raw content of an example pdf (left) and its structural tree (right)
</p>

<p align="center">
<img src="/images/swf_structure.png" width="400" >
<br> <b>Figure:</b> Decoded swf file (left) and its logical structure(right)
</p>

### Distinguishing benign from malicious files
In a pdf structural tree, a path is defined as a sequence of edges starting in the Catalog dictionary and ending with an object of primitive type. For example, as we see in the above figure of pdf structural tree, there s a path from the root path from the root, i.e., leftmost, node
through the edges named /Pages and /Count to the terminal node with the value 2. This definition of a path in the PDF document structure, which is denoted as PDF structural path, plays a central role in Hidost approach. Paths are printed as a sequence of all edge labels encountered during path traversal starting from the root node and ending in the leaf node. The path from our earlier example
would be printed as /Pages/Count. 

Following is a list of example structural paths of real world benign files: <br/>
/Metadata <br/>
/Type <br/>
/Pages/Kids <br/>
/OpenAction/Contents <br/>
/StructTreeRoot/RoleMap <br/>
/Pages/Kids/Contents/Length <br/>
/OpenAction/D/Resources/ProcSet <br/>
/OpenAction/D <br/>
/Pages/Count <br/>
/PageLayout <br/>

Presence of these structural paths in a file indicates that it is benign. Alternatively, the absence of these paths is indicative to the fact that the file may be malicious. In addition, malicious files are not likely to contain metadata in order to minimize file size, they do not jump to a page in the document when it is opened and are not well-formed so they are missing paths such as /Type and /Pages/Count.
The following is a list of structural paths from real-world malicious PDF files:<br/>
/AcroForm/XFA <br/>
/Names/JavaScript <br/>
/Names/EmbeddedFiles <br/>
/Names/JavaScript/Names <br/>
/Pages/Kids/Type <br/>
/StructTreeRoot <br/>
/OpenAction/Type <br/>
/OpenAction/S <br/>
/OpenAction/JS <br/>
/OpenAction <br/>

### System Design
The system design of Hidost consists of six stages: structure extraction, structural path consolidation, feature selection, vectorization, learning, and classification as it is illustrated in the following figure.

<p align="center">
<img src="/images/system_design.png" width="500" >
<br> <b>Figure:</b> Hidost system design
</p>

### File structure extraction
In this stage, files are transformed into more abstract representation (logical structure) i.e.; into a structural multimap. Multimap is basically a map with associations between every structural path to the set of all leaves that lie on a given path.
The concept is similar to map but you have multiple leafs as we see in path mediabox in the following figure.
 
<p align="center">
<img src="/images/structure_extraction.png" width="300" >
<br> <b>Figure:</b> Complete structural multimap of the PDF file used as an example in previous section
</p>

### Structural path consolidation
There may be cases in which semantically equivalent, but syntactically different structures can avoid detection. In Hidost, heuristic technique to consolidate structural paths to reduce polymorphic paths.  For example, /Pages/Kids/Resources and /Pages/Kids/Kids/Resources
can be reduced to /Pages/Resources.

### Feature Selection
Feature Selection is used to limit the rare features once again similar to many Machine Learning techniques.

### Vectorization
In the vectorization stage, structural multimaps are first replaced by structural maps—ordinary map data structures that map a structural path to a corresponding single numeric value. To this end, every set of values corresponding to one structural path in the multimap is reduced to its median.

<p align="center">
<img src="/images/vectorization.png" width="200" >
<br> <b>Figure:</b> Reduction to median
</p>

### Learning and Classification
The authors have used Random Forest implementation at this stage. But, it can be any classifier as per reader's choice. 

### Experimental evaluation
The authors run their experiment on two datasets, one for PDF and one for SWF file formats. Both of the datasets were collected from [VirusTotal](https://www.virustotal.com/). A file is considered as malicious if it is labeled as malicious by at least five engines. Alternatively, a file is labeled benign by all antivirus engines. Following figure shows that HiDost performs better that the antivirus engines in the VirusTotal. It should be mentioned that VirusTotal may not contain the most efficient antivirus developed by the proprietor company.

<p align="center">
<img src="/images/experiment.png" width="500" >
<br> <b>Figure:</b> Experimental evaluation
</p>

### Machine Learning on Malware is Hard
While machine learning algorithms have been successfully applied in various vision and natural language related tasks, they have not been fully successful in malware detection. The reasons are mainly three-fold:

#### High Cost of Error
In the case of malware detection, when measuring the performance of a classifier, both false positive rate and false negative rate are important factors. While the false positive rate means a benign software is classified as a malware, in the false negative case a malware is classified as benign. Both the events are undesirable for malware detection. Since, false positive requires excessive spending of a human analyst's time which could be expensive and time consuming. On the other hand, false negative means a malware is left undetected, which can have serious security consequences. A typical machine learning model trained for the malware detection task tends to have non-zero false positive rates and false negative rates.

#### Semantic Gap
There is a disconnect between the prediction result of a machine learning model and the action to be taken based on the result. For instance, if a model predicts a malware with 65% confidence, what action should be taken? Should the target software be removed without intimation to the end user? Should the user be alerted for a manual inspection? In such a scenario, it is hard to interpret the results and take a meaningful action.

#### Difficulty with Evaluation
One of the major difficulty in the evaluation of malware detection tool is the availability of datasets. Most of the public datasets like Contagio or Drebin are small or outdated, whereas the malwares keep updating aggressively. It is difficult to create new rich malware datasets due to data privacy concerns and the labelling of malwares in the wild. This poses problem for training and evaluating the machine learning models.

Another difficulty is the evasiveness of the malwares. Malwares have become dynamic enough to evade the malware classifiers. Given a white-box access to the classifier, malware can perform adversarial training like gradient-based method to evade detection. Even with black-box access, the malware can perform mimicry attack by appending features of benign samples. However, this may or may not break the malware's functionality and hence it is hard to generate useful samples. One variant is to use [reverse-mimicry attack](https://dl.acm.org/citation.cfm?id=2484327) by embedding malicious features into benign file to generate malicious samples. Recent literature has also shown evasion by [genetic mutation](http://people.cs.vt.edu/~gangwang/class/cs6604/papers/ndss16.pdf) and [hill climbing method](https://acmccs.github.io/papers/p119-dangA.pdf), where the malware can dynamically adapt to evade detection

### State of the Art

### Future of the Field

### Conclusion

We've explored the current state of adversarial malware detection including Hidost. From simple to metamorphic viruses, malware has evolved into increasingly sophisticated threats. The field of adversarial malware detection has been evolving as well, as methods are developed that are increasingly effective at detecting malicious code. As the cutting edge of malware detection moves forward, we will see new fronts open up in the contest between malicious files and the algorithms that detect them.

### Sources
> Liang Tong, Bo Li, Chen Hajaj, Chaowei Xiao, Yevgeniy Vorobeychik. 2017. Hardening Classifiers against Evasion: the Good, the Bad, and the Ugly.  arXiv:1708.08327. Retrieved from https://arxiv.org/abs/1708.08327v2 [[PDF]] (https://arxiv.org/abs/1708.08327v2)

> R. Sommer and V. Paxson, "Outside the Closed World: On Using Machine Learning for Network Intrusion Detection," 2010 IEEE Symposium on Security and Privacy, Oakland, CA, USA, 2010, pp. 305-316.
https://doi.org/10.1109/SP.2010.25 [[PDF]] (http://ieeexplore.ieee.org/document/5504793/)
