# proxy_pass后的url加不加/的区别

```
# 在nginx中配置proxy_pass时，当在后面的url加上了/，相当于是绝对根路径，则nginx不会把location中匹配的路径部分代理走;
# 如果没有/，则会把匹配的路径部分也给代理走。

下面四种情况分别用http://192.168.1.4/proxy/test.html 进行访问。
第一种：
location  /proxy/ {
          proxy_pass http://127.0.0.1:81/;
}
会被代理到http://127.0.0.1:81/test.html 这个url


第二：(相对于第一种，最后少一个 /)
location  /proxy/ {
          proxy_pass http://127.0.0.1:81;
}
会被代理到http://127.0.0.1:81/proxy/test.html 这个url


第三种：
location  /proxy/ {
          proxy_pass http://127.0.0.1:81/ftlynx/;
}
会被代理到http://127.0.0.1:81/ftlynx/test.html 这个url。


第四种情况(相对于第三种，最后少一个 / )：
location  /proxy/ {
          proxy_pass http://127.0.0.1:81/ftlynx;
}
会被代理到http://127.0.0.1:81/ftlynxtest.html 这个url


 从结果可以看出，应该说分为两种情况才正确。即http://127.0.0.1:81 (上面的第二种) 这种和 http://127.0.0.1:81/.... （上面的第1，3，4种） 这种。
```
