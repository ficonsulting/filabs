---
layout:     post
comments: true
tags:	[FI Consulting, FI Labs, Angular]
type: Blog
title:      "Angular Routing"
subtitle:   "How Lazy Loading can go wrong"
date:       2017-08-24 12:00:00
authors: jon_crandall 
permalink: /:title
---
## Getting Started
Angular 2 is a front-end development framework built on Microsoft’s TypeScript, which is a superset of JavaScript. Angular 2 is a framework that empowers developers to build robust web applications.  

As a front-end language, all of the code runs locally in the user’s browser. With a small application, this is not an issue but as your application grows, the startup process can slow significantly. Angular provides an intelligent, scalable design that only loads code when required. This process is called “Lazy Loading,” which requires a strong understanding of Angular’s routing system to effectively implement.

The routing system relies on a list of route objects, which are made up of a path and a component or module. Lazy loading allows developers to provide a module to be loaded upon navigation to the identified path. Angular’s routing system allows the user to break up the navigation path between the parent and lazy loaded module.  This is an important concept when developing an application with many lazy modules which will be illustrated in the samples in this post.

Sample (app.routing.ts):
{% highlight javascript %}
const appRoutes: Routes = [
	{
		path: '',
		component: HomeComponent
	},
	{
		path: 'eager',
		component: EagerComponent
	},
	{
		path: 'lazy',
		loanChildren: 'app/lazy/lazy.module#LazyModule'
	},
	{
		path: '404',
		component: _404Component
	},
{
		path: '**',
		redirectTo: '404'
	}

];  
{% endhighlight %}

In this sample, both the HomeComponent and EagerComponent are part of the application’s default app module. When the application is first loaded, the code for both the HomeComponent and the EagerCompoent will be loaded by the browser. The LazyModule, which can contain one or many components, will not be loaded until navigating to a ‘lazy’ path. This allows the developer to spread out the load time. The last two routes are our 404 Page Not Found routes which we will discuss later.

Routing is straight forward when a lazy loaded module  only has one component/route to load. Once you add a second, developers will have to be careful with the way they load the component. 

Let’s take a look at what the lazy routing  module (lazy.routing.ts), which would be imported by the Lazy Module (lazy.module.ts), might be:
{% highlight javascript %}
const lazyRoutes: Routes = [
	{
		path: '',
		component: LazyComponent
	},
	{
		path: 'brother',
		component: LazyBrotherComponent
	},
	{
		path: 'sister',
		component: LazySisterComponent
	}
];
{% endhighlight %}

Our Lazy Module now has three paths:
* http://myapp.com/lazy
* http://myapp.com/lazy/brother
* http://myapp.com/lazy/sister

In our example, the full path is broken up between the app.routing.ts file and the lazy.routing.ts file. Alternatively, the path in the app.routing.ts file could be left blank and the lazy routing could contain the full path. Although this will work, it is not best practice  .

Alternate routers:
{% highlight javascript %}
const appRoutes: Routes = [
	{
		path: '',
		component: HomeComponent
	},
	{
		path: 'eager',
		component: EagerComponent
	},
	{
		path: '',
		loanChildren: 'app/lazy/lazy.module#LazyModule'
	},
	{
		path: '',
		loanChildren: 'app/lazier/lazier.module#LazierModule'
	},
	{
		path: '404',
		component: _404Component
	},
{
		path: '**',
		redirectTo: '404'
	}

];
{% endhighlight %}

{% highlight javascript %}
const lazyRoutes: Routes = [
	{
		path: ' lazy ',
		component: LazyComponent
	},
	{
		path: ' lazy/brother',
		component: LazyBrotherComponent
	},
	{
		path: ' lazy/sister',
		component: LazySisterComponent
	}
];
{% endhighlight %}

{% highlight javascript %}
const lazierRoutes: Routes = [
	{
		path: ' lazier ',
		component: LazierComponent
	},
	{
		path: ' lazier/brother',
		component: LazierBrotherComponent
	},
	{
		path: ' lazier/sister',
		component: LazierSisterComponent
	}
];
{% endhighlight %}

The Angular routing system processes each route in the order they are typed. When provided with a URL, the system will start at the top and continue through each option until it has found the full path. If the path variable is left blank for a lazy loaded module, Angular must load the module to check if the path in the URL exists in the module. 

In our alternate example, we added a second lazy module to help illustrate our point. Since we left the path for both lazy modules blank, the application must load each to check if the path exists before moving on. If the user navigates to one of the lazier paths, the system will have to load the lazy module first to check if the path exists. With a larger application, this can become very costly when loading paths further down in the list. By breaking up the path between the app-routing and the lazy-routing, the application will only load the lazy module if the path begins with `/lazy` . 

Earlier in this blog, we mentioned 404 Page Not Found routing. In our example we have a 404 component in out router and redirect path. The ** wildcards indicates any route. By adding this path to the end of the router, any path that is not specified in the router will be caught here. This path could redirect the user to any screen in our application. Using a 404 Page Not Found screen will indicate to the user that they chose an invalid path without causing the application to error.

Improperly using the routing system can cause the application to load modules when the developer did not intend. By breaking up the full path between your routers, developers can confirm that modules will only be loaded when expected.
