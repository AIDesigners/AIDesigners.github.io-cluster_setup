## Spark Cluster Setup

In 2010, Google introduced MapReduce, a batch-processing framework for running computations involving large data volume over a commodity cluster. MapReduce was later adopted by several companies including Google, Amazon, IBM etc. It solved the scalability issues to great extent when dealing with large datasets and showed good performance, provided that the recipes for data processing do not involve any repeatative processing of a single step. The reason for this was that MapReduce reads data at the begining of a step and the scope of the objects holding this data are upto the end of this step. So the data objects are destoyed once a step ends. As a result, repeating a step incurs significant parallelism overhead. 

Spark was introduced to mainly overcome this limitation of MapReduce.Spark is a fast and memory-efficent clustering computing framework. It support both batch-style as well as computations over data streams. It provides high-level APIs in Scala, Java, Python, and R. It also provides an optimized engine that supports general computation graphs for data analysis and a machine learning library. Support for running on YARN (Hadoop NextGen) was added to Spark in version 0.6.0. You can learn more about this <a href="https://spark.apache.org/docs/latest/index.html">here</a> and <a href="https://en.wikipedia.org/wiki/Apache_Spark">here</a> 

In this tutorial, we are going to show you how to setup a spark cluster in distributed mode using a cluster of machine running Linux. For simplicity, we are going to setup a cluster of two nodes only. 

1. First, we need to  

You can use the [editor on GitHub](https://github.com/AIDesigners/AIDesigners.github.io-cluster_setup/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/AIDesigners/AIDesigners.github.io-cluster_setup/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
