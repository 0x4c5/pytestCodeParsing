# ǰ��
###### ���˽���pluggy֮�����ǻ���Ҫ���˽�Щ֪ʶ��Ϊ��������߼���׼��
##### ����׾�����д����λָ����
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
</br>
</br>
</br>

# hook��plugin�Ĺ�ϵ
#### hook��plugin��`1:N`�Ķ�Ӧ��ϵ������ͬʱע���˶��ʵ����ͬһhook��plugin������Ӧ�ķ��ض�������
### Demo����
```
# -*- coding:utf-8 -*-

from pluggy import PluginManager, HookspecMarker, HookimplMarker

hookspec = HookspecMarker("myPluggyDemo_2")
hookimpl = HookimplMarker("myPluggyDemo_2")


class HookSpec:
    @hookspec
    def calculate(self, a, b):
        pass


class HookImpl1:
    @hookimpl
    def calculate(self, a, b):
        return a + b


class HookImpl2:
    @hookimpl
    def calculate(self, a, b):
        return a * b
    

pm = PluginManager("myPluggyDemo_2")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.register(HookImpl2())
print(pm.hook.calculate(a=2, b=3))
```
### Output
```
[6, 5]
```
### ������
* #### ��Demo2�У�����ע��������`plugin`��`HookImpl1`��`HookImpl2`���ֱ�ʵ���˼ӷ��ͳ˷������߼���
* #### ÿ�ε���hook���᷵������`plugin`ִ�еĽ������ִ�к�ע���`HookImpl2`����ִ����ע���`HookImpl1`,��Խ��ע���pluginԽ��ִ�С�(�����ὲԭ��)

<br/>
<br/>
</br>

# Plugin�ĵ���˳�������
***
<br/>
</br>

## HookspecMarkerװ��������
#### HookspckMarkerװ����֧�ִ���һЩ�ض��Ĳ��������õ���
* ##### firstresult - ���firstresultֵΪTrueʱ����ȡ��һ��pluginִ�н�����ֹͣ���жϣ�����ִ�С�
* ##### historic - ���ֵΪTrueʱ����ʾ���hook����Ҫ������ü�¼��call history���ģ������õ��ü�¼�ط���δ����ע���plugins�ϡ�
#### ��װ����������firstresult=Trueʱ��plugin��ִ�л��ں�ע���HookImpl2ִ����Ϻ�ֹͣ����������ִ�С�
### Demo����
```
# -*- coding:utf-8 -*-

from pluggy import PluginManager, HookspecMarker, HookimplMarker

hookspec = HookspecMarker("myPluggyDemo_3")
hookimpl = HookimplMarker("myPluggyDemo_3")


class HookSpec:
    @hookspec(firstresult=True)
    def calculate(self, a, b): 
        pass


class HookImpl1:
    @hookimpl
    def calculate(self, a, b):
        return a + b            #�������в���ִ�мӷ��߼�


class HookImpl2:
    @hookimpl
    def calculate(self, a, b):
        return a * b


pm = PluginManager("myPluggyDemo_3")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.register(HookImpl2())
print(pm.hook.calculate(a=2, b=3))
```
### Output
```
6
```

<br/>
</br>

## HookimplMarkerװ��������
#### HookimplMarkerװ����֧�ִ���һЩ�ض��Ĳ��������õ���
* #### tryfirst - ���tryfirstֵΪTrue�����plugin�ᾡ���������1:N��ʵ����·ִ��
* #### trylast - ���trylastֵΪTrue�����plugin����Ӧ�ؾ����������1:N��ʵ������ִ��
* #### hookwrapper - ����ò���ΪTrue����Ҫ��plugin��ʵ��һ��yield��pluginִ��ʱ��ִ��wrapper pluginǰ�沿�ֵ��߼���Ȼ��תȥִ������plugin������ٻ���ִ��wrapper plugin���沿�ֵ��߼���
* #### optionalhook - ����ò���ΪTrue���ڴ�pluginȱ����ƥ���hookʱ�����ᱨerror��spec is found����

</br>
</br>

## tryfirst��Demo
##### �����޸�һ��demo3����HookImpl1����tryfirst=True���������ɴﵽ��ִ����ע���HookImpl1��
```
# -*- coding:utf-8 -*-

from pluggy import PluginManager, HookspecMarker, HookimplMarker

hookspec = HookspecMarker("myPluggyDemo_4")
hookimpl = HookimplMarker("myPluggyDemo_4")


class HookSpec:
    @hookspec()
    def calculate(self, a, b):
        pass


class HookImpl1:
    @hookimpl(tryfirst=True)
    def calculate(self, a, b):
        return a + b


class HookImpl2:
    @hookimpl
    def calculate(self, a, b):
        return a * b


pm = PluginManager("myPluggyDemo_4")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.register(HookImpl2())
print(pm.hook.calculate(a=2, b=3))
```
### Output

``` 
[5, 6]
```
#### trylast�Դ����ƣ�Я���߱�Ϊ��ִ��

</br>

## hookwrapper
#### ����ʵ��һ�������plugin`WrapperPlugin`
```
# -*- coding:utf-8 -*-

from pluggy import PluginManager, HookspecMarker, HookimplMarker

hookspec = HookspecMarker("myPluggyDemo_5")
hookimpl = HookimplMarker("myPluggyDemo_5")


class HookSpec:
    @hookspec
    def calculate(self, a, b):
        pass


class HookImpl1:
    @hookimpl
    def calculate(self, a, b):
        print('HookImpl1 execute!')
        return a + b


class HookImpl2:
    @hookimpl(tryfirst=True)
    def calculate(self, a, b):
        print('HookImpl2 execute!')
        return a * b


class WrapperPluggy:
    @hookimpl(hookwrapper=True)
    def calculate(self, a, b):
        print('WrapperPluggy execute!')
        print("Before yield")
        result = yield            #�˴����ص�ֵΪ��������pluggyִ�еĽ��
        print(f"After yield,result is {result.get_result()}")
        return a * b + (a + b)


pm = PluginManager("myPluggyDemo_5")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.register(HookImpl2())
pm.register(WrapperPluggy())
print(pm.hook.calculate(a=2, b=3))
```
### Output
```
WrapperPluggy execute!
Before yield
HookImpl2 execute!
HookImpl1 execute!
After yield,result is [6, 5]
[6, 5]
```
### ������
* ##### `ImplWrapper`�е�`pluggy`�Ĵ����߼�����`result = yield` Ϊ�ָ��ߣ��ֳ��������֡���һ����ִ����Ϻ��жϼ���ִ�У�תȥִ������`plugin`��������`plugin`��ִ����ʱ����������ִ��ʣ�µĲ��֡�
* ##### `result = yield`resultͨ��yield����ȡ������pluginִ�еĽ��������`wrapper plugin`��ִ�н����`HookImpl2`��`HookImpl1`��
* ##### ��Output�п��Կ���������`WrapperPluggy`�ķ��ؽ��û�б���ӡ������������Ϊ`wrapper plugin`�ķ���ֵ�ᱻ`Ignore`��ԭ��������ᵽ��