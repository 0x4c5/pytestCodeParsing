# ǰ��
###### ������������������ĵ͹��ˣ����鶼��̫˳�ġ�����ú��о���pytest��Դ�롣֮ǰһֱ���������ٳ�û�п�ʼ������¶�����������ɡ�
##### ����׾�����д����λָ����
#### Դ�����������ô���ֻ���ͦ�����ģ��Ҵ����pytest�ĺ��Ŀ��Python Pluggy���������Ƚ�����Pluggy��
#### ���������Ҫ�������߼��ߣ����ᰴ��Դ��ֲ�ȥ���������⡣
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
<br/>

## Pluggy
#### [�ٷ�](https://github.com/pytest-dev/pluggy "������ʹٷ���ַ")����ô����Pluggy�� 
**"This is the core framework used by the `pytest`, `tox`, and `devpi` projects."**

**������`pytest`��`tox`��`devpi`��Ŀʹ�õĺ��Ŀ�ܡ���**
#### ����Ȼ����������Ƕ�pytestԴ��Ҫ������ʼ��ԭ��  
<br/>

## ��һ��Demo
#### Pluggyһ��ʼ����ΪPytestԴ���һ���ִ��ڵģ��ں��ڱ���������ˣ���Ϊһ���ⲿ��������ʹ�á�
### ������һ��Demo����ʶ�����������£�

```python
# -*- coding:utf-8 -*-

from pluggy import PluginManager, HookspecMarker, HookimplMarker

hookspec = HookspecMarker("myPluggyDemo_1") #һ������hook method���࣬ÿ��hook method����Ҫ��@hookspec��װ��
hookimpl = HookimplMarker("myPluggyDemo_1") #һ��plugin��ʵ�֣���Ҫ����ʵ�ֶ�Ӧ��hook��������ͨ��@hookimpl��װ��


class HookSpec:
    @hookspec
    def calculate(self, a, b):
        pass


class HookImpl1:
    @hookimpl
    def calculate(self, a, b):
        return a + b


pm = PluginManager("myPluggyDemo_1") #����PluginManager����
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
results = pm.hook.calculate(a=1, b=5)
print(results)
```
### Output
```bash
[6]
```
### ������
* #### `Pluggy`�ĺ���Ϊ������`PluginManager`��`HookspecMarker`��`HookimplMarker`
    - ##### `PluginManager`����ע��hook�����hook��ʵ��
    - ##### `HookspecMarker`��������һ��hook���󷽷�װ����
    - ##### `HookimplMarker`��������һ��hook����ʵ�ַ���װ����
* #### ������Ŀ����Ҫ��֤һ��ȫ��Ψһ��Project Name������ԭ��ŵ������ٽ�������DemoΪ`myPluggyDemo_1`
* #### `Hookspec`��һ������hook method���࣬ÿһ��hook method��Ҫ��`hookspec`װ����װ��
* #### `HookImpl1`��һ��plugin��ʵ�֣���Ҫ����ʵ�ֶ�Ӧ��hook��������ͨ��`hookimpl`װ����װ��ʵ�ַ���
* #### ����ĺ����߼����ȴ���һ������������PluginManager�����ڸö�����ע��hook����HookSpec����֮��Ӧ��plugin����HookImpl1��Ȼ��ͨ��PluginManager�Դ���hook���������ö�Ӧ��hook������������ز�����
  ##### ע�⣺����hook����ʱ�������Թؼ��ֵ���ʽ���ݡ�
  
