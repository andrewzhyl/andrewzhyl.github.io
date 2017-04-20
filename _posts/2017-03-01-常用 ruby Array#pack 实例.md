---
layout: post
title:  "常用 ruby Array#pack String#unpack 实例"
date:   2017-03-01 09:08:00
description: 'ruby Array#pack》'
category: notes
---

#  常用 ruby Array#pack String#unpack 实例


## m

被 base64编码过的字符串。每隔60个八位组(或在结尾)添加一个换行代码。


``` ruby
["\0"].pack("m") # => "AA==\n"
["\0"].pack("m*") # => "AA==\n"
["\0\0"].pack("m") => "AAA=\n"

#以上等同于
Base64.encode64("\0")

# 不添加换行代码
["\0"].pack("m0") # => "AA==" 

# 以上等同于
Base64.encode64("\0").gsub("\n", "")

```

``` ruby
# 编码
["hello"].pack("m") # => "aGVsbG8=\n" 

# 解码
"aGVsbG8=\n".unpack("m").first # => "hello"
```


## H

