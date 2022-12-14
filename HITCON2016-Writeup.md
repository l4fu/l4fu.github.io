# HITCON2016-Writeup
## WELCOME

把题目反过来。

## WEB 50 Are You Rich

`verify.php` 处抓包，直接丢 SQLMAP 跑即可。

## WEB 100 Are You Rich 2

仍然利用上面的注入点，抓包，去网上搜一个比特币大于十万的（我搜了个排行榜，只有第一的钱包大于十万。。），放个 Payload：

```
121pKptniA4RsapGFbz3BxsUy3TDFPwvNz-' union select '3Nxwenay9Z8Lc9JBiywExpnEFiLp6Afp8v'
```

我也不知道为什么一定 `-` 才行，加别的都会报错。。。

## WEB 50 Secure Post

Flask 的一个博客？Flask 框架我就知道一个报错出源码和注入。

```

from flask import Flask
import config

# init app
app = Flask(__name__)
app.secret_key = config.flag1
accept_datatype = ['json', 'yaml']

from flask import Response
from flask import request, session
from flask import redirect, url_for, safe_join, abort
from flask import render_template_string

# load utils
def load_eval(data):
    return eval(data)

def load_pickle(data):
    import pickle
    return pickle.loads(data)

def load_json(data):
    import json
    return json.loads(data)

def load_yaml(data):
    import yaml
    return yaml.load(data)

# dump utils
def dump_eval(data):
    return repr(data)

def dump_pickle(data):
    import pickle
    return pickle.dumps(data)

def dump_json(data):
    import json
    return json.dumps(data)

def dump_yaml(data):
    import yaml
    return yaml.dump(data)


def render_template(filename, **args):
    with open(safe_join(app.template_folder, filename)) as f:
        template = f.read()
    name = session.get('name', 'anonymous')[:10]
    return render_template_string(template.format(name=name), **args)

def load_posts():
    handlers = {
        # disabled insecure data type
        #"eval": load_eval,
        #"pickle": load_pickle,

        "json": load_json,
        "yaml": load_yaml
    }

    datatype = session.get("post_type", config.default_datatype)
    data = session.get("post_data", config.default_data)

    if datatype not in handlers: abort(403)
    return handlers[datatype](data)

def store_posts(posts, datatype):
    handlers = {
        "eval": dump_eval,
        "pickle": dump_pickle,

        "json": dump_json,
        "yaml": dump_yaml
    }
    if datatype not in handlers: abort(403)
    data = handlers[datatype](posts)

    session["post_type"] = datatype
    session["post_data"] = data


@app.route('/')
def index():
    posts = load_posts()
    return render_template('index.html', posts = posts, accept_datatype = accept_datatype)

@app.route('/post', methods=['POST'])
def add_post():
    posts = load_posts()

    title = request.form.get('title', 'empty')
    content = request.form.get('content', 'empty')
    datatype = request.form.get('datatype', 'json')
    if datatype not in accept_datatype: abort(403)
    name = request.form.get('author', 'anonymous')[:10]

    from datetime import datetime
    posts.append({
        'title': title,
        'author': name,
        'content': content,
        'date': datetime.now().strftime("%B %d, %Y %X")
    })
    session["name"] = name
    store_posts(posts, datatype)
    return redirect(url_for('index'))

@app.route('/source')
def get_source():
    with open(__file__, "r") as f:
        resp = f.read()
    return Response(resp, mimetype="text/plain")
```

`app.secret_key = config.flag1` 就是目标，我们需要读取 `config`。

