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


# 3.`PluginManager.hook.method()`����ôʵ�ֵģ�
**Demo�е�pm.hook��ʲô��ʵ�ֵ���pluggy���߼���ʲô��**
#### ������漰������һ����`_HookCaller`�ˣ�`pm.hook.calculate`��ʵ���൱�ڻ�ȡ�˶�Ӧ`_HookCaller`�����õ�������`__call__`�����������´���
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