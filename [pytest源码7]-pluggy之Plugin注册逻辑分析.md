# ǰ��
###### ���˽���pluggy֮�����ǻ���Ҫ���˽�Щ֪ʶ��Ϊ��������߼���׼��
##### ����׾�����д����λָ����
###### _������ҵ����¶����а����������������Ľ���ָ����Star������õ�壬�������㣬��ʤ�м���_
</br>
</br>
</br>

# pluggyע���߼�������

#### ��������ϸ����һ��plugin��ע���߼�register����
```python
  plugin_name = name or self.get_canonical_name(plugin)    #��ȡ�����

  if plugin_name in self._name2plugin or plugin in self._plugin2hookcallers:
      if self._name2plugin.get(plugin_name, -1) is None:
          return  # blocked plugin, return None to indicate no registration
      raise ValueError(
          "Plugin already registered: %s=%s\n%s"
          % (plugin_name, plugin, self._name2plugin)
      )
```
  * **���ݴ���plugin name����plugin�����ȡ������������丳ֵ��plugin_name**
  * **self._name2plugin����plugin_nameΪkey��dict**
  * **self._pluginhookcallers����plugin objectΪkey��dict**
  * **ͨ����������dict���жϴ����plugin�Ƿ��Ѿ�ע�����**
```python
  self._name2plugin[plugin_name] = plugin
```
  * **�����pluggy��`plugin_name:plugin object`����ʽ���浽`self._name2plugin`��**
```python
  self._plugin2hookcallers[plugin] = hookcallers = []
```
  * **����һ��`list`����`hookcallers`��������ÿ��pluggy��ʵ�ʵ��ö���`_HookCaller`����`plugin object:hookcallers object`����ʽ������`self._plugin2hookcallers`**
```python
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
#### ��������������������ᵽ����������HookImpl��_HookCaller
#### ���ȿ���HookImpl��ʵ�֣���ʵ����һ�����ݷ�װ��
```python
  class HookImpl(object):
      def __init__(self, plugin, plugin_name, function, hook_impl_opts):
          self.function = function
          self.argnames, self.kwargnames = varnames(self.function)
          self.plugin = plugin
          self.opts = hook_impl_opts
          self.plugin_name = plugin_name
          self.__dict__.update(hook_impl_opts)

      def __repr__(self):
          return "<HookImpl plugin_name=%r, plugin=%r>" % (self.plugin_name, self.plugin)
```
#### ��󿴿�����_HookCaller��ʵ�֣���������plugin�ĺ�����
```python
  class _HookCaller(object):
      def __init__(self, name, hook_execute, specmodule_or_class=None, spec_opts=None):
          self.name = name
          self._wrappers = []
          self._nonwrappers = []
          self._hookexec = hook_execute
          self.argnames = None
          self.kwargnames = None
          self.multicall = _multicall
          self.spec = None
          if specmodule_or_class is not None:
              assert spec_opts is not None
              self.set_specification(specmodule_or_class, spec_opts)
 ```
  * ##### ��register�߼��У����Ǵ���Ĳ�����name��self._hookexec(hook_execute)������hook_execute��ʾ����һ���������󣬸���ʵ��plugin�ĵ���
```python
    def has_spec(self):
        return self.spec is not None

    def set_specification(self, specmodule_or_class, spec_opts):
        assert not self.has_spec()
        self.spec = HookSpec(specmodule_or_class, self.name, spec_opts)
        if spec_opts.get("historic"):
            self._call_history = []

    def is_historic(self):
        return hasattr(self, "_call_history")
```
  * **���ӵ�����ʷ��¼�����ص�����ʷ��¼**
```python
    def _add_hookimpl(self, hookimpl):
        """Add an implementation to the callback chain.
        """
        if hookimpl.hookwrapper:    #����hookimpl.hookwrapper��plugin��Ϊ����
            methods = self._wrappers
        else:
            methods = self._nonwrappers

        if hookimpl.trylast:    #���ݲ���Ϊplugin������Ӧ��ִ��˳��
            methods.insert(0, hookimpl)
        elif hookimpl.tryfirst:
            methods.append(hookimpl)
        else:
            # find last non-tryfirst method
            i = len(methods) - 1
            while i >= 0 and methods[i].tryfirst:
                i -= 1
            methods.insert(i + 1, hookimpl)

        if "__multicall__" in hookimpl.argnames:    #�汾�������Ĵ�������֧��__multicall__�Ĳ�����ʽ
            warnings.warn(
                "Support for __multicall__ is now deprecated and will be"
                "removed in an upcoming release.",
                DeprecationWarning,
            )
            self.multicall = _legacymulticall 
```
  * **�ڹ��캯�������ǿ��Կ���`self._wrappers`��`self._nonwrappers`��ͨ��`_add_hookimpl`���ǽ�`plugin`�ֳ�����**
  * **����װ��������Ĳ�����plugin��ִ��˳���������**
```python
    def _remove_plugin(self, plugin):
        def remove(wrappers):
            for i, method in enumerate(wrappers):
                if method.plugin == plugin:
                    del wrappers[i]
                    return True

        if remove(self._wrappers) is None:
            if remove(self._nonwrappers) is None:
                raise ValueError("plugin %r not found" % (plugin,))
```
  * **remove pluginʱ�轫`_wrappers`��`_nonwrappers`�����е�`plugin`��`remove`**
  * **ͨ�����������е�`method`��`plugin`�������ҵ�Ҫɾ����`plugin`��ͨ��`del`��ɾ�����ñ���**
```python
    def call_extra(self, methods, kwargs):
        """ Call the hook with some additional temporarily participating
        methods using the specified kwargs as call parameters. """
        old = list(self._nonwrappers), list(self._wrappers)    #��ȡ������ԭʼ���list
        for method in methods:    #����plugin method
            opts = dict(hookwrapper=False, trylast=False, tryfirst=False)    #ͳһ��ʱplugin����
            hookimpl = HookImpl(None, "<temp>", method, opts)    #������ʱpluginʵ��
            self._add_hookimpl(hookimpl)    #Ĭ������pluginִ��˳��
        try:
            return self(**kwargs)    #���ز����������ʱplugin�Ĳ������
        finally:
            self._nonwrappers, self._wrappers = old    #ִ����ϣ��ָ�ԭʼ���list
```
  * **��ʱ���ǻ���Ҫ��ĳһ��ִ������һЩ��ʱ��`plugin`����`Plugin`Ϊ�����ṩһ������`call_extra`**
  * **���Ȼ�ȡԭ����`plugin`�б��ڷ���ִ�е������Ҫ�ָ�ԭ����plugin�б�**
  * **�����Ǵ������ʱ`plugin method`��ͳһ����Ĭ��ִ��˳�����Ϊ"<temp>"����ʱ`plugin object`**
  * **������������ʱ`plugin`��`_HookCaller object`���أ��Դ�����**

```python
    def _maybe_apply_history(self, method):
        """Apply call history to a new hookimpl if it is marked as historic.
        """
        if self.is_historic():
            for kwargs, result_callback in self._call_history:
                res = self._hookexec(self, [method], kwargs)
                if res and result_callback is not None:
                    result_callback(res[0])
```
  * **�ж�hook�Ƿ���ָ������**
  * **����������ʷ���õ����ִ�еĽ��**
