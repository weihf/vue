# 阅读vue-router官方文档的学习笔记

官方文档：https://router.vuejs.org/zh-cn/

# 安装

### 直接下载 / CDN

链接：https://unpkg.com/vue-router/dist/vue-router.js

# NPM

如果在一个模块化工程中使用它，必须要通过Vue.use()明确地安装路由功能：

    npm install vue-router
    

    import Vue from 'vue'
    import VueRouter from 'vue-router'
    Vue.use(VueRouter)

如何使用全局的script标签，则无需手动安装。

# 基础

### 开始

用Vue.js+vue-router创建单应用，其实就是将组件(components)映射到路由(routes)，然后告诉vue-router在哪里渲染它们。

一个小例子：

### 动态路由匹配

1.把某种模式匹配到的所有路由，全都映射到同个组件，需要在vue-router的路由路径中使用“动态路径参数”。
例如：

	const User = {
				template: '<div>User{{$route.params.id}}</div>'
			};
			
	const router = new VueRouter({
		routes: [{
			path: '/user/:id',
			component: User
		}]
	})

当匹配到一个路由时，参数值会被设置到this.$route.params。

2.响应路由参数的变化（watch）

当使用路由参数时，原来的组件实例会被复用，所有的路由都渲染同一个组件。好处是利用高效，坏处是组件的生命周期钩子不会再被调用。
复用组件时，想对路由参数的变化作出响应，可以使用watch监测$route对象：

	const User = {
				template: '<div>User {{$route.params.id}}</div>',
				watch:{
					'$route'(to,from){
						console.log(this.$route.params.id)
					}
				}
			}
			

3.　高级匹配模式

path-to-regexp 

4.　匹配优先级

路由匹配的优先级：谁先定义的，谁的优先级就最高。

### 嵌套路由

用于多层组件嵌套，URL中各段动态路径对应嵌套的组件。

一个<router-view>中可以嵌套另一个<router-view>:

	const User = {
				template: `
					<div>
						<h2>User {{$route.params.id}}</h2>						
						<router-view></router-view>
					</div>
				`
			}

在要嵌套的出口中渲染组件，需要在VueRouter的参数中使用children配置：

const router = new VueRouter({
				routes: [{
					path: '/user/:id',
					component: User,
					children: [{
							path: '',
							component: {
								template: '<div>home</div>'
							}
						}, {
							path: '/profile',
							component: {
								template: '<div>profile</div>'
							}
						},
						{
							path: 'posts',
							component: {
								template: '<div>posts</div>'
							}
						}
					]
				}]
			})

要注意，以　/　开头的嵌套路径会被当作根路径。

### 编程式的导航

除了使用<router-link>创建a标签来定义导航链接，还可以借助router的实例方法，通过编写代码来实现。

#### router.push(location)

这个方法会向history栈添加一个新的记录，当用户点击浏览器后退按钮时，则回到之前的URL。
点击<router-link>时，这个方法会在内部调用。所以说，点击<router-link　:to="...">等同于调用了router.push(...)。

声明式：<router-link　:to="...">
编程式：router.push(...)

该方法的参数可以是一个字符串路径，或者一个描述地址的对象。例如：
	//字符串
	router.push('home')
	
	//对象
	router.push({path:'home'})
	
	//命名的路由
	router.push({name:'user',params:{userId:123}})
	
	//带查询参数，变成/register?plan=private
	router.push({path:'register',query:{plan:'private'}})

#### router.replace(location)

跟router.push很像，唯一的不同就是，它不会向history添加新记录，而是替换掉当前的history记录。

声明式：<router-link　:to="..." replace>
编程式：router.replace(...)

#### router.go(n)

这个方法的参数是一个整数，意思是在History记录中向前或者后退多少步，类似window.history.go(n)

	//前进一步，等同于history.forward()
	router.go(1)
	
	//后退一步，等同于history.back()
	
	//前进3步
	router.go(3)
	
如果history记录不够，就失败。

### 命名路由

在routes配置中给路由设置名称：

	const router = new VueRouter({
				routes: [{
					path: '/user/:id',
					component: Foo,
					name:'foo' //命名
				}]
		})

要链接到一个命名路由，需要在router-link的to属性传一个对象：

	<router-link :to="{name:'foo',params:{id:123}}">/user/foo</router-link>

这跟router.push(location)等同：

	router.push({name:'foo',params:{id:123}})

### 命名视图

用于同时（同级）展示多个视图router-view，一个视图对应一个组件渲染。
在router-view中设置name即可，如果没有设置则默认name为default

	<router-view></router-view>
	<router-view name="a"></router-view>
	
