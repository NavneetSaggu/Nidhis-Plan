# Abstract Factory

1. If you need a way to make different kinds of objects, one single class, you’ll make a factory. Suppose you make widgets
    1. Button Widget
    2. Image Widget
    3. Title Widget
2. You’ll make a factory that makes different kinds of widgets, using some switch on enum
3. But suppose now you need a way to make two kinds of each widget. For different operating systems
    1. Button Widget
        1. For iOS
        2. For Android
    2. Image Widget
        1. For iOS
        2. For Android
    3. Title Widget
        1. For iOS
        2. For Android
4. The way to implement this would be as follow

```java
interface IWidgetFactory {
	Widget makeWidget();
}

class AndroidWidgetFactory implements IWidgetFactory {
	public Widget makeWidget(WidgetType widgetType) {
		// make widget
	}
}

class IosWidgetFactory implements IWidgetFactory {
	public Widget makeWidget(WidgetType widgetType) {
		// make widget
	}
}

// now whoever makes this, can use the abstract factory without caring about 
// what OS widgets are for

class PageService {
	Map<OperatingSystem, IWidgetFactory> widgetFactories;
	
	List<Widget> getWidgets(RequestContext requestContext) {
		IWidgetFactory widgetFactory = widgetFactories.get(requestContext.getOS());
		List<Widget> widgets = new ArrayList<>(); 
		for(WidgetType widgetType: requestContext.getWidgetTypesRequested()) {
			widgets.add(widgetFactory.makeWidget(widgetType));
		}
		return widgets;
	}
}
```

# Where is it used?

- You want to design a system to create cars with specific configurations for different regions, such as North America and Europe. Each region may have unique requirements and regulations, and you want to ensure that cars produced for each region meet those standards.
-