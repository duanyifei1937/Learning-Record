# Index Template & Dynamic Template

## index template
* 组成：
    mappings + settings
    
### index template使用
* template只在索引创建是产生作用，修改模板不影响已经创建的索引；
* 可以设定过个索引模板，自动merge到一起；
* 指定`order`, 控制”merging“的过程；

### index template 工作方式
* 当一个索引配创建时：
    * 使用default setting + mappings;
    * 应用`order`数值低的index template;
    * 应用`order`数值高的index template, 覆盖之前；
    * 用户指定的setting、mappings, 覆盖之前模板中的设定；


## Dynimac Template
* 
    
