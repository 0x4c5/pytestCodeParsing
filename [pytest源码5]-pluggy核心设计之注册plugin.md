# ǰ��
###### ���˽���pluggy֮�����ǻ���Ҫ���˽�Щ֪ʶ��Ϊ��������߼���׼��
##### ����׾�����д����λָ����
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
</br>
</br>
</br>

# pluggy�����߼�
```python
pm = PluginManager("myPluggyDemo")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.hook.calculate(a=2, b=3)
``` 

<br/>
<br/>


# `PluginManager.register()`����ôʵ�ֵģ�
**Demo�е�`pm.register(HookImpl1())`����ôʵ�ֵģ�**

</br>

### pm.register`��������ע��һ��pluggy��ʵ�ֲ��������Ӧ��hook��������������������Ҫ����
```python
# register matching hook implementations of the plugin
self._plugin2hookcallers[plugin] = hookcallers = []
for name in dir(plugin):
    hookimpl_opts = self.parse_hookimpl_opts(plugin, name)    #��ȡpluggy�����Ի򷽷��е�����attribute project_name + _impl
    if hookimpl_opts is not None:
        normalize_hookimpl_opts(hookimpl_opts)
        method = getattr(plugin, name)    #����attribute����ʱ��ȡ��plugin�Ķ�Ӧ����
        hookimpl = HookImpl(plugin, plugin_name, method, hookimpl_opts)
        hook = getattr(self.hook, name, None)
        if hook is None:
            hook = _HookCaller(name, self._hookexec)    #Ϊhook���һ��_HookCaller����
            setattr(self.hook, name, hook)
        elif hook.has_spec():
            self._verify_hook(hook, hookimpl)
            hook._maybe_apply_history(hookimpl)
        hook._add_hookimpl(hookimpl)    #��hookimpl��ӵ�hook��
        hookcallers.append(hook)    #�������ҵ���ÿһ��plugin hook��ӵ�hookcallers,�Դ�����
```
  - **����pluggy������������Ի򷽷���method��������ȡ��pluggy method������`attribute` `project_name + _impl`**
  - **������project_name + _impl��method��װ��һ��HookImpl��**
  - **�ٰ�һ��`_HookCaller`�Ķ�����ӵ�`hook`�У���Ϊ`self.hook`����һ��valueΪ`hook`��nameΪ`method`�����ԣ�����ǰ���demo��`calculate`��**
  - **��󽫱����ҵ���ÿһ��`_HookCaller`��ӵ�hookcallers,�Դ�����**
  