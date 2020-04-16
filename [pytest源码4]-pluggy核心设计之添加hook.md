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

## `PluginManager.add_hookspecs()`����ôʵ�ֵģ�
### `pm.add_hookspecs(HookSpec)`����ôʵ�ֵģ�

</br>

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