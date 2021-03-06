# 如何绘制出简易的CFG图
代码可在code_cmp_cfg.py中查看。
## 1.提取c语言函数体，以及对应的函数名。
使用正则表达式：
```Python
pattern = re.compile(r'((?:const[ \t\*]+)?(?:void|int|char|long|double|float|unsigned|unsigned int|unsigned long|long long)[ \t\*]+(?:const[ \t*]+)?)(\w+)(\([^\;]*\))[^\;]*(\{([^{}]*(\{([^{}]*(\{([^{}]*(\{[^{}]*\})*[^\{\}]*?)*\})*[^\{\}]*?)*\})*[^\{\}]*?)*\})', re.S)
```
对读取的c语言代码进行匹配，代码如下：
```Python
'''
    input:
        filepath:要分析的c代码路径（包括文件名），字符串
    output:
        group:获得的函数列表
'''
def get_c_functions(filepath):
    f = open(filepath, "r")
    str = f.read()
    f.close()
    c_group = c_pattern.findall(str)
    return c_group
```

## 2.在函数体中查找已知的函数名,并使用pygraphviz绘制出cfg图。
```Python
'''
    input:
        names: 函数名称，列表
        functions：函数名称和对应的函数体，字典
    output:
        在当前文件夹下生成“cfg.png”,并且在命令行中打印函数调用
'''
def get_cfg_graph(c_group):
    names = []
    functions = {}
    for g in c_group:
        names.append(g[1])
        functions[g[1]] = g[3]

    graph = gv.AGraph(directed=True)

    for name in names:
        graph.add_node(name)

    for (key, value) in functions.items():
        flag = False
        for name in names:
            result = value.find(name + "(")
            if result !=-1:
                flag = True
                graph.add_edge(key, name)
    graph.layout(prog='dot')    
    graph.draw("tmp/cfg.png")
```
