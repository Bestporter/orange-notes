logging模块



http://www.liuqingzheng.top/python/%E5%B8%B8%E7%94%A8%E6%A8%A1%E5%9D%97/8-logging%E6%A8%A1%E5%9D%97/

```python
import os
import logging.config

# 定义三种日志输出格式 开始
standard_format = '[%(asctime)s][%(threadName)s:%(thread)d][task_id:%(name)s][%(filename)s:%(lineno)d]' \
                  '[%(levelname)s][%(message)s]'  # 其中name为getLogger()指定的名字；lineno为调用日志输出函数的语句所在的代码行
simple_format = '[%(levelname)s][%(asctime)s][%(filename)s:%(lineno)d]%(message)s'
id_simple_format = '[%(levelname)s][%(asctime)s] %(message)s'
# 定义日志输出格式 结束

logfile_dir = os.path.dirname(os.path.abspath(__file__))  # log文件的目录，需要自定义文件路径 # atm
logfile_dir = os.path.join(logfile_dir, 'log')  # C:\Users\oldboy\Desktop\atm\log

logfile_name = 'log.log'  # log文件名，需要自定义路径名

# 如果不存在定义的日志目录就创建一个
if not os.path.isdir(logfile_dir):  # C:\Users\oldboy\Desktop\atm\log
    os.mkdir(logfile_dir)

# log文件的全路径
logfile_path = os.path.join(logfile_dir, logfile_name)  # C:\Users\oldboy\Desktop\atm\log\log.log
# 定义日志路径 结束

# log配置字典
LOGGING_DIC = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': standard_format
        },
        'simple': {
            'format': simple_format
        },
    },
    'filters': {},  # filter可以不定义
    'handlers': {
        # 打印到终端的日志
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',  # 打印到屏幕
            'formatter': 'simple'
        },
        # 打印到文件的日志,收集info及以上的日志
        'default': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件
            'formatter': 'standard',
            'filename': logfile_path,  # 日志文件
            'maxBytes': 1024 * 1024 * 5,  # 日志大小 5M  (*****)
            'backupCount': 5,
            'encoding': 'utf-8',  # 日志文件的编码，再也不用担心中文log乱码了
        },
    },
    'loggers': {
        # logging.getLogger(__name__)拿到的logger配置。如果''设置为固定值logger1，则下次导入必须设置成logging.getLogger('logger1')
        '': {
            # 这里把上面定义的两个handler都加上，即log数据既写入文件又打印到屏幕
            'handlers': ['default', 'console'],
            'level': 'DEBUG',
            'propagate': False,  # 向上（更高level的logger）传递
        },
    },
}

def f():
    raise Exception("ERROR !!!!!!!!")
    
# 日志模块
def load_my_logging_cfg():
    logging.config.dictConfig(LOGGING_DIC)  # 导入上面定义的logging配置
    logger = logging.getLogger(__name__)  # 生成一个log实例
    logger.info('It works!')  # 记录该文件的运行状态
    try:
        f()
    except Exception as e:
        logger.error(e.args)
    return logger


if __name__ == '__main__':
    load_my_logging_cfg()
# '''
# critical=50
# error =40
# warning =30
# info = 20
# debug =10
# '''
#
#
# import logging
#
# # 1、logger对象：负责产生日志，然后交给Filter过滤，然后交给不同的Handler输出
# logger = logging.getLogger(__file__)
#
# # 2、Filter对象：不常用，略
#
# # 3、Handler对象：接收logger传来的日志，然后控制输出
# h1 = logging.FileHandler('t1.log')  # 打印到文件
# h2 = logging.FileHandler('t2.log')  # 打印到文件
# sm = logging.StreamHandler()  # 打印到终端
#
# # 4、Formatter对象：日志格式
# formmater1 = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s -%(module)s:  %(message)s',
#                                datefmt='%Y-%m-%d %H:%M:%S %p',)
#
# formmater2 = logging.Formatter('%(asctime)s :  %(message)s',
#                                datefmt='%Y-%m-%d %H:%M:%S %p',)
#
# formmater3 = logging.Formatter('%(name)s %(message)s',)
#
#
# # 5、为Handler对象绑定格式
# h1.setFormatter(formmater1)
# h2.setFormatter(formmater2)
# sm.setFormatter(formmater3)
#
# # 6、将Handler添加给logger并设置日志级别
# logger.addHandler(h1)
# logger.addHandler(h2)
# logger.addHandler(sm)
#
# # 设置日志级别，可以在两个关卡进行设置logger与handler
# # logger是第一级过滤，然后才能到handler
# logger.setLevel(30)
# h1.setLevel(10)
# h2.setLevel(10)
# sm.setLevel(10)
#
# # 7、测试
# logger.debug('debug')
# logger.info('info')
# logger.warning('warning')
# logger.error('error')
# logger.critical('critical')
```

