##�ܽ�
####��ǰ�������Դ�����һ���ܽᣬ����Ͳ�����ϸ��ȥ���ˡ�
####�����Դ��֮������������pluggy��demo

1.���ȱ�֤��ĿΨһ��project name����Ϊ�������hook��ע��pluggyʱ����Ҫ���÷�װ�࣬װ������hook������pluginʵ�ֵ�������project name +  spec+ method Ϊ���������ӵ���װ���С���֤Ψһ������ȷ���ҵ���Ӧ��hook��plugin��

2.ʵ����HookspecMarker, HookimplMarkerʱ�᷵��һ��װ���������ǿ���ͨ�������ʽ�Զ��壨���¶��壩װ������������

3.��ͨ��װ��������hook������plugin����ʱ�����벻ͬ�Ĳ������Դﵽ�磺�ı�ִ��˳�򡢶���wrapper plugin��yield����Ч��

4.ͨ��`add_hookspecs`�������`hook instance`�е����ⷽ����`hook name`���Ӧ��hook���õķ�װʵ��`_HookCaller`��`name:hc`����ʽ��Ϊ������Ӽӵ�`PluginManager`ʵ���е�`self.hook`�����ϣ���������ⷽ��`hook method`���ڵĻ���

- _�������ⷽ��`hook method`�����ڵĻ�����`module_or_class`��á�`hook name`��ͨ��`HookSpec()`ִ��һ��2.���߼�����������ǰ`PluginManager`ʵ���е�`_nonwrappers`��`_wrappers`���hook�Ƿ���ȷ��ӡ�_


5.ͨ��`register`�������`pluggy instance`�����ⷽ��`pluggy method`�������ݷ�װ�õ�`HookImpl instance`������ȡ����ǰ`PluginManager`ʵ���е�`self.hook`��ͬ�����ԣ��ؼ�������⣬��ǰ�����ڵ�4�����hookʱ������ӵ�ͬ��hook�������������ʱ�������ͬ�����Թ��������ˡ�Ȼ��`plugin`��ʵ�ַ����ĵ��÷�����һ��˳��ͨ��װ��������ĵĲ�������ӵ�ͬ�������У������浽�б�hookcallers�У������洢��`PluginManager`ʵ����`self._plugin2hookcallers[plugin]`

- ��������û�ж�Ӧ��ͬ��hook�أ���ʱ�ͻ�ͨ��ִ�����4������ͬ�Ĳ�������װ`_HookCaller`��`name:hook`����ʽ��Ϊ������ӵ�`PluginManager`ʵ���е�`self.hook`�����ϡ�Ȼ���ٽ�`plugin`�ĵ��÷�����ӵ�ͬ�������У������浽`PluginManager`ʵ����`self._plugin2hookcallers[plugin]`��
- ǰ�ã���Ԥ�ȼ�鵱ǰpluggy�Ƿ��Ѿ�ע����ˣ��׳�ValueError

6.ÿ��pluggyʵ�ֶ��ᱻ��װ��`_HookCaller instance`������`PluginManager`ʵ������pm.hook�У�ÿ�ζ�ͬ��pluggy�������е���ʱ�����µĵ����߼��������ģ���ô洢�Ŷ�Ӧ��pluginʵ�ֵ��б����䷴ת�����Ժ�register��plugin��ִ�У��������plugin�����Ĳ����Ƿ������ض�Ӧhook������Լ�����ֳ�hookwrapper��nonwrapper������ʽȥִ��`plugin`��ʵ�ִ��룬ִ�н����_Result����з�װ�������plugin call���������쳣��û���쳣�ͷ���ִ�н����

