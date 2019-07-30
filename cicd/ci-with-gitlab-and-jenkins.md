# CI/CD

* [Create A Continuous Integration Pipeline With GitLab And Jenkins](https://docs.bitnami.com/bch/how-to/create-ci-pipeline/)
介绍了一个基本的ci流程：
    * dev -- GitLab -- Jenkins
    * 当gitlab改变，jenkins自动触发；
    * jenkins build成功，代码自动合并到production分支；


猜想的一个完成流程(减少手动步骤)：
    * development --> gitlab dev branch --> jenkins trigger build / docker / push harbor --> deploy test --> trigger QA --> if(ok) --> jenkins push production --> 人工卡点 --> productino deploy k8s --> 一键回退之前版本
(测试实现一遍)


* 明天试用下 gitlab ci/cd  [Creating and using CI/CD pipelines](https://docs.gitlab.com/ee/ci/pipelines.html)
    