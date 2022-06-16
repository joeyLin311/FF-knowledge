登录权限控制
======

点击登录后，我这里采用的是`mock`模拟数据，通过用户名去用户`常量list`里面进去查找，找到后返回对应的用户信息和`token`, 然后在请求头里面携带上`token`，如果不存在`token`或者`token`失效，需要进行登录，严格意义来说，登录时需要对密码和返回的 token 进行加密，不能以明文的方式传递，这里因为是自己项目，所以就没有进行这些。

登录后，拿到`token`信息和`userInfo`信息，存储在`vuex`和`session`里面 (当然你也可以不存`vuex`，看个人)，返回信息中最主要的是`roleIds`这个字段，这个字段表示的是当前用户所拥有的权限`id`

登录返回结果示例：

```jsx
  token: "admin2022012030427",
  userInfo: {
    date: "2020-10-13",
    id: 472467715762,
    key: "admin",
    label: "年轻人不讲武德",
    location: "Wuhan",
    password: "123456",
    position: "混元太极门掌门人",
    role: "admin",     
    roleIds: "1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36",
    roleString: "管理员"
    skill: "闪电五连鞭"
    text: "系统管理员，拥有所有权限"
    username: "admin"
  } 
 


```

请求头携带 token：

```jsx
service.interceptors.request.use(config => {
  if (store.getters.token) {
    config.headers['authorization'] = store.getters.token;
  }
  return config;
});


```

菜单权限控制
======

实现思路
----

常见的菜单权限控制有两种办法，前端控制和后端控制，不过最好是前端来进行控制，不然你想加一个菜单还必须要和后端说，让他加一个菜单，又回到被后端支配的恐惧。。。  
不过不管是前端控制还是后端控制，都需要拿到登录用户对应的菜单列表，然后通过`vue-router`的`addRoutes`方法将路由添加上去。

具体实现方法是，前端控制可以给路由表每一个路由都添加一个唯一 Id, 我之前的写法是直接给死`role:['admin','test']`这样写的，但是这样写有一个问题，如果某个菜单想修改权限或者是新加了一个角色，这样子还得重新修改代码，所以这次修改直接给每一个菜单一个对应的`roleId`, 之后就根据登录时的`roleIds`和路由表里面`roleId`进行比对，获取到当前用户对应的菜单。

```jsx
export const asyncRoutes = [
  {
    path: '/',
    component: Layout,
    redirect: '/index',
    hidden: true,
    children: [
      {
        name: 'index',
        path: '/index',
        component: () => import('@/views/index/index'),
        meta: {
          role: 1,
          title: '首页',
          icon: 'dashboard'
        }
      },

      {
        name: 'icon',
        path: '/icon',
        component: () => import('@/views/icon/index'),
        meta: {
          role: 2,
          title: '图标',
          icon: 'icon'
        }
      },
  ]


```

beforeEach
----------

这里还需要用到`vue-router`的一个钩子，`beforeEach`这个全局守卫，他有三个参数: to,from,next

* `to`: 你要跳转去的页面的`route`信息
* `from`：当前页面的`route`信息
* `next`: 需要使用这个方法来 resolve 钩子，不然无法成功进行页面跳转

例如：从 A 页面跳转到 B 页面，A 页面就是`from`，B 页面就是`to`

```jsx
router.beforeEach( (to, from, next) => {
  const localId = localStorage.getItem('userId');
  if (to.name === 'login') { // 访问login,缓存判断
    if (localId) { //有缓存，访问主页
      next({ ...to, replace: true });
    } else {// 无缓存，跳转登录页
      next('/login');
    }
  } 
});


```

我们可以单独把`beforeEac`h 抽离出来，建立`permission.js`文件，然后在`main.js`引入

权限控制
----

这里需要仔细调试下，如果判断没写完整，容易出现死循环，可以打断点仔细调试，`permission.js`大概如下：

