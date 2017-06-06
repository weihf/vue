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

