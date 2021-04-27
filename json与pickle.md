# json与pickle

json序列化可以全平台通用，pickle只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，即不能成功地反序列化也没关系。但是pickle的好处是可以存储Python中的所有的数据类型，包括对象，而json不可以。

pickle存储数据到文件是以二进制写入，

普通操作都是dumps和loads， 写入文件和读出，都是dump和load

参考http://www.liuqingzheng.top/python/%E5%B8%B8%E7%94%A8%E6%A8%A1%E5%9D%97/6-json%E5%92%8Cpickle%E6%A8%A1%E5%9D%97/