# load理解

* 查看load方式
    * `uptime`
    * `top`

* load含义：
    * 指单位时间内，系统处于可运行状态(R)和不可中断状态(D)的平均进程数，也就是平均活跃进程数；(使用CPU process + wait cpu process + wait i/o process)
    * 可运行状态
    * 不可中断状态: disk sleep;
* load阈值：
    * 70%
    
* 场景

``` yaml
# stress test systems
stress --cpu 1 --timeout 600

# cpu使用率变化情况
mpstat -P ALL 5
20时48分53秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
20时48分58秒  all   50.55    0.00    0.10    0.00    0.00    0.10    0.00    0.00    0.00   49.25
20时48分58秒    0   69.74    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   30.26
20时48分58秒    1   31.33    0.00    0.20    0.00    0.00    0.00    0.00    0.00    0.00   68.47

# 查看具体进程
pidstat -u 1 100
20时50分22秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
20时50分27秒     0         1    0.00    0.20    0.00    0.20     0  systemd
20时50分27秒     0       378    0.00    0.20    0.00    0.20     1  auditd
20时50分27秒     0      2281   98.80    0.00    0.00   98.80     0  stress
20时50分27秒     0      2537    0.00    0.20    0.00    0.20     0  pidstat
20时50分27秒   998      9297   98.61    0.00    0.00   98.61     1  mysqld
20时50分27秒     0     10783    0.20    0.00    0.00    0.20     0  java
20时50分27秒     0     10961    0.20    0.00    0.00    0.20     1  java
20时50分27秒     0     24781    0.20    0.00    0.00    0.20     0  dockerd
20时50分27秒     0     29600    0.60    0.00    0.00    0.60     0  java

---
# stress I/O test
stress -i 1 --timeout 600

# multip process
stress -c 8 --timeout 600
```

* load飙高：
    * cpu密集进程 and I/O wait;