参考[乱弹 Flask 注入](http://www.freebuf.com/articles/web/88768.html)。

提交一个 `{{config}}` 即可。

## WEB 100 %%%

这纯粹是一个脑洞题。

访问页面，浏览器提示 HTTPS 证书不合法，点开看一下，发现证书是颁发给域名 `very-secret-area-for-ctf.orange.tw` 的，直接访问并不解析，抓一下原来的包把 `HOST` 字段改了就行了。

## RE 50 Handcrafted pyc

题目给的代码

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import marshal, zlib, base64

exec(marshal.loads(zlib.decompress(base64.b64decode('eJyNVktv00AQXm/eL0igiaFA01IO4cIVCUGFBBJwqRAckLhEIQmtRfPwI0QIeio/hRO/hJ/CiStH2M/prj07diGRP43Hs9+MZ2fWMxbnP6mux+oK9xVMHPFViLdCTB0xkeKDFEFfTIU4E8KZq8dCvB4UlN3hGEsdddXU9QTLv1eFiGKGM4cKUgsFCNLFH7dFrS9poayFYmIZm1b0gyqxMOwJaU3r6xs9sW1ooakXuRv+un7Q0sIlLVzOCZq/XtsK2oTSYaZlStogXi1HV0iazoN2CV2HZeXqRQ54TlJRb7FUlKyUatISsdzo+P7UU1Gb1POdMruckepGwk9tIXQTftz2yBaT5JQovWvpSa6poJPuqgao+b9l5Aj/R+mLQIP4f6Q8Vb3g/5TB/TJxWGdZr9EQrmn99fwKtTvAZGU7wzS7GNpZpDm2JgCrr8wrmPoo54UqGampFIeS9ojXjc4E2yI06bq/4DRoUAc0nVnng4k6p7Ks0+j/S8z9V+NZ5dhmrJUM/y7JTJeRtnJ2TSYJvsFq3CQt/vnfqmQXt5KlpuRcIvDAmhnn2E0t9BJ3SvB/SfLWhuOWNiNVZ+h28g4wlwUp00w95si43rZ3r6+fUIEdgOZbQAsyFRRvBR6dla8KCzRdslar7WS+a5HFb39peIAmG7uZTHVm17Czxju4m6bayz8e7J40DzqM0jr0bmv9PmPvk6y5z57HU8wdTDHeiUJvBMAM4+0CpoAZ4BPgJeAYEAHmgAUgAHiAj4AVAGORtwd4AVgC3gEmgBBwCPgMWANOAQ8AbwBHgHuAp4D3gLuARwoGmNUizF/j4yDC5BWM1kNvvlxFA8xikRrBxHIUhutFMBlgQoshhPphGAXe/OggKqqb2cibxwuEXjUcQjccxi5eFRL1fDSbKrUhy2CMb2aLyepkegDWsBwPlrVC0/kLHmeCBQ=='))))
```

load 进来，看看变量。

```
>>> a = marshal.loads(zlib.decompress(base64.b64decode('eJyNVktv00AQXm/eL0igiaFA01IO4cIVCUGFBBJwqRAckLhEIQmtRfPwI0QIeio/hRO/hJ/CiStH2M/prj07diGRP43Hs9+MZ2fWMxbnP6mux+oK9xVMHPFViLdCTB0xkeKDFEFfTIU4E8KZq8dCvB4UlN3hGEsdddXU9QTLv1eFiGKGM4cKUgsFCNLFH7dFrS9poayFYmIZm1b0gyqxMOwJaU3r6xs9sW1ooakXuRv+un7Q0sIlLVzOCZq/XtsK2oTSYaZlStogXi1HV0iazoN2CV2HZeXqRQ54TlJRb7FUlKyUatISsdzo+P7UU1Gb1POdMruckepGwk9tIXQTftz2yBaT5JQovWvpSa6poJPuqgao+b9l5Aj/R+mLQIP4f6Q8Vb3g/5TB/TJxWGdZr9EQrmn99fwKtTvAZGU7wzS7GNpZpDm2JgCrr8wrmPoo54UqGampFIeS9ojXjc4E2yI06bq/4DRoUAc0nVnng4k6p7Ks0+j/S8z9V+NZ5dhmrJUM/y7JTJeRtnJ2TSYJvsFq3CQt/vnfqmQXt5KlpuRcIvDAmhnn2E0t9BJ3SvB/SfLWhuOWNiNVZ+h28g4wlwUp00w95si43rZ3r6+fUIEdgOZbQAsyFRRvBR6dla8KCzRdslar7WS+a5HFb39peIAmG7uZTHVm17Czxju4m6bayz8e7J40DzqM0jr0bmv9PmPvk6y5z57HU8wdTDHeiUJvBMAM4+0CpoAZ4BPgJeAYEAHmgAUgAHiAj4AVAGORtwd4AVgC3gEmgBBwCPgMWANOAQ8AbwBHgHuAp4D3gLuARwoGmNUizF/j4yDC5BWM1kNvvlxFA8xikRrBxHIUhutFMBlgQoshhPphGAXe/OggKqqb2cibxwuEXjUcQjccxi5eFRL1fDSbKrUhy2CMb2aLyepkegDWsBwPlrVC0/kLHmeCBQ==')))
>>> type(a)
<type 'code'>
>>> dir(a)
['__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']
>>> a.co_consts
(None, <code object main at 0x7fca8f844e30, file "<string>", line 1>, '__main__')
>>> a.co_code
'd\x01\x00\x84\x00\x00Z\x00\x00e\x01\x00d\x02\x00k\x02\x00r\x1f\x00e\x00\x00\x83\x00\x00\x01n\x00\x00d\x00\x00S'
>>> a.co_consts[1]
<code object main at 0x7fca8f844e30, file "<string>", line 1>
```

可以看出，原本的字节码里还包含着一块字节码，把这一块东西用 `marshal.dump` 弄出来，然后再反编译看内容。

```
  1           0 LOAD_GLOBAL              0 (chr)
              3 LOAD_CONST               1 (108)
              6 CALL_FUNCTION            1
              9 LOAD_GLOBAL              0 (chr)
             12 LOAD_CONST               1 (108)
             15 CALL_FUNCTION            1
             18 LOAD_GLOBAL              0 (chr)
             21 LOAD_CONST               2 (97)
             24 CALL_FUNCTION            1
             27 LOAD_GLOBAL              0 (chr)
             30 LOAD_CONST               3 (67)
             33 CALL_FUNCTION            1
             36 ROT_TWO             
             37 BINARY_ADD          
             38 ROT_TWO             
             39 BINARY_ADD          
             40 ROT_TWO             
             41 BINARY_ADD          
             42 LOAD_GLOBAL              0 (chr)
             45 LOAD_CONST               4 (32)
             48 CALL_FUNCTION            1
             51 LOAD_GLOBAL              0 (chr)
             54 LOAD_CONST               5 (101)
             57 CALL_FUNCTION            1
             60 LOAD_GLOBAL              0 (chr)
             63 LOAD_CONST               6 (109)
             66 CALL_FUNCTION            1
             69 LOAD_GLOBAL              0 (chr)
             72 LOAD_CONST               4 (32)
             75 CALL_FUNCTION            1
             78 ROT_TWO             
             79 BINARY_ADD          
             80 ROT_TWO             
             81 BINARY_ADD          
             82 ROT_TWO             
             83 BINARY_ADD          
             84 BINARY_ADD          
             85 LOAD_GLOBAL              0 (chr)
             88 LOAD_CONST               7 (121)
             91 CALL_FUNCTION            1
             94 LOAD_GLOBAL              0 (chr)
             97 LOAD_CONST               8 (80)
            100 CALL_FUNCTION            1
            103 LOAD_GLOBAL              0 (chr)
            106 LOAD_CONST               4 (32)
            109 CALL_FUNCTION            1
            112 LOAD_GLOBAL              0 (chr)
            115 LOAD_CONST               2 (97)
            118 CALL_FUNCTION            1
            121 ROT_TWO             
            122 BINARY_ADD          
            123 ROT_TWO             
            124 BINARY_ADD          
            125 ROT_TWO             
            126 BINARY_ADD          
            127 LOAD_GLOBAL              0 (chr)
            130 LOAD_CONST               9 (104)
            133 CALL_FUNCTION            1
            136 LOAD_GLOBAL              0 (chr)
            139 LOAD_CONST              10 (116)
            142 CALL_FUNCTION            1
            145 ROT_TWO             
            146 BINARY_ADD          
            147 LOAD_GLOBAL              0 (chr)
            150 LOAD_CONST               4 (32)
            153 CALL_FUNCTION            1
            156 LOAD_GLOBAL              0 (chr)
            159 LOAD_CONST              11 (110)
            162 CALL_FUNCTION            1
            165 LOAD_GLOBAL              0 (chr)
            168 LOAD_CONST              12 (111)
            171 CALL_FUNCTION            1
            174 ROT_TWO             
            175 BINARY_ADD          
            176 ROT_TWO             
            177 BINARY_ADD          
            178 BINARY_ADD          
            179 BINARY_ADD          
            180 BINARY_ADD          
            181 LOAD_GLOBAL              0 (chr)
            184 LOAD_CONST              10 (116)
            187 CALL_FUNCTION            1
            190 LOAD_GLOBAL              0 (chr)
            193 LOAD_CONST              13 (114)
            196 CALL_FUNCTION            1
            199 LOAD_GLOBAL              0 (chr)
            202 LOAD_CONST              14 (105)
            205 CALL_FUNCTION            1
            208 LOAD_GLOBAL              0 (chr)
            211 LOAD_CONST              15 (118)
            214 CALL_FUNCTION            1
            217 ROT_TWO             
            218 BINARY_ADD          
            219 ROT_TWO             
            220 BINARY_ADD          
            221 ROT_TWO             
            222 BINARY_ADD          
            223 LOAD_GLOBAL              0 (chr)
            226 LOAD_CONST               4 (32)
            229 CALL_FUNCTION            1
            232 LOAD_GLOBAL              0 (chr)
            235 LOAD_CONST               1 (108)
            238 CALL_FUNCTION            1
            241 LOAD_GLOBAL              0 (chr)
            244 LOAD_CONST               2 (97)
            247 CALL_FUNCTION            1
            250 LOAD_GLOBAL              0 (chr)
            253 LOAD_CONST              16 (117)
            256 CALL_FUNCTION            1
            259 ROT_TWO             
            260 BINARY_ADD          
            261 ROT_TWO             
            262 BINARY_ADD          
            263 ROT_TWO             
            264 BINARY_ADD          
            265 BINARY_ADD          
            266 LOAD_GLOBAL              0 (chr)
            269 LOAD_CONST               9 (104)
            272 CALL_FUNCTION            1
            275 LOAD_GLOBAL              0 (chr)
            278 LOAD_CONST              17 (99)
            281 CALL_FUNCTION            1
            284 LOAD_GLOBAL              0 (chr)
            287 LOAD_CONST               2 (97)
            290 CALL_FUNCTION            1
            293 LOAD_GLOBAL              0 (chr)
            296 LOAD_CONST               6 (109)
            299 CALL_FUNCTION            1
            302 ROT_TWO             
            303 BINARY_ADD          
            304 ROT_TWO             
            305 BINARY_ADD          
            306 ROT_TWO             
            307 BINARY_ADD          
            308 LOAD_GLOBAL              0 (chr)
            311 LOAD_CONST              11 (110)
            314 CALL_FUNCTION            1
            317 LOAD_GLOBAL              0 (chr)
            320 LOAD_CONST              14 (105)
            323 CALL_FUNCTION            1
            326 ROT_TWO             
            327 BINARY_ADD          
            328 LOAD_GLOBAL              0 (chr)
            331 LOAD_CONST               4 (32)
            334 CALL_FUNCTION            1
            337 LOAD_GLOBAL              0 (chr)
            340 LOAD_CONST              18 (33)
            343 CALL_FUNCTION            1
            346 LOAD_GLOBAL              0 (chr)
            349 LOAD_CONST               5 (101)
            352 CALL_FUNCTION            1
            355 ROT_TWO             
            356 BINARY_ADD          
            357 ROT_TWO             
            358 BINARY_ADD          
            359 BINARY_ADD          
            360 BINARY_ADD          
            361 BINARY_ADD          
            362 BINARY_ADD          
            363 LOAD_GLOBAL              0 (chr)
            366 LOAD_CONST               2 (97)
            369 CALL_FUNCTION            1
            372 LOAD_GLOBAL              0 (chr)
            375 LOAD_CONST              17 (99)
            378 CALL_FUNCTION            1
            381 LOAD_GLOBAL              0 (chr)
            384 LOAD_CONST               4 (32)
            387 CALL_FUNCTION            1
            390 LOAD_GLOBAL              0 (chr)
            393 LOAD_CONST              19 (73)
            396 CALL_FUNCTION            1
            399 ROT_TWO             
            400 BINARY_ADD          
            401 ROT_TWO             
            402 BINARY_ADD          
            403 ROT_TWO             
            404 BINARY_ADD          
            405 LOAD_GLOBAL              0 (chr)
            408 LOAD_CONST              11 (110)
            411 CALL_FUNCTION            1
            414 LOAD_GLOBAL              0 (chr)
            417 LOAD_CONST              14 (105)
            420 CALL_FUNCTION            1
            423 LOAD_GLOBAL              0 (chr)
            426 LOAD_CONST               4 (32)
            429 CALL_FUNCTION            1
            432 LOAD_GLOBAL              0 (chr)
            435 LOAD_CONST              11 (110)
            438 CALL_FUNCTION            1
            441 ROT_TWO             
            442 BINARY_ADD          
            443 ROT_TWO             
            444 BINARY_ADD          
            445 ROT_TWO             
            446 BINARY_ADD          
            447 BINARY_ADD          
            448 LOAD_GLOBAL              0 (chr)
            451 LOAD_CONST              20 (112)
            454 CALL_FUNCTION            1
            457 LOAD_GLOBAL              0 (chr)
            460 LOAD_CONST              13 (114)
            463 CALL_FUNCTION            1
            466 LOAD_GLOBAL              0 (chr)
            469 LOAD_CONST               5 (101)
            472 CALL_FUNCTION            1
            475 LOAD_GLOBAL              0 (chr)
            478 LOAD_CONST              10 (116)
            481 CALL_FUNCTION            1
            484 ROT_TWO             
            485 BINARY_ADD          
            486 ROT_TWO             
            487 BINARY_ADD          
            488 ROT_TWO             
            489 BINARY_ADD          
            490 LOAD_GLOBAL              0 (chr)
            493 LOAD_CONST               5 (101)
            496 CALL_FUNCTION            1
            499 LOAD_GLOBAL              0 (chr)
            502 LOAD_CONST              13 (114)
            505 CALL_FUNCTION            1
            508 ROT_TWO             
            509 BINARY_ADD          
            510 LOAD_GLOBAL              0 (chr)
            513 LOAD_CONST               8 (80)
            516 CALL_FUNCTION            1
            519 LOAD_GLOBAL              0 (chr)
            522 LOAD_CONST               4 (32)
            525 CALL_FUNCTION            1
            528 LOAD_GLOBAL              0 (chr)
            531 LOAD_CONST              10 (116)
            534 CALL_FUNCTION            1
            537 ROT_TWO             
            538 BINARY_ADD          
            539 ROT_TWO             
            540 BINARY_ADD          
            541 BINARY_ADD          
            542 BINARY_ADD          
            543 BINARY_ADD          
            544 LOAD_GLOBAL              0 (chr)
            547 LOAD_CONST              12 (111)
            550 CALL_FUNCTION            1
            553 LOAD_GLOBAL              0 (chr)
            556 LOAD_CONST               9 (104)
            559 CALL_FUNCTION            1
            562 LOAD_GLOBAL              0 (chr)
            565 LOAD_CONST              10 (116)
            568 CALL_FUNCTION            1
            571 LOAD_GLOBAL              0 (chr)
            574 LOAD_CONST               7 (121)
            577 CALL_FUNCTION            1
            580 ROT_TWO             
            581 BINARY_ADD          
            582 ROT_TWO             
            583 BINARY_ADD          
            584 ROT_TWO             
            585 BINARY_ADD          
            586 LOAD_GLOBAL              0 (chr)
            589 LOAD_CONST               4 (32)
            592 CALL_FUNCTION            1
            595 LOAD_GLOBAL              0 (chr)
            598 LOAD_CONST              11 (110)
            601 CALL_FUNCTION            1
            604 ROT_TWO             
            605 BINARY_ADD          
            606 LOAD_GLOBAL              0 (chr)
            609 LOAD_CONST              10 (116)
            612 CALL_FUNCTION            1
            615 LOAD_GLOBAL              0 (chr)
            618 LOAD_CONST               7 (121)
            621 CALL_FUNCTION            1
            624 LOAD_GLOBAL              0 (chr)
            627 LOAD_CONST              21 (98)
            630 CALL_FUNCTION            1
            633 ROT_TWO             
            634 BINARY_ADD          
            635 ROT_TWO             
            636 BINARY_ADD          
            637 BINARY_ADD          
            638 BINARY_ADD          
            639 LOAD_GLOBAL              0 (chr)
            642 LOAD_CONST              22 (100)
            645 CALL_FUNCTION            1
            648 LOAD_GLOBAL              0 (chr)
            651 LOAD_CONST              12 (111)
            654 CALL_FUNCTION            1
            657 LOAD_GLOBAL              0 (chr)
            660 LOAD_CONST              17 (99)
            663 CALL_FUNCTION            1
            666 LOAD_GLOBAL              0 (chr)
            669 LOAD_CONST               5 (101)
            672 CALL_FUNCTION            1
            675 ROT_TWO             
            676 BINARY_ADD          
            677 ROT_TWO             
            678 BINARY_ADD          
            679 ROT_TWO             
            680 BINARY_ADD          
            681 LOAD_GLOBAL              0 (chr)
            684 LOAD_CONST              23 (115)
            687 CALL_FUNCTION            1
            690 LOAD_GLOBAL              0 (chr)
            693 LOAD_CONST               5 (101)
            696 CALL_FUNCTION            1
            699 ROT_TWO             
            700 BINARY_ADD          
            701 LOAD_GLOBAL              0 (chr)
            704 LOAD_CONST              18 (33)
            707 CALL_FUNCTION            1
            710 LOAD_GLOBAL              0 (chr)
            713 LOAD_CONST              18 (33)
            716 CALL_FUNCTION            1
            719 LOAD_GLOBAL              0 (chr)
            722 LOAD_CONST              18 (33)
            725 CALL_FUNCTION            1
            728 ROT_TWO             
            729 BINARY_ADD          
            730 ROT_TWO             
            731 BINARY_ADD          
            732 BINARY_ADD          
            733 BINARY_ADD          
            734 BINARY_ADD          
            735 BINARY_ADD          
            736 BINARY_ADD          
            737 LOAD_CONST               0 (None)
            740 NOP                 
            741 JUMP_ABSOLUTE          759
        >>  744 LOAD_GLOBAL              1 (raw_input)
            747 JUMP_ABSOLUTE         1480
        >>  750 LOAD_FAST                0 (password)
            753 COMPARE_OP               2 (==)
            756 JUMP_ABSOLUTE          767
        >>  759 ROT_TWO             
            760 STORE_FAST               0 (password)
            763 POP_TOP             
            764 JUMP_ABSOLUTE          744
        >>  767 POP_JUMP_IF_FALSE     1591
            770 LOAD_GLOBAL              0 (chr)
            773 LOAD_CONST              17 (99)
            776 CALL_FUNCTION            1
            779 LOAD_GLOBAL              0 (chr)
            782 LOAD_CONST              10 (116)
            785 CALL_FUNCTION            1
            788 LOAD_GLOBAL              0 (chr)
            791 LOAD_CONST              14 (105)
            794 CALL_FUNCTION            1
            797 LOAD_GLOBAL              0 (chr)
            800 LOAD_CONST               9 (104)
            803 CALL_FUNCTION            1
            806 ROT_TWO             
            807 BINARY_ADD          
            808 ROT_TWO             
            809 BINARY_ADD          
            810 ROT_TWO             
            811 BINARY_ADD          
            812 LOAD_GLOBAL              0 (chr)
            815 LOAD_CONST              24 (78)
            818 CALL_FUNCTION            1
            821 LOAD_GLOBAL              0 (chr)
            824 LOAD_CONST              25 (123)
            827 CALL_FUNCTION            1
            830 LOAD_GLOBAL              0 (chr)
            833 LOAD_CONST              11 (110)
            836 CALL_FUNCTION            1
            839 LOAD_GLOBAL              0 (chr)
            842 LOAD_CONST              12 (111)
            845 CALL_FUNCTION            1
            848 ROT_TWO             
            849 BINARY_ADD          
            850 ROT_TWO             
            851 BINARY_ADD          
            852 ROT_TWO             
            853 BINARY_ADD          
            854 BINARY_ADD          
            855 LOAD_GLOBAL              0 (chr)
            858 LOAD_CONST               7 (121)
            861 CALL_FUNCTION            1
            864 LOAD_GLOBAL              0 (chr)
            867 LOAD_CONST               4 (32)
            870 CALL_FUNCTION            1
            873 LOAD_GLOBAL              0 (chr)
            876 LOAD_CONST              26 (119)
            879 CALL_FUNCTION            1
            882 LOAD_GLOBAL              0 (chr)
            885 LOAD_CONST              12 (111)
            888 CALL_FUNCTION            1
            891 ROT_TWO             
            892 BINARY_ADD          
            893 ROT_TWO             
            894 BINARY_ADD          
            895 ROT_TWO             
            896 BINARY_ADD          
            897 LOAD_GLOBAL              0 (chr)
            900 LOAD_CONST              17 (99)
            903 CALL_FUNCTION            1
            906 LOAD_GLOBAL              0 (chr)
            909 LOAD_CONST               4 (32)
            912 CALL_FUNCTION            1
            915 LOAD_GLOBAL              0 (chr)
            918 LOAD_CONST              16 (117)
            921 CALL_FUNCTION            1
            924 LOAD_GLOBAL              0 (chr)
            927 LOAD_CONST              12 (111)
            930 CALL_FUNCTION            1
            933 ROT_TWO             
            934 BINARY_ADD          
            935 ROT_TWO             
            936 BINARY_ADD          
            937 ROT_TWO             
            938 BINARY_ADD          
            939 BINARY_ADD          
            940 BINARY_ADD          
            941 LOAD_GLOBAL              0 (chr)
            944 LOAD_CONST              17 (99)
            947 CALL_FUNCTION            1
            950 LOAD_GLOBAL              0 (chr)
            953 LOAD_CONST               4 (32)
            956 CALL_FUNCTION            1
            959 LOAD_GLOBAL              0 (chr)
            962 LOAD_CONST              11 (110)
            965 CALL_FUNCTION            1
            968 LOAD_GLOBAL              0 (chr)
            971 LOAD_CONST               2 (97)
            974 CALL_FUNCTION            1
            977 ROT_TWO             
            978 BINARY_ADD          
            979 ROT_TWO             
            980 BINARY_ADD          
            981 ROT_TWO             
            982 BINARY_ADD          
            983 LOAD_GLOBAL              0 (chr)
            986 LOAD_CONST              14 (105)
            989 CALL_FUNCTION            1
            992 LOAD_GLOBAL              0 (chr)
            995 LOAD_CONST              20 (112)
            998 CALL_FUNCTION            1
           1001 LOAD_GLOBAL              0 (chr)
           1004 LOAD_CONST               6 (109)
           1007 CALL_FUNCTION            1
           1010 LOAD_GLOBAL              0 (chr)
           1013 LOAD_CONST              12 (111)
           1016 CALL_FUNCTION            1
           1019 ROT_TWO             
           1020 BINARY_ADD          
           1021 ROT_TWO             
           1022 BINARY_ADD          
           1023 ROT_TWO             
           1024 BINARY_ADD          
           1025 BINARY_ADD          
           1026 LOAD_GLOBAL              0 (chr)
           1029 LOAD_CONST               2 (97)
           1032 CALL_FUNCTION            1
           1035 LOAD_GLOBAL              0 (chr)
           1038 LOAD_CONST               4 (32)
           1041 CALL_FUNCTION            1
           1044 LOAD_GLOBAL              0 (chr)
           1047 LOAD_CONST               5 (101)
           1050 CALL_FUNCTION            1
           1053 LOAD_GLOBAL              0 (chr)
           1056 LOAD_CONST               1 (108)
           1059 CALL_FUNCTION            1
           1062 ROT_TWO             
           1063 BINARY_ADD          
           1064 ROT_TWO             
           1065 BINARY_ADD          
           1066 ROT_TWO             
           1067 BINARY_ADD          
           1068 LOAD_GLOBAL              0 (chr)
           1071 LOAD_CONST              22 (100)
           1074 CALL_FUNCTION            1
           1077 LOAD_GLOBAL              0 (chr)
           1080 LOAD_CONST              11 (110)
           1083 CALL_FUNCTION            1
           1086 ROT_TWO             
           1087 BINARY_ADD          
           1088 LOAD_GLOBAL              0 (chr)
           1091 LOAD_CONST              16 (117)
           1094 CALL_FUNCTION            1
           1097 LOAD_GLOBAL              0 (chr)
           1100 LOAD_CONST              13 (114)
           1103 CALL_FUNCTION            1
           1106 LOAD_GLOBAL              0 (chr)
           1109 LOAD_CONST               4 (32)
           1112 CALL_FUNCTION            1
           1115 ROT_TWO             
           1116 BINARY_ADD          
           1117 ROT_TWO             
           1118 BINARY_ADD          
           1119 BINARY_ADD          
           1120 BINARY_ADD          
           1121 BINARY_ADD          
           1122 BINARY_ADD          
           1123 LOAD_GLOBAL              0 (chr)
           1126 LOAD_CONST               7 (121)
           1129 CALL_FUNCTION            1
           1132 LOAD_GLOBAL              0 (chr)
           1135 LOAD_CONST               8 (80)
           1138 CALL_FUNCTION            1
           1141 LOAD_GLOBAL              0 (chr)
           1144 LOAD_CONST               4 (32)
           1147 CALL_FUNCTION            1
           1150 LOAD_GLOBAL              0 (chr)
           1153 LOAD_CONST              11 (110)
           1156 CALL_FUNCTION            1
           1159 ROT_TWO             
           1160 BINARY_ADD          
           1161 ROT_TWO             
           1162 BINARY_ADD          
           1163 ROT_TWO             
           1164 BINARY_ADD          
           1165 LOAD_GLOBAL              0 (chr)
           1168 LOAD_CONST              11 (110)
           1171 CALL_FUNCTION            1
           1174 LOAD_GLOBAL              0 (chr)
           1177 LOAD_CONST              12 (111)
           1180 CALL_FUNCTION            1
           1183 LOAD_GLOBAL              0 (chr)
           1186 LOAD_CONST               9 (104)
           1189 CALL_FUNCTION            1
           1192 LOAD_GLOBAL              0 (chr)
           1195 LOAD_CONST              10 (116)
           1198 CALL_FUNCTION            1
           1201 ROT_TWO             
           1202 BINARY_ADD          
           1203 ROT_TWO             
           1204 BINARY_ADD          
           1205 ROT_TWO             
           1206 BINARY_ADD          
           1207 BINARY_ADD          
           1208 LOAD_GLOBAL              0 (chr)
           1211 LOAD_CONST              10 (116)
           1214 CALL_FUNCTION            1
           1217 LOAD_GLOBAL              0 (chr)
           1220 LOAD_CONST               7 (121)
           1223 CALL_FUNCTION            1
           1226 LOAD_GLOBAL              0 (chr)
           1229 LOAD_CONST              21 (98)
           1232 CALL_FUNCTION            1
           1235 LOAD_GLOBAL              0 (chr)
           1238 LOAD_CONST               4 (32)
           1241 CALL_FUNCTION            1
           1244 ROT_TWO             
           1245 BINARY_ADD          
           1246 ROT_TWO             
           1247 BINARY_ADD          
           1248 ROT_TWO             
           1249 BINARY_ADD          
           1250 LOAD_GLOBAL              0 (chr)
           1253 LOAD_CONST              22 (100)
           1256 CALL_FUNCTION            1
           1259 LOAD_GLOBAL              0 (chr)
           1262 LOAD_CONST              12 (111)
           1265 CALL_FUNCTION            1
           1268 LOAD_GLOBAL              0 (chr)
           1271 LOAD_CONST              17 (99)
           1274 CALL_FUNCTION            1
           1277 LOAD_GLOBAL              0 (chr)
           1280 LOAD_CONST               5 (101)
           1283 CALL_FUNCTION            1
           1286 ROT_TWO             
           1287 BINARY_ADD          
           1288 ROT_TWO             
           1289 BINARY_ADD          
           1290 ROT_TWO             
           1291 BINARY_ADD          
           1292 BINARY_ADD          
           1293 BINARY_ADD          
           1294 LOAD_GLOBAL              0 (chr)
           1297 LOAD_CONST              11 (110)
           1300 CALL_FUNCTION            1
           1303 LOAD_GLOBAL              0 (chr)
           1306 LOAD_CONST              14 (105)
           1309 CALL_FUNCTION            1
           1312 LOAD_GLOBAL              0 (chr)
           1315 LOAD_CONST               4 (32)
           1318 CALL_FUNCTION            1
           1321 LOAD_GLOBAL              0 (chr)
           1324 LOAD_CONST               5 (101)
           1327 CALL_FUNCTION            1
           1330 ROT_TWO             
           1331 BINARY_ADD          
           1332 ROT_TWO             
           1333 BINARY_ADD          
           1334 ROT_TWO             
           1335 BINARY_ADD          
           1336 LOAD_GLOBAL              0 (chr)
           1339 LOAD_CONST              16 (117)
           1342 CALL_FUNCTION            1
           1345 LOAD_GLOBAL              0 (chr)
           1348 LOAD_CONST              12 (111)
           1351 CALL_FUNCTION            1
           1354 LOAD_GLOBAL              0 (chr)
           1357 LOAD_CONST               7 (121)
           1360 CALL_FUNCTION            1
           1363 LOAD_GLOBAL              0 (chr)
           1366 LOAD_CONST               4 (32)
           1369 CALL_FUNCTION            1
           1372 ROT_TWO             
           1373 BINARY_ADD          
           1374 ROT_TWO             
           1375 BINARY_ADD          
           1376 ROT_TWO             
           1377 BINARY_ADD          
           1378 BINARY_ADD          
           1379 LOAD_GLOBAL              0 (chr)
           1382 LOAD_CONST              13 (114)
           1385 CALL_FUNCTION            1
           1388 LOAD_GLOBAL              0 (chr)
           1391 LOAD_CONST              21 (98)
           1394 CALL_FUNCTION            1
           1397 LOAD_GLOBAL              0 (chr)
           1400 LOAD_CONST               4 (32)
           1403 CALL_FUNCTION            1
           1406 LOAD_GLOBAL              0 (chr)
           1409 LOAD_CONST              13 (114)
           1412 CALL_FUNCTION            1
           1415 ROT_TWO             
           1416 BINARY_ADD          
           1417 ROT_TWO             
           1418 BINARY_ADD          
           1419 ROT_TWO             
           1420 BINARY_ADD          
           1421 LOAD_GLOBAL              0 (chr)
           1424 LOAD_CONST              14 (105)
           1427 CALL_FUNCTION            1
           1430 LOAD_GLOBAL              0 (chr)
           1433 LOAD_CONST               2 (97)
           1436 CALL_FUNCTION            1
           1439 ROT_TWO             
           1440 BINARY_ADD          
           1441 LOAD_GLOBAL              0 (chr)
           1444 LOAD_CONST              27 (125)
           1447 CALL_FUNCTION            1
           1450 LOAD_GLOBAL              0 (chr)
           1453 LOAD_CONST              18 (33)
           1456 CALL_FUNCTION            1
           1459 LOAD_GLOBAL              0 (chr)
           1462 LOAD_CONST              11 (110)
           1465 CALL_FUNCTION            1
           1468 ROT_TWO             
           1469 BINARY_ADD          
           1470 ROT_TWO             
           1471 BINARY_ADD          
           1472 BINARY_ADD          
           1473 BINARY_ADD          
           1474 BINARY_ADD          
           1475 BINARY_ADD          
           1476 BINARY_ADD          
           1477 JUMP_ABSOLUTE         2212
        >> 1480 LOAD_GLOBAL              0 (chr)
           1483 LOAD_CONST               2 (97)
           1486 CALL_FUNCTION            1
           1489 LOAD_GLOBAL              0 (chr)
           1492 LOAD_CONST              20 (112)
           1495 CALL_FUNCTION            1
           1498 ROT_TWO             
           1499 BINARY_ADD          
           1500 LOAD_GLOBAL              0 (chr)
           1503 LOAD_CONST              26 (119)
           1506 CALL_FUNCTION            1
           1509 LOAD_GLOBAL              0 (chr)
           1512 LOAD_CONST              23 (115)
           1515 CALL_FUNCTION            1
           1518 LOAD_GLOBAL              0 (chr)
           1521 LOAD_CONST              23 (115)
           1524 CALL_FUNCTION            1
           1527 ROT_TWO             
           1528 BINARY_ADD          
           1529 ROT_TWO             
           1530 BINARY_ADD          
           1531 BINARY_ADD          
           1532 LOAD_GLOBAL              0 (chr)
           1535 LOAD_CONST              13 (114)
           1538 CALL_FUNCTION            1
           1541 LOAD_GLOBAL              0 (chr)
           1544 LOAD_CONST              12 (111)
           1547 CALL_FUNCTION            1
           1550 ROT_TWO             
           1551 BINARY_ADD          
           1552 LOAD_GLOBAL              0 (chr)
           1555 LOAD_CONST               4 (32)
           1558 CALL_FUNCTION            1
           1561 LOAD_GLOBAL              0 (chr)
           1564 LOAD_CONST              28 (58)
           1567 CALL_FUNCTION            1
           1570 LOAD_GLOBAL              0 (chr)
           1573 LOAD_CONST              22 (100)
           1576 CALL_FUNCTION            1
           1579 ROT_TWO             
           1580 BINARY_ADD          
           1581 ROT_TWO             
           1582 BINARY_ADD          
           1583 BINARY_ADD          
           1584 BINARY_ADD          
           1585 CALL_FUNCTION            1
           1588 JUMP_ABSOLUTE          750
        >> 1591 LOAD_GLOBAL              0 (chr)
           1594 LOAD_CONST              12 (111)
           1597 CALL_FUNCTION            1
           1600 LOAD_GLOBAL              0 (chr)
           1603 LOAD_CONST              13 (114)
           1606 CALL_FUNCTION            1
           1609 LOAD_GLOBAL              0 (chr)
           1612 LOAD_CONST              29 (87)
           1615 CALL_FUNCTION            1
           1618 ROT_TWO             
           1619 BINARY_ADD          
           1620 ROT_TWO             
           1621 BINARY_ADD          
           1622 LOAD_GLOBAL              0 (chr)
           1625 LOAD_CONST              20 (112)
           1628 CALL_FUNCTION            1
           1631 LOAD_GLOBAL              0 (chr)
           1634 LOAD_CONST               4 (32)
           1637 CALL_FUNCTION            1
           1640 LOAD_GLOBAL              0 (chr)
           1643 LOAD_CONST              30 (103)
           1646 CALL_FUNCTION            1
           1649 LOAD_GLOBAL              0 (chr)
           1652 LOAD_CONST              11 (110)
           1655 CALL_FUNCTION            1
           1658 ROT_TWO             
           1659 BINARY_ADD          
           1660 ROT_TWO             
           1661 BINARY_ADD          
           1662 ROT_TWO             
           1663 BINARY_ADD          
           1664 BINARY_ADD          
           1665 LOAD_GLOBAL              0 (chr)
           1668 LOAD_CONST              23 (115)
           1671 CALL_FUNCTION            1
           1674 LOAD_GLOBAL              0 (chr)
           1677 LOAD_CONST              23 (115)
           1680 CALL_FUNCTION            1
           1683 LOAD_GLOBAL              0 (chr)
           1686 LOAD_CONST               2 (97)
           1689 CALL_FUNCTION            1
           1692 ROT_TWO             
           1693 BINARY_ADD          
           1694 ROT_TWO             
           1695 BINARY_ADD          
           1696 LOAD_GLOBAL              0 (chr)
           1699 LOAD_CONST              22 (100)
           1702 CALL_FUNCTION            1
           1705 LOAD_GLOBAL              0 (chr)
           1708 LOAD_CONST              13 (114)
           1711 CALL_FUNCTION            1
           1714 LOAD_GLOBAL              0 (chr)
           1717 LOAD_CONST              12 (111)
           1720 CALL_FUNCTION            1
           1723 LOAD_GLOBAL              0 (chr)
           1726 LOAD_CONST              26 (119)
           1729 CALL_FUNCTION            1
           1732 ROT_TWO             
           1733 BINARY_ADD          
           1734 ROT_TWO             
           1735 BINARY_ADD          
           1736 ROT_TWO             
           1737 BINARY_ADD          
           1738 BINARY_ADD          
           1739 BINARY_ADD          
           1740 LOAD_GLOBAL              0 (chr)
           1743 LOAD_CONST              31 (46)
           1746 CALL_FUNCTION            1
           1749 LOAD_GLOBAL              0 (chr)
           1752 LOAD_CONST              31 (46)
           1755 CALL_FUNCTION            1
           1758 LOAD_GLOBAL              0 (chr)
           1761 LOAD_CONST              31 (46)
           1764 CALL_FUNCTION            1
           1767 ROT_TWO             
           1768 BINARY_ADD          
           1769 ROT_TWO             
           1770 BINARY_ADD          
           1771 LOAD_GLOBAL              0 (chr)
           1774 LOAD_CONST               5 (101)
           1777 CALL_FUNCTION            1
           1780 LOAD_GLOBAL              0 (chr)
           1783 LOAD_CONST               1 (108)
           1786 CALL_FUNCTION            1
           1789 LOAD_GLOBAL              0 (chr)
           1792 LOAD_CONST               8 (80)
           1795 CALL_FUNCTION            1
           1798 LOAD_GLOBAL              0 (chr)
           1801 LOAD_CONST               4 (32)
           1804 CALL_FUNCTION            1
           1807 ROT_TWO             
           1808 BINARY_ADD          
           1809 ROT_TWO             
           1810 BINARY_ADD          
           1811 ROT_TWO             
           1812 BINARY_ADD          
           1813 BINARY_ADD          
           1814 LOAD_GLOBAL              0 (chr)
           1817 LOAD_CONST               4 (32)
           1820 CALL_FUNCTION            1
           1823 LOAD_GLOBAL              0 (chr)
           1826 LOAD_CONST               5 (101)
           1829 CALL_FUNCTION            1
           1832 LOAD_GLOBAL              0 (chr)
           1835 LOAD_CONST              23 (115)
           1838 CALL_FUNCTION            1
           1841 LOAD_GLOBAL              0 (chr)
           1844 LOAD_CONST               2 (97)
           1847 CALL_FUNCTION            1
           1850 ROT_TWO             
           1851 BINARY_ADD          
           1852 ROT_TWO             
           1853 BINARY_ADD          
           1854 ROT_TWO             
           1855 BINARY_ADD          
           1856 LOAD_GLOBAL              0 (chr)
           1859 LOAD_CONST               4 (32)
           1862 CALL_FUNCTION            1
           1865 LOAD_GLOBAL              0 (chr)
           1868 LOAD_CONST               7 (121)
           1871 CALL_FUNCTION            1
           1874 LOAD_GLOBAL              0 (chr)
           1877 LOAD_CONST              13 (114)
           1880 CALL_FUNCTION            1
           1883 LOAD_GLOBAL              0 (chr)
           1886 LOAD_CONST              10 (116)
           1889 CALL_FUNCTION            1
           1892 ROT_TWO             
           1893 BINARY_ADD          
           1894 ROT_TWO             
           1895 BINARY_ADD          
           1896 ROT_TWO             
           1897 BINARY_ADD          
           1898 BINARY_ADD          
           1899 BINARY_ADD          
           1900 BINARY_ADD          
           1901 LOAD_GLOBAL              0 (chr)
           1904 LOAD_CONST               2 (97)
           1907 CALL_FUNCTION            1
           1910 LOAD_GLOBAL              0 (chr)
           1913 LOAD_CONST              30 (103)
           1916 CALL_FUNCTION            1
           1919 LOAD_GLOBAL              0 (chr)
           1922 LOAD_CONST               2 (97)
           1925 CALL_FUNCTION            1
           1928 ROT_TWO             
           1929 BINARY_ADD          
           1930 ROT_TWO             
           1931 BINARY_ADD          
           1932 LOAD_GLOBAL              0 (chr)
           1935 LOAD_CONST               4 (32)
           1938 CALL_FUNCTION            1
           1941 LOAD_GLOBAL              0 (chr)
           1944 LOAD_CONST              31 (46)
           1947 CALL_FUNCTION            1
           1950 LOAD_GLOBAL              0 (chr)
           1953 LOAD_CONST              11 (110)
           1956 CALL_FUNCTION            1
           1959 LOAD_GLOBAL              0 (chr)
           1962 LOAD_CONST              14 (105)
           1965 CALL_FUNCTION            1
           1968 ROT_TWO             
           1969 BINARY_ADD          
           1970 ROT_TWO             
           1971 BINARY_ADD          
           1972 ROT_TWO             
           1973 BINARY_ADD          
           1974 BINARY_ADD          
           1975 LOAD_GLOBAL              0 (chr)
           1978 LOAD_CONST               4 (32)
           1981 CALL_FUNCTION            1
           1984 LOAD_GLOBAL              0 (chr)
           1987 LOAD_CONST              12 (111)
           1990 CALL_FUNCTION            1
           1993 LOAD_GLOBAL              0 (chr)
           1996 LOAD_CONST              32 (68)
           1999 CALL_FUNCTION            1
           2002 ROT_TWO             
           2003 BINARY_ADD          
           2004 ROT_TWO             
           2005 BINARY_ADD          
           2006 LOAD_GLOBAL              0 (chr)
           2009 LOAD_CONST               4 (32)
           2012 CALL_FUNCTION            1
           2015 LOAD_GLOBAL              0 (chr)
           2018 LOAD_CONST              10 (116)
           2021 CALL_FUNCTION            1
           2024 LOAD_GLOBAL              0 (chr)
           2027 LOAD_CONST              12 (111)
           2030 CALL_FUNCTION            1
           2033 LOAD_GLOBAL              0 (chr)
           2036 LOAD_CONST              11 (110)
           2039 CALL_FUNCTION            1
           2042 ROT_TWO             
           2043 BINARY_ADD          
           2044 ROT_TWO             
           2045 BINARY_ADD          
           2046 ROT_TWO             
           2047 BINARY_ADD          
           2048 BINARY_ADD          
           2049 BINARY_ADD          
           2050 LOAD_GLOBAL              0 (chr)
           2053 LOAD_CONST              16 (117)
           2056 CALL_FUNCTION            1
           2059 LOAD_GLOBAL              0 (chr)
           2062 LOAD_CONST              13 (114)
           2065 CALL_FUNCTION            1
           2068 LOAD_GLOBAL              0 (chr)
           2071 LOAD_CONST              21 (98)
           2074 CALL_FUNCTION            1
           2077 ROT_TWO             
           2078 BINARY_ADD          
           2079 ROT_TWO             
           2080 BINARY_ADD          
           2081 LOAD_GLOBAL              0 (chr)
           2084 LOAD_CONST              33 (102)
           2087 CALL_FUNCTION            1
           2090 LOAD_GLOBAL              0 (chr)
           2093 LOAD_CONST               4 (32)
           2096 CALL_FUNCTION            1
           2099 LOAD_GLOBAL              0 (chr)
           2102 LOAD_CONST               5 (101)
           2105 CALL_FUNCTION            1
           2108 LOAD_GLOBAL              0 (chr)
           2111 LOAD_CONST              10 (116)
           2114 CALL_FUNCTION            1
           2117 ROT_TWO             
           2118 BINARY_ADD          
           2119 ROT_TWO             
           2120 BINARY_ADD          
           2121 ROT_TWO             
           2122 BINARY_ADD          
           2123 BINARY_ADD          
           2124 LOAD_GLOBAL              0 (chr)
           2127 LOAD_CONST               5 (101)
           2130 CALL_FUNCTION            1
           2133 LOAD_GLOBAL              0 (chr)
           2136 LOAD_CONST              17 (99)
           2139 CALL_FUNCTION            1
           2142 LOAD_GLOBAL              0 (chr)
           2145 LOAD_CONST              13 (114)
           2148 CALL_FUNCTION            1
           2151 LOAD_GLOBAL              0 (chr)
           2154 LOAD_CONST              12 (111)
           2157 CALL_FUNCTION            1
           2160 ROT_TWO             
           2161 BINARY_ADD          
           2162 ROT_TWO             
           2163 BINARY_ADD          
           2164 ROT_TWO             
           2165 BINARY_ADD          
           2166 LOAD_GLOBAL              0 (chr)
           2169 LOAD_CONST              34 (41)
           2172 CALL_FUNCTION            1
           2175 LOAD_GLOBAL              0 (chr)
           2178 LOAD_CONST              35 (61)
           2181 CALL_FUNCTION            1
           2184 LOAD_GLOBAL              0 (chr)
           2187 LOAD_CONST               4 (32)
           2190 CALL_FUNCTION            1
           2193 LOAD_GLOBAL              0 (chr)
           2196 LOAD_CONST              31 (46)
           2199 CALL_FUNCTION            1
           2202 ROT_TWO             
           2203 BINARY_ADD          
           2204 ROT_TWO             
           2205 BINARY_ADD          
           2206 ROT_TWO             
           2207 BINARY_ADD          
           2208 BINARY_ADD          
           2209 BINARY_ADD          
           2210 BINARY_ADD          
           2211 BINARY_ADD          
        >> 2212 PRINT_ITEM          
           2213 PRINT_NEWLINE       
           2214 LOAD_CONST               0 (None)
           2217 RETURN_VALUE        
```

把 `password` 之前的那些数字都弄出来，拼一下，发现是 `llaC em yP aht notriv lauhcamni !eac Ini npreterP tohty ntybdocese!!!`，很像是一句话，结合汇编中的 `ROT_TWO` 和 `BINARY_ADD`，做一下相应的操作：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import marshal, zlib, base64, dis, re

mar = (marshal.loads(zlib.decompress(base64.b64decode('eJyNVktv00AQXm/eL0igiaFA01IO4cIVCUGFBBJwqRAckLhEIQmtRfPwI0QIeio/hRO/hJ/CiStH2M/prj07diGRP43Hs9+MZ2fWMxbnP6mux+oK9xVMHPFViLdCTB0xkeKDFEFfTIU4E8KZq8dCvB4UlN3hGEsdddXU9QTLv1eFiGKGM4cKUgsFCNLFH7dFrS9poayFYmIZm1b0gyqxMOwJaU3r6xs9sW1ooakXuRv+un7Q0sIlLVzOCZq/XtsK2oTSYaZlStogXi1HV0iazoN2CV2HZeXqRQ54TlJRb7FUlKyUatISsdzo+P7UU1Gb1POdMruckepGwk9tIXQTftz2yBaT5JQovWvpSa6poJPuqgao+b9l5Aj/R+mLQIP4f6Q8Vb3g/5TB/TJxWGdZr9EQrmn99fwKtTvAZGU7wzS7GNpZpDm2JgCrr8wrmPoo54UqGampFIeS9ojXjc4E2yI06bq/4DRoUAc0nVnng4k6p7Ks0+j/S8z9V+NZ5dhmrJUM/y7JTJeRtnJ2TSYJvsFq3CQt/vnfqmQXt5KlpuRcIvDAmhnn2E0t9BJ3SvB/SfLWhuOWNiNVZ+h28g4wlwUp00w95si43rZ3r6+fUIEdgOZbQAsyFRRvBR6dla8KCzRdslar7WS+a5HFb39peIAmG7uZTHVm17Czxju4m6bayz8e7J40DzqM0jr0bmv9PmPvk6y5z57HU8wdTDHeiUJvBMAM4+0CpoAZ4BPgJeAYEAHmgAUgAHiAj4AVAGORtwd4AVgC3gEmgBBwCPgMWANOAQ8AbwBHgHuAp4D3gLuARwoGmNUizF/j4yDC5BWM1kNvvlxFA8xikRrBxHIUhutFMBlgQoshhPphGAXe/OggKqqb2cibxwuEXjUcQjccxi5eFRL1fDSbKrUhy2CMb2aLyepkegDWsBwPlrVC0/kLHmeCBQ=='))))
# exec(mar)
# print mar.co_consts
with open('1.pyc', 'wb') as f:
    marshal.dump(mar.co_consts[1], f)

with open('ass.txt', 'r') as f:
    lines = []
    for i in range(324):
        lines.append(f.readline())
    # print lines


def rot(_list):
    a = _list.pop()
    b = _list.pop()
    _list.append(a)
    _list.append(b)
    return _list


def add(_list):
    a = _list.pop()
    b = _list.pop()
    _list.append(b + a)
    return _list


cipher = 'llaC em yP aht notriv lauhcamni !eac Ini npreterP tohty ntybdocese!!!'
cipher = list(cipher)
stack = []

j = 0
for i in lines:
    if 'LOAD_CONST' in i and j < len(cipher):
        stack.append(cipher[j])
        j += 1
    elif 'ROT_TWO' in i:
        stack = rot(stack)
    elif 'BINARY_ADD' in i:
        stack = add(stack)
print stack
```

拿到密码 `Call me a Python virtual machine! I can interpret Python bytecodes!!!`。再跑一下源程序即可。