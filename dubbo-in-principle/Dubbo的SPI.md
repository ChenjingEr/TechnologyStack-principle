## Dubbo的SPI

#### Dubbo的SPI实现
ExtensionLoader：扩展加载，根据接口加载对应的实现
```
    //获取一个扩展加载器
    ExtensionLoader<Protocol> loader = ExtensionLoader.getExtensionLoader(Protocol.class);
    //在扩展加载器中找到扩展器(服务)
    Protocol protocol = loader.getExtension(DubboProtocol.NAME);    
```

#### Dubbo的SPI实现原理

##### 获取一个对应接口的扩展加载器

```
//已经解析的加载器缓存
private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<Class<?>, ExtensionLoader<?>>();


public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {

    if (type == null)
        throw new IllegalArgumentException("Extension type == null");
    //type必须是接口
    if (!type.isInterface()) {
        throw new IllegalArgumentException("Extension type(" + type + ") is not interface!");
    }
    //需要有@SPI注解的接口
    if (!withExtensionAnnotation(type)) {
        throw new IllegalArgumentException("Extension type(" + type +
                ") is not extension, because WITHOUT @" + SPI.class.getSimpleName() + " Annotation!");
    }
    //先在以及解析加载的扩展缓存中查找，
    ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
    if (loader == null) {
        //找不到new一个ExtensionLoader
        EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
        loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
    }
    return loader;
}

private ExtensionLoader(Class<?> type) {
    this.type = type;
    //new 里面如果type==ExtensionFactory.class才会结束，否者递归getExtensionLoader()
    objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
}
```
取得一个**接口**的扩展加载类，这个接口必须被SPI注解。先在缓存中查找时候有这个扩展器，如果没有，新建一个对应的扩展加载放到缓存中(call) <br>
new一个对应type的接口，构造器中递归调用ExtensionLoader.getExtensionLoader(),type=ExtensionFactory。先获取一个扩展器工厂的加载器类，获取适配扩展作为objectFactory。

* 获取适配扩展
```

private final Holder<Object> cachedAdaptiveInstance = new Holder<Object>();

public T getAdaptiveExtension() {
//先缓存中获取，不存在就获取create一个。duble-check
Object instance = cachedAdaptiveInstance.get();
if (instance == null) {
    if (createAdaptiveInstanceError == null) {
        synchronized (cachedAdaptiveInstance) {
            //
            instance = cachedAdaptiveInstance.get();
            if (instance == null) {
                try {
                    //创建适配扩展
                    instance = createAdaptiveExtension();
                    cachedAdaptiveInstance.set(instance);
                } catch (Throwable t) {
                    createAdaptiveInstanceError = t;
                    throw new IllegalStateException("fail to create adaptive instance: " + t.toString(), t);
                }
            }
        }
    } else {
        throw new IllegalStateException("fail to create adaptive instance: " + createAdaptiveInstanceError.toString(), createAdaptiveInstanceError);
    }
}

    return (T) instance;
}

private T createAdaptiveExtension() {
    try {
        //获取一个适配类型，然后通过反射获取一个实例，
        return injectExtension((T) getAdaptiveExtensionClass().newInstance());
    } catch (Exception e) {
        throw new IllegalStateException("Can not create adaptive extension " + type + ", cause: " + e.getMessage(), e);
    }
}

private Class<?> getAdaptiveExtensionClass() {
    //这里会加载解析配置文件中的class，放到Map<String, Class<?>> extensionClasses中。
    //这里会加载META-INFO中name=class的信息并解析,adaptive放入cachedAdaptiveClass，包装类(有一个构造函数的入参是type的类)
    //放入cachedWrapperClasses缓存中
    getExtensionClasses();
    //加载时有adaptive就放入cachedAdaptiveClass缓存中
    if (cachedAdaptiveClass != null) {
        return cachedAdaptiveClass;
    }
    //没有就动态创建一个,通过StringBuilder的appends动态拼成一个Java，然后编译生成一个class
    return cachedAdaptiveClass = createAdaptiveExtensionClass();
}
```
获取适配的扩展工厂，解析META-INFO中的配置name=class_name，adaptive放入Class<?> cachedAdaptiveClass缓存,wapper放入Set<Class<?>> cachedWrapperClasses缓存。动态生成的AdaptiveExtensionClass。具体生成的代码例子：Protocol$Adaptive.java

到这里获取一个扩展加载器结束。通过扩展加载器就可以获取扩展器了。

##### 获取扩展器
```

private final ConcurrentMap<String, Holder<Object>> cachedInstances = new ConcurrentHashMap<String, Holder<Object>>();


//通过name来获取，实例的创建在createExtension(name)中完成
public T getExtension(String name) {
    if (name == null || name.length() == 0)
        throw new IllegalArgumentException("Extension name == null");
    if ("true".equals(name)) {
        return getDefaultExtension();
    }
    //缓存
    Holder<Object> holder = cachedInstances.get(name);
    if (holder == null) {
        cachedInstances.putIfAbsent(name, new Holder<Object>());
        holder = cachedInstances.get(name);
    }
    Object instance = holder.get();
    if (instance == null) {
        synchronized (holder) {
            instance = holder.get();
            if (instance == null) {
                instance = createExtension(name);
                holder.set(instance);
            }
        }
    }
    return (T) instance;
}
```

createExtension细节：
```
//缓存
private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<Class<?>, Object>();

private T createExtension(String name) {
    Class<?> clazz = getExtensionClasses().get(name);
    if (clazz == null) {
        throw findException(name);
    }
    try {
        T instance = (T) EXTENSION_INSTANCES.get(clazz);
        if (instance == null) {
            //instance通过class的newInstance
            EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
            instance = (T) EXTENSION_INSTANCES.get(clazz);
        }
        injectExtension(instance);
        //获取的最终的扩展器是由wapperClass形成的链
        Set<Class<?>> wrapperClasses = cachedWrapperClasses;
        if (wrapperClasses != null && !wrapperClasses.isEmpty()) {
            for (Class<?> wrapperClass : wrapperClasses) {
                instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
            }
        }
        return instance;
    } catch (Throwable t) {
        throw new IllegalStateException("Extension instance(name: " + name + ", class: " +
                type + ")  could not be instantiated: " + t.getMessage(), t);
    }
}
```

得到的最终的扩展器是由wapper形成的链(如果由Wapper的话)。这样就找到需要的接口类型对应的扩展器。