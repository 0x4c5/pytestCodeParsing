# pluggy����ṹ

#### ����ǰ��demo�еĴ���˳���ڷ���pluggy�ĺ����߼�֮ǰ�����������˽�`HookspecMarker`��`HookspecMarker`���ô���ʲô��

<br/>

### 1.`HookspecMarker`��ʵ���߼���ʲô?
#### ���������������Ĵ���ע��

```python
class HookspecMarker(object):
      """ Decorator helper class for marking functions as hook specifications.

      You can instantiate it with a project_name to get a decorator.
      Calling PluginManager.add_hookspecs later will discover all marked functions
      if the PluginManager uses the same project_name.
      """

      def __init__(self, project_name):
          self.project_name = project_name
```
* **���ǿ��Դ���`project_name`ʵ����`HookspecMarker`�Ի��װ�����������ǵ���`PluginManager.add_hookspec`����Ѱ�������뵱ǰ`PluginManager`ͬ`project_name`�ı�Ǻ�������Ҳ��ǰ��Ҫ��������Ŀproject nameһ�µ�ԭ��֮һ��**
```python
def __call__(
        self, function=None, firstresult=False, historic=False, warn_on_impl=None
    ):
        """ if passed a function, directly sets attributes on the function
        which will make it discoverable to add_hookspecs().  If passed no
        function, returns a decorator which can be applied to a function
        later using the attributes supplied.

        If firstresult is True the 1:N hook call (N being the number of registered
        hook implementation functions) will stop at I<=N when the I'th function
        returns a non-None result.

        If historic is True calls to a hook will be memorized and replayed
        on later registered plugins.

        """

        def setattr_hookspec_opts(func):
            if historic and firstresult:
                raise ValueError("cannot have a historic firstresult hook")
            setattr(
                func,
                self.project_name + "_spec",
                dict(
                    firstresult=firstresult,
                    historic=historic,
                    warn_on_impl=warn_on_impl,
                ),
            )
            return func

        if function is not None:
            return setattr_hookspec_opts(function)
        else:
            return setattr_hookspec_opts
```
**ͨ������`__call__`���߼�������Է��֣���Ҫ�����ǵ�����һ��`setattr(object, name, value)`������װ�εĺ�������һ������`project_nam + _spec`�����Ҹ����Ե�valueΪװ��������ȡֵ��**

<br/>

### 2.`HookspecMarker`��ʵ���߼���ʲô?
#### `HookimplMarker`��ʵ���߼����ƣ��������ڱ�װ�εĺ�������������Ϊ`project_name + _impl`������ֻ��ʾ�˲��ִ���
```python
        def setattr_hookimpl_opts(func):
            setattr(
                func,
                self.project_name + "_impl",
                dict(
                    hookwrapper=hookwrapper,
                    optionalhook=optionalhook,
                    tryfirst=tryfirst,
                    trylast=trylast,
                ),
            )
            return func
            
        if function is None:
            return setattr_hookimpl_opts
        else:
            return setattr_hookimpl_opts(function)
```

<br/><br/>

## pluggy�������
#### plugy�ĺ����߼����Ǽ��д���
```python
pm = PluginManager("myPluggyDemo")
pm.add_hookspecs(HookSpec)
pm.register(HookImpl1())
pm.hook.calculate(a=2, b=3)
```
* **����һ��`PluginManager`�������ڹ���plugin**
* **����`add_hookspecs`, ����һ���µ�hook module object����׼����**
* **����`register`��ע��һ���µ�plugin object**
* **ͨ��`pm.hook`ʵ�ֶ���`calculate`ͬ��������`plugin`�ĵ���**
#### ��������Ĵ����߼����ߣ��������������д����ʵ�֣��԰������Ǹ��õ����
1. #### `pm.add_hookspecs(HookSpec)`����ôʵ�ֵģ�
2. #### `pm.register(HookImpl1())`����ôʵ�ֵģ�
3. #### `pm.hook.calculate(a=2, b=3)`����ôʵ�ֵģ�
```python
def add_hookspecs(self, module_or_class):
    """ add new hook specifications defined in the given module_or_class.
    Functions are recognized if they have been decorated accordingly. """
    names = []
    for name in dir(module_or_class):          #1.�������������������Է����б�
        spec_opts = self.parse_hookspec_opts(module_or_class, name)        #2.�õ�����ǰ����HookspecMarkerΪ�����������Ǹ�����project_name + _spec
```

  - **�������������������Է����б�**
  - **�õ�ÿ�����Է�����������������`project_name + _spec`���򷵻��������򷵻�`None`�������Ǹ÷����Ĵ���չʾ**
 ```python
def parse_hookspec_opts(self, module_or_class, name):        #2.1�õ������Եķ���ʵ��
    method = getattr(module_or_class, name)        #�˴���ȡ������֮ǰ�����hook����
    return getattr(method, self.project_name + "_spec", None)        #�˴���ȡ��Ϊ�÷�������������project_name + _spec
 ```
 
 <br/>
 
