---
date: "2014-03-26"
layout: post
title: "@inject依赖注入的过程"
categories: struts2
tags: struts2源码解析
---

1.容器container实现

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
构造时，传入factories和factoryNamesByType。前者是根据key（由class和name组成）查找类的构造器（授人以鱼不如授人以渔），通过构造器来构造实例，然后进行依赖注入。后者是根据class查找所有的实现类名。