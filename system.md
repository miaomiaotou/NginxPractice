# 一、概述
这里主要记录一些运维中和操作系统相关的问题，以及解决方案。

# 一、重要的性能参数

# 二、重要的系统日志

# 三、进程莫名down
在个人开发过程中，经常会发现服务进程down，但是又并非core。这种现象在测试环境频发，正式线上环境反而会较少遇到。例如本人维护的测试环境Redis经常会掉线，对于这种情况，最简单的处理方案是写一个简单的监控启动脚本，并通过`crontab`来定时执行，该脚本的功能是监控到相关进程down，就进行启动, 下面是一个python的redis进程监控启动脚本，以及对应的定时器。
```py
#!/bin/python
# -*- coding:utf-8 -*-

import os
import time

def redis_exist():
    out = os.popen("ps aux|grep redis-server").readlines()
    for proc in out:
        proc = proc.strip()
        if "6379" in proc :
            return True
    return False

def main():
    now = int(time.time())
    exist = redis_exist()
    if exist:
        return True
    out = os.popen("redis-server /etc/redis.conf").readlines()
    exist = redis_exist()
    log = "%s : redis not exist and restart %s" % (now, "success" if exist else "failed")
    os.popen("echo %s >> /root/redis-sbin/log-redis-restart" % log)
    return exist




if __name__ == "__main__":
    os.chdir(os.path.dirname(os.path.abspath(__file__)))
    print "redis : ", main()
```

```sh
$ crontab -e

# ------------- 下面是crontab的内容 -------------
# 定时启动
* * * * * cd <监控脚本路径> && ./<监控脚本>
```

监控到进程是否down的方案很多，这只是其中一种方式，但无论如何，监控只是一种缓解的方案，并不能解决到问题的根本原因。