# Multiple process

``` bash
echo "Spawning 100 processes"
for i in {1..100} ;
do
    ( ./my_script & )
; done
```


* https://www.jianshu.com/p/2d60e6513fdd