```jsx
router.beforeEach(async (to, from, next) => {
  document.title = getPageTitle(to.meta.title); //title添加当前菜单名字
  const isLogin = getCache('TOKEN');  //根据token判断当前是否登录
  if (to.path == '/login') {  //如果是从其他页面来到登录页，就直接next
    next();
  } else {
    if (!isLogin) {  //如果未登录，强行跳转到登录，防止强行输入地址跳转
      next('/login');
    } else {
      const route = store.state.permission.routes;  //是否生成了路由菜单
      if (route.length > 0) {  //如果生成了，直接next
        next();  
      } else {              //如果未生成
        const userInfo = store.state.user.accountInfo;
        //根据用户信息的rolesIds进行递归遍历，拿到符合条件的路由菜单
        try {
          const { roleIds } = userInfo;
          const accountRoute = await store.dispatch('permission/getRoute', roleIds); //拿到路由菜单
          router.addRoutes(accountRoute);  //添加到路由上面去
          if (from.path == '/login') { //这里判断是防止首页不存在，所以直接取路由菜单的第一个
            next(accountRoute[0].children[0].path);
          } else {
            next({ ...to, replace: true });
          }
        } catch (error) {
          console.log(error);
          message.error('获取用户信息失败');
          next('/login');
        }
      }
    }
  }
});


```

获取路由菜单
------

路由表里面可以定好两种路由，一种是不需要权限控制的，如：登录页，404 页面等，一种是需要权限控制的； 这里`basRoute`就是基础路由，`asyncRoutes`是需要权限控制的路由，这里是`store`的`permission`文件，主要功能就是根据登录用户的 roleIds 获取对应的路由菜单

```jsx
import { baseRoute, asyncRoutes } from '@/router';
const state = {
  routes: []
};

const mutations = {
  SET_ROUTE(state, route) {
    state.routes = baseRoute.concat(route);
  }
};

const actions = {
  getRoute({ commit }, role) {  //这个的role是登录人的roleIds
    return new Promise((resolve, reject) => {
      let accessedRoutes = [];
      accessedRoutes = filterAsyncRoute(asyncRoutes, role);
      commit('SET_ROUTE', accessedRoutes);
      resolve(accessedRoutes);
    });
  },
};

//递归去遍历
export function filterAsyncRoute(routes, role) {
  let arr = [];
  routes.forEach(item => {
    const temp = { ...item };
    if (hasChildren(temp, role)) {
      if (temp.children) {
        temp.children = filterAsyncRoute(temp.children, role);
      }
      arr.push(temp);
    }
  });
  return arr;
}

//判断登录人的roleIds是否包含路由菜单的role
export function hasChildren(route, role) {
  let roleIds = role.split(',');
  if (route.meta && route.meta.role) {
    return roleIds.includes(String(route.meta.role));
  } else {
    return true;
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
};



```

侧边栏显示
-----

这里已经拿到了用户对应的权限，接下来就是将其渲染到侧边栏上面去，这里使用是递归遍历，由于`antd`根目录如果存在 div 标签就会导致渲染不出来，所以这里使用函数式组件的方法

```jsx
      <a-menu
        :mode="mode"
        :inline-collapsed="!collapsed"
        theme="dark"
        :selectedKeys="[$route.path]"
        :open-keys="openKeys"
        @openChange="changeOpen"
      >
        <template v-for="item in baseRoute">
          <menu-item v-if="!item.children && !item.hidden" :key="item.path :currentRoute="item" />
          <template v-else v-for="temp in item.children">
            <menu-item v-if="!temp.children" :key="temp.path" :currentRoute="temp" />
            <sub-menu v-else :key="temp.path" :currentRoute="temp"></sub-menu>
          </template>
        </template>
      </a-menu>


```

