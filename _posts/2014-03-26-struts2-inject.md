---
date: "2014-03-26"
layout: post
title: "@inject依赖注入的过程"
categories: struts2
tags: struts2源码解析
---

首先需要知道实例是如何创建以及如何被注入的，而这一切都由container这个容器进行管理。

**1.实例构建**

	class ContainerImpl implements Container {
		final Map<Key<?>, InternalFactory<?>> factories;
		final Map<Class<?>, Set<String>> factoryNamesByType;
	
		ContainerImpl( Map<Key<?>, InternalFactory<?>> factories ) {
			this.factories = factories;
			Map<Class<?>, Set<String>> map = new HashMap<Class<?>, Set<String>>();
			for ( Key<?> key : factories.keySet() ) {
				Set<String> names = map.get(key.getType());
				if (names == null) {
					names = new HashSet<String>();
					map.put(key.getType(), names);
				}
				names.add(key.getName());
			}
	
			for ( Entry<Class<?>, Set<String>> entry : map.entrySet() ) {
				entry.setValue(Collections.unmodifiableSet(entry.getValue()));
			}
	
			this.factoryNamesByType = Collections.unmodifiableMap(map);
		}
		...
	}

构造时，传入factories和factoryNamesByType。前者是根据key（由class和name组成）查找类的实例构造方法（授人以鱼不如授人以渔），以此来构造实例，然后进行依赖注入。后者是根据class查找所有的实现类名。

**2.injector注入器**

	class ContainerImpl implements Container {
		final Map<Class<?>, List<Injector>> injectors =
				new ReferenceCache<Class<?>, List<Injector>>() {
					@Override
					protected List<Injector> create( Class<?> key ) {
						List<Injector> injectors = new ArrayList<Injector>();
						addInjectors(key, injectors);
						return injectors;
					}
				};
		...
	}
注入器包括属性注入器和方法注入器等。（每个@injector注解都会被解析为一个注入器类，FieldInjector、MethodInjector）。ReferenceCache继承Map并对其做了扩展，当get(key)时，如果不存在，则会调用create方法创建相应的injectors。injector内部有个inject方法，会调用method.invoke从而调用加了@inject注解的方法(针对方法@inject，属性@inject类似)