# ǰ��
###### ���˽���pluggy֮�����ǻ���Ҫ���˽�Щ֪ʶ��Ϊ��������߼���׼��
##### ����׾�����д����λָ����
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
</br>
</br>
</br>


# �ܽ�
#### ��ǰ�������Դ�����һ���ܽᣬ����Ͳ�����ϸ��ȥ���ˡ�
#### �Ķ���һ��Դ��֮�������ٶ���pluggy��demo�����ܽ�

<br/><br/>

### pluggy Demo

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

<br/><br/>

1.���ȱ�֤��ĿΨһ��project name����Ϊ�������hook��ע��pluggyʱ����Ҫ���÷�װ�࣬װ������hook������pluginʵ�ֵ�������project name +  spec+ method Ϊ���������ӵ���װ���С���֤Ψһ������ȷ���ҵ���Ӧ��hook��plugin��

2.ʵ����HookspecMarker, HookimplMarkerʱ�᷵��һ��װ���������ǿ���ͨ�������ʽ�Զ��壨���¶��壩װ������������

3.��ͨ��װ��������hook������plugin����ʱ�����벻ͬ�Ĳ������Դﵽ�磺�ı�ִ��˳�򡢶���wrapper plugin��yield����Ч��

4.ͨ��`add_hookspecs`�������`hook instance`�е����ⷽ����`hook name`���Ӧ��hook���õķ�װʵ��`_HookCaller`��`name:hc`����ʽ��Ϊ������Ӽӵ�`PluginManager`ʵ���е�`self.hook`�����ϣ���������ⷽ��`hook method`���ڵĻ���

- _�������ⷽ��`hook method`�����ڵĻ�����`module_or_class`��á�`hook name`��ͨ��`HookSpec()`ִ��һ��2.���߼�����������ǰ`PluginManager`ʵ���е�`_nonwrappers`��`_wrappers`���hook�Ƿ���ȷ��ӡ�_


5.ͨ��`register`�������`pluggy instance`�����ⷽ��`pluggy method`�������ݷ�װ�õ�`HookImpl instance`������ȡ����ǰ`PluginManager`ʵ���е�`self.hook`��ͬ�����ԣ��ؼ�������⣬��ǰ�����ڵ�4�����hookʱ������ӵ�ͬ��hook�������������ʱ�������ͬ�����Թ��������ˡ�Ȼ��`plugin`��ʵ�ַ����ĵ��÷�����һ��˳��ͨ��װ��������ĵĲ�������ӵ�ͬ�������У������浽�б�hookcallers�У������洢��`PluginManager`ʵ����`self._plugin2hookcallers[plugin]`

- ��������û�ж�Ӧ��ͬ��hook�أ���ʱ�ͻ�ͨ��ִ�����4������ͬ�Ĳ�������װ`_HookCaller`��`name:hook`����ʽ��Ϊ������ӵ�`PluginManager`ʵ���е�`self.hook`�����ϡ�Ȼ���ٽ�`plugin`�ĵ��÷�����ӵ�ͬ�������У������浽`PluginManager`ʵ����`self._plugin2hookcallers[plugin]`��
- ǰ�ã���Ԥ�ȼ�鵱ǰpluggy�Ƿ��Ѿ�ע����ˣ��׳�ValueError

6.ÿ��pluggyʵ�ֶ��ᱻ��װ��`_HookCaller instance`������`PluginManager`ʵ������pm.hook�У�ÿ�ζ�ͬ��pluggy�������е���ʱ�����µĵ����߼��������ģ���ô洢�Ŷ�Ӧ��pluginʵ�ֵ��б����䷴ת�����Ժ�register��plugin��ִ�У��������plugin�����Ĳ����Ƿ������ض�Ӧhook������Լ�����ֳ�hookwrapper��nonwrapper������ʽȥִ��`plugin`��ʵ�ִ��룬ִ�н����_Result����з�װ�������plugin call���������쳣��û���쳣�ͷ���ִ�н����

