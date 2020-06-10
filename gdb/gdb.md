## 手工产生core文件
```shell
gdb -p $pid
gcore $corename
quit
```

## gdb中打印结构体指针
```shell
p (*(struct server *)0x1ff3240)

p (*(struct server *)srv)
```

## gdb打印list对象信息
```shell
$ cat plist.gdb

define plist

set $p = proxy

while $p
    printf "%x, %s\n", $p, $p->id
    set $srv = $p->srv

    while $srv
        printf "%x, %s\n", $srv, $srv->id
        set $srv = $srv->next
    end

    set $p = $p->next
end
```
### gdb中执行plist
```shell
source plist.gdb
plist
```