```jsx
subMenuItem.vue
 <a-sub-menu :key="props.currentRoute.path">
    <template slot="title">
      <svg-icon :icon="props.currentRoute.meta.icon" v-if="props.currentRoute.meta.icon"> </svg-icon>
      <span style="margin-left:16px" class="menu-title">{{ props.currentRoute.meta.title }}</span>
    </template>
    <template v-for="item in props.currentRoute.children">
      <menu-item v-if="!item.children" :key="item.path" :currentRoute="item" />
      <sub-menu v-else :key="item.path" :currentRoute="item" />
    </template>
  </a-sub-menu>


```

角色管理菜单
------

用户和角色绑定，角色和菜单绑定，所以要修改菜单就需要修改角色对应的 tree，这里有个小细节需要注意，因为 tree 是如果父节点被勾选，所有子节点都会被选中的，但是我们实际中，可能需要勾选父节点，但是只勾选一个子节点，所以这里需要单独进行控制下，也是需要递归遍历

```jsx
     mounted(){
        if (this.role) {
          //比对
          this.handleBatter(data);
        } else {
          this.checkKeyList = [];
        }
      }
    
     handleBatter(data) {
      //先递归拿到所有路由子节点
      const allRoute = this.getRouteId(data); //这个是获取所有路由的子节点集合
      let roleRoute = this.role.split(',');
      let newRoute = [];
      //进行比对，取交集
      allRoute.forEach(item => {
        roleRoute.forEach(val => {
          if (item == val) {
            newRoute.push(val);
          }
        });
      });
      this.checkKeyList = newRoute;  //这个就是选中的check
    },

    getRouteId(routes, role = null) {
      const res = [];
      routes.forEach(item => {
        if (item.children && item.children.length > 0) {
          res.push(...this.getRouteId(item.children, item.meta.role));
        } else if (item.meta && item.meta.role) {
          res.push(String(item.meta.role));
        } else if (role) {
          res.push(String(role));
        }
      });
      return res;
    },


```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bbd29e77c184d289ac22d67a168a997~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

页面按钮控制
------

道理同菜单一样，也是可以给页面按钮一个 id，根据用户信息的 roleIds 是否包含这个 id，来控制显示隐藏，不过需要代码注释和文档齐全，不然之后一个按钮不知道是啥用户才能看到，调试麻烦。

项目说明
====

[Vue Antd Admin](https://link.juejin.cn?target=http%3A%2F%2Fgithub.com%2Fbiubiubiu01%2Fvue-antd-admin "http://github.com/biubiubiu01/vue-antd-admin") 基于 vue-cli4.x+vuex+ant-design-vue 开发的后台管理系统，包括权限管理，多种布局方式，换肤功能，可视化拖拽，大屏展示，用户多角色和其他一些功能，有兴趣可以看看。

最后
==

到这里，系统后台管理权限已经完成了，前端控制权限办法有很多种，可以根据具体情况来使用，这次对 admin 系统修改了角色权限，用户添加多角色，换肤和外链等功能，有兴趣的可以看一下

其他文章
====

* [七. 使用 vue+antd 搭建后台管理系统 (需求分析和搭建篇)](https://juejin.cn/editor/drafts/6917437501728620558 "https://juejin.cn/editor/drafts/6917437501728620558")

* [六. 记一次 Vue3.0 尝鲜](https://juejin.cn/post/6870392360946106382 "https://juejin.cn/post/6870392360946106382")      \

* [五. 记一次用 webpack 搭建 vue 项目](https://juejin.cn/post/6844904183150149639 "https://juejin.cn/post/6844904183150149639")

* [四. 记一次用 ts+vuecli4 重构项目](https://juejin.cn/post/6844904164015734798 "https://juejin.cn/post/6844904164015734798")

* [二. Echarts+Amap 实现点击下钻功能](https://juejin.cn/post/6844903982377205768 "https://juejin.cn/post/6844903982377205768")

* [一. vue keep-alive 踩坑，删除 keep-alive 缓存](https://juejin.cn/post/6844903982301708302 "https://juejin.cn/post/6844903982301708302")
