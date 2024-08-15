* shell定义变量直接使用等号即可与python相同但是等号两边不能有空格python确要求，bat定义变量直接使用set指令
```shell
var="path"
```

```bat
set var=path
```

```python
var = "path"
```

---

* sed匹配字符串

```shell
sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
```

```shell
sed -i 's/#PermitRootLogin*/PermitRootLogin yes/' /etc/ssh/sshd_config
```

`#PermitRootLogin*` 匹配的是 `#PermitRootLogin` 后面跟随零个或多个 `#` 字符的行
`#PermitRootLogin.*` 匹配的是以 `#PermitRootLogin` 开头后面跟着任意字符的行

---
