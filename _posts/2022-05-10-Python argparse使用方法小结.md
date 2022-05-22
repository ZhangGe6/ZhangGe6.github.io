---
title: Python argparse使用方法小结
author: Zhang Ge
date: 202２-05-10 19:42:00 +0800
categories: [实践, 框架学习]
tags: [Python, argparse]
---

本页对Python常用的命令行解析模块`argparse`的常用方法做一个小结，为了清晰和简化，每个例程只涉及一种功能，便于日后参考。

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument('name', help='test positional arg 1')
parser.add_argument('age', help='test positional arg 2')
## ==> usage: python demo.py zg 25

parser.add_argument('-i', '--input', help='test short and long arg')
# optional, default is None

parser.add_argument('--bool', action='store_true', help='test action#store_true')
## ==> usage: python demo.py --bool
# out: Namespace(bool=True)

parser.add_argument('-v', action='append', help="test action#append")
## ==> usage: python demo.py -v v1 -v v2
# out: Namespace(v=['v1', 'v2'])

parser.add_argument('--required', required=True, help='test required')

parser.add_argument('-d', dest='d_dest', help='test dest')
## ==> usage: python demo.py -d 3
## out: Namespace(d_dest='3')
# https://www.pynote.net/archives/2121

parser.add_argument('-i',  help='test type')
## ==> usage: python demo.py -i 3
## out: Namespace(i='3')
parser.add_argument('-i', type=int, help='test type')
## ==> usage: python demo.py -i 3
## out: Namespace(i=3)

parser.add_argument('-i', default=3, help='test default')
## ==> usage: python demo.py
## out: Namespace(i=3)
## ==> usage: python demo.py -i 5
## out: Namespace(i='5')   # pay attention to the datatype here

parser.add_argument('-v', metavar='--value', help="test metavar")
## ==> usage: python demo.py -h
# It provides a different name for optional argument in [error and help messages]

# https://docs.python.org/zh-cn/3/library/argparse.html#nargs
parser.add_argument('-v', nargs=2, help="test nargs N")
## ==> usage: python demo.py -v 1 2 (only allow constant num)
# out: Namespace(v=['1', '2'])  => list
## ==> usage: python demo.py -v 1
# out: demo.py: error: argument -v: expected 2 arguments
parser.add_argument('-v', nargs='*', help="test nargs *")
# * nargs expects 0 or more arguments (allow varible num)
## ==> usage: python demo.py -v 1 2... (variable num)
parser.add_argument('-v', nargs='+', help="test nargs *")
# similar to '+' nargs

parser.add_argument('--platform', '-p', choices=['windows', 'linux'], help="test choices")
# choices limits argument values to the given list

args = parser.parse_args()
print(args)

# a = args.XXX
```