2. #### pm.register`��������ע��һ��pluggy��ʵ�ֲ��������Ӧ��hook��������������������Ҫ����
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
#### pm.hook��ʲô��ʵ�ֵ���pluggy���߼���ʲô��
##### ������漰������һ����`_HookCaller`�ˣ�`pm.hook.calculate`��ʵ���൱�ڻ�ȡ�˶�Ӧ`_HookCaller`�����õ�������`__call__`�����������´���
```python
    def __call__(self, *args, **kwargs):
        if args:      #ֻ�ܴ����ֵ����ʽ�Ĳ���
            raise TypeError("hook calling supports only keyword arguments")
        assert not self.is_historic()
        if self.spec and self.spec.argnames:
            notincall = (
                set(self.spec.argnames) - set(["__multicall__"]) - set(kwargs.keys())
            )
            if notincall:
                warnings.warn(
                    "Argument(s) {} which are declared in the hookspec "
                    "can not be found in this hook call".format(tuple(notincall)),
                    stacklevel=2,
                )
        return self._hookexec(self, self.get_hookimpls(), kwargs)
```
#### ���Ĵ��������һ�У�������������self._hhokexec��ʲô�����������ڹ���_HookCallerʱ�����һ�����������ҵ����Ķ���
```python
    def _hookexec(self, hook, methods, kwargs):
        # called from all hookcaller instances.
        # enable_tracing will set its own wrapping function at self._inner_hookexec
        return self._inner_hookexec(hook, methods, kwargs)
```
#### ˳���ߵ���󣬷��ֺ�����ʵ��hook.multicall

```python
self._inner_hookexec = lambda hook, methods, kwargs: hook.multicall(
            methods,
            kwargs,
            firstresult=hook.spec.opts.get("firstresult") if hook.spec else False,
        )
```
#### ����һ��PluggyManager����ʱ�ķ�װ����`_multicall`������ʵ�����£���ϸ�߼����������ٽ���
```python
def _multicall(hook_impls, caller_kwargs, firstresult=False):
    """Execute a call into multiple python functions/methods and return the
    result(s).

    ``caller_kwargs`` comes from _HookCaller.__call__().
    """
    __tracebackhide__ = True
    results = []
    excinfo = None
    try:  # run impl and wrapper setup functions in a loop
        teardowns = []
        try:
            for hook_impl in reversed(hook_impls):
                try:
                    args = [caller_kwargs[argname] for argname in hook_impl.argnames]
                except KeyError:
                    for argname in hook_impl.argnames:
                        if argname not in caller_kwargs:
                            raise HookCallError(
                                "hook call must provide argument %r" % (argname,)
                            )

                if hook_impl.hookwrapper:
                    try:
                        gen = hook_impl.function(*args)
                        next(gen)  # first yield
                        teardowns.append(gen)
                    except StopIteration:
                        _raise_wrapfail(gen, "did not yield")
                else:
                    res = hook_impl.function(*args)
                    if res is not None:
                        results.append(res)
                        if firstresult:  # halt further impl calls
                            break
        except BaseException:
            excinfo = sys.exc_info()
    finally:
        if firstresult:  # first result hooks return a single value
            outcome = _Result(results[0] if results else None, excinfo)
        else:
            outcome = _Result(results, excinfo)

        # run all wrapper post-yield blocks
        for gen in reversed(teardowns):
            try:
                gen.send(outcome)
                _raise_wrapfail(gen, "has second yield")
            except StopIteration:
                pass

        return outcome.get_result()
```

