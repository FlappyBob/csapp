# Cache lab
*且发一些牢骚*：读并且理解这个题目要求本身就需要对lecture中的内容有较为透彻的理解，看了好多cmu的帮助文档然后稍微复习一下才差不多看懂了这个lab的handout在做什么，不过cmu的讲义真的是相当细了啊，lab讲解也相当的充分。（毕竟网上还有不少其他牛牛讲解 （不是

**收获**
1. markdown要当成习惯写，有时候反复看文档不如自己整理思路写下来（效率要高的多）：白板上或者是md里。
2. 专注：一天能做完的事情趁热打铁。
## handout要求总结。
**第一部分**。
要求：书写csim.c文件，读取valgrind如下格式的trace文件，统计并且展示hit, miss, 和eviction数量。
``` shell
valgrind --log-fd=1 --tool=lackey -v --trace-mem=yes ls -l

> I 0400d7d4, 8 
> M 0421c7f0, 4 
> L 04f6b868, 8 
> S 7ff0005c8, 8
(a) [space] (b) operation address, (c)size
```
./csim-ref 是一个事先写好的二进制文件，我们可以用着玩一玩，了解csim应该怎么写。
```shell 
Usage: ./csim-ref [-hv] -s <num> -E <num> -b <num> -t <file>
Options:
  -h         Print this help message.
  -v         Optional verbose flag.
  -s <num>   Number of set index bits.
  -E <num>   Number of lines per set.
  -b <num>   Number of block offset bits.
  -t <file>  Trace file.

Examples:
  linux>  ./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace
  linux>  ./csim-ref -v -s 8 -E 2 -b 4 -t traces/yi.trace
```

跑一下结果如下：
``` shell
./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace
hits:4 misses:5 evictions:3
./csim-ref -v -s 8 -E 2 -b 4 -t traces/yi.trace
L 10,1 miss 
M 20,1 miss hit 
L 22,1 hit 
S 18,1 hit 
L 110,1 miss 
L 210,1 miss 
M 12,1 hit hit 
hits:5 misses:4 evictions:0
```

## 设计逻辑
我们的头文件需要包含cache统计模型。
```c
i
typedef struct cache_line {
    int valid_bit;
    int tag;  
    int LRUcounter; 
    // since real address are not meant to be stored, real bytes in a cache_line is ignored. 
} Cache_line;
typedef struct cache {
    int S; // number of sets 
    int E; // lines per sets  
    int B; // bytes per line
} Cache;
```

main function logic: 
```c
int main(int argc, char *argv[])
{
    int opt, s, E, b;
    char *t;
    while (-1 != (opt = getopt(argc, argv, "hvs:E:b:t:")))
    {
        switch (opt)
        {
        case 'h':
            printf("h\n");
            break;
        case 'v':
            printf("v\n");
            break;
        case 's':
            s = atoi(optarg);
            break;
        case 'E':
            E = atoi(optarg);
        case 'b':
            b = atoi(optarg);
        case 't':
            t = optarg;
            break;
        default:
            printf("wrong argument\n");
            break;
        }
    }
}
```