一个路由设置多个视图，一个视图对对应一个组件，因此就需要添加多个组件，所有的组件都设置在components（带上s）:


			const router = new VueRouter({
				routes: [{
						path: '/foo',
						components: {
							default: {
								template: '<div>home</div>'
							},
							a: A,
							b:B
						}
					},
					{
						path: '/bar',
						components: {
							a:A
						}
					}
				]

### 重定向 和 别名

#### 重定向
重定义的意思是，当用户访问/a时，URL会被替换成/b，然后匹配
重定向通过routes来配置redirect，重定向的目标可以是一个命名的路由，也可以是一个方法用来动态返回重定向目标

	//绝对
	redirect:'/bar'
	
	//路由
	redirect: {
				name: 'bar'
			  }
	
	//方法
	redirect: function(to) {
		if(to.path === '/foo') {
			return '/bar'
		}
	}

#### 别名

/a的别名是/b，意味着，当用户访问/b时，URL会保持为/b，但路由匹配则为/a，就像用户访问/a一样。

	
			const router = new VueRouter({
				routes: [{
					path: '/foo',
					alias:'/bar',
					components: {
						default: {
							template: '<div>foo</div>'
						}
					}
				}, {
					path: '/bar',
					name: 'bar',
					components: {
						default: {
							template: '<div>bar</div>'
						}
					}
				}]
			})

### HTML5 History 模式

vue-router默认hash模式--使用URL的hash来模拟一个完整的URL，于是当URL改变时，页面不会重新加载。
如果不想要hash，可以用路由的history模式，这种模式充分利用history.pushState API来完成URL跳转而无须重新加载页面。

	const router = new VueRouter({
		mode: 'history',
		routes:[...]
	})

使用history模式需要后台配置支持。

# 基础

### 导航钩子

导航钩子主要用来拦截导航，让它完成跳转或取消。

#### 全局钩子

使用router.beforeEach注册一个全局的before钩子：
			
	router.beforeEach(function(to,from,next){
		next();
	})

当一个导航触发时，全局的before钩子按照创建顺序调用。钩子是异步解析执行，此时导航在所有钩子resolve完之前一直处于等待中。

每个金子方法接收三个参数：

- *to: Route*: 即将要进入的目标路由对象

- *from: Route*: 当前导航正要离开的路由

- *next: Function*: 一定要调用该方法来resolve这个钩子。执行效果依赖next方法的调用参数。

  - *next()*:进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是confirmed（确认的）。
  - *next(false)*: 中断当前的导航。如果浏览器的URL改变了（可能是手动或者浏览器后退按钮），那么URL地址会重置到from路由对应的地址。
  -　*next('/')*或者next({path:'/'}):跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
  
  确保要调用next方法，否则钩子就不会被　resolved。
  
  同样可以注册一个全局的after钩子，不过它不像before钩子那样，afeter钩子没有next方法，不能改变导航：
  
	router.afterEach(function(route){
		//..
	})

#### 某个路由独享的钩子

在路由配置上直接定义beforeEnter钩子：

	const router = new VueRouter({
		routes:[{
			path:'/foo',
			component:Foo,
			beforeEnter:function(to,from,next){
				//..
			}
		}]
	})

#### 组件内的钩子

- beforeRouteEnter
- beforeRouteUpdate
- beforeRouteLeave

	const Foo = {
		template:`...`,
		beforeRouterEnter(to,from,next){
			//在渲染该组件的对应路由被confirm前调用
			//不能获取组件实例this
			//因为当前钩子执行前，组件实例还没被创建
		},
		beforeRouterUpdate(to,from,next){
			//在当前路由改变，但是该组件被利用时调用
			//举例来说，对于一个带有动态参数的路径/foo/:id,在/foo/1和/foo/2之间跳转的时候，
			//由于会渲染同样的Foo组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
			//可以访问组件实例this
		},
		beforeRouteLeave(to,from,next){
			//导航离开该组件的对应路由时调用
			//可以访问组件实例this
		}
	}

### 路由元信息

定义路由的时候可以配置meta字段：

	const router = new VueRouter({
		routes:[{
			path:'/foo',
			component:Foo,
			meta:{
				requiresAuth:true
			}
		}]
	})

路由记录：routes配置中的每个路由对象

路由记录可以嵌套，当一个路由匹配成功后，他可能匹配多个路由记录。

一个路由匹配到的路由记录会暴露为$route对象（还有在导航钩子中的route对象）的$route.matched数组。需要遍历$route.matched来检查路由记录中的meta字段。
