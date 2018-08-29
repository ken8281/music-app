# music-app
仿写移动端音乐Web APP，数据来源于QQ音乐

**注意：** 由于项目部署在国外，并且是单页应用，首次加载会比较久，若首屏的图片不能正常加载，请刷新页面

[项目演示地址](https://ken8281-music-app.herokuapp.com/)

![二维码](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAARgAAAEYCAIAAAAI7H7bAAAG5ElEQVR4nO3dUY4jKRBFUddo9r/lngUMJYHiRYBb53y2quwsu6+QSBJ+/vz58wFq/rl9AfA3EBIECAkChAQBQoIAIUHAv8t//fn5Gb6OHcuZ+uWl7v9k/WWXjt7r6BUqr3n0svX3Kvqi/4QfIxJECAkChAQBQoIAIUGAkCBgPf299ObU59FM9/571V/2iy5gaXKuf9+b/wk/RiSIEBIECAkChAQBQoIAIUHAwfT3Un1C9s3dV5oWX3e8wrNfQXEFffGNjtSvyogEAUKCACFBgJAgQEgQICQIqE5/X3d985OjSd6xGeHfdOy+wseIBBFCggAhQYCQIEBIECAkCPj66e/ru5Q0vezy76pPlF9/r+s3AJoYkSBASBAgJAgQEgQICQKEBAFCgoDqfaQvugPwRZf6md3GqOmZkTHXL+BjRIIIIUGAkCBASBAgJAgQEgQcTH+/ua1M0y5C+z/ZtIP75JR0xwWc/vCmN/8TfoxIECEkCBASBAgJAoQEAUKCgPX09wvLaTc1XerkJzC5pLpjrv/6pb7AiAQBQoIAIUGAkCBASBAgJAiobn5SXyO8b3JCtmnvkeInU/8Mix9X/Zu9/iUWT9H97deNSBAgJAgQEgQICQKEBAFCgoCfsTW2k5Okk4uUj96r425B004vdR0blUz+saa/YZqQIEBIECAkCBASBAgJAtarvzvWPh/NRXbMX9e3qK7PHTd9XPu/Pjb/fuT6BdQZkSBASBAgJAgQEgQICQKEBAFCgoCDxyjeXNg/eaho0x2n63/CvqZnYYq//sJNMyMSBAgJAoQEAUKCACFBgJAgoLqJfn1X9eLk7+SG/fsmZ7TrOnbWr2s6n6HpqzEiQYCQIEBIECAkCBASBAgJAqq7CC1d39Wlvvz8zU+gvkh5cl6+4wjX+urvfUdfohEJAoQEAUKCACFBgJAgQEgQsJ7+3p8RnpyQvT6rvnR9+fn1qfb6yzb9CcWvxhmyME1IECAkCBASBAgJAoQEAQd7fx+8aM/k7/WJ8qbtODo2Wql/ApNf4uQxvkv2/oYnCAkChAQBQoIAIUGAkCBASBBQvY9Uv7XScbdh/42O3qvj1397haU3HyR51uQ9KyMSBAgJAoQEAUKCACFBgJAg4OAM2cnl+h1b87xwrmvHZ1h/ZOOFT+b/3ny2wmMU0EhIECAkCBASBAgJAoQEAQdnyE7u7HN99XfTjHBxUXnxNZs0HeE6uYt/nREJAoQEAUKCACFBgJAgQEgQUD1D9otWfx8pzlM3baI/+Wnv//rRtjb73tzF/zdGJAgQEgQICQKEBAFCggAhQcDc6u/fdKzpfnY/kMmZ7v2XfXZHkbvs/Q3ThAQBQoIAIUGAkCBASBCwPvqyY/L3+jrxpqMv666fvXnd9QcL6oxIECAkCBASBAgJAoQEAUKCACFBwMEZsk0r8Mf2BprchOhoY53inY1nTyfYv4D9nzy61MkjC4xIECAkCBASBAgJAoQEAUKCgPVjFAe///1r+CeN7Y7UNCfeZPLA3KaHcYxIECAkCBASBAgJAoQEAUKCgOom+k3enA+d1LRWvWPxdf0VinP99c+q/gpGJAgQEgQICQKEBAFCggAhQcB6+nvyWNXiBfytk9fF9chH+4FMHpj7t54YYESCACFBgJAgQEgQICQIEBIE3F/9PbYfyJE35/qblrqP/fowZ8jClxESBAgJAoQEAUKCACFBgJAg4P5jFB0nqH7X7Y6lpicLOs6Q/U3Hy755M/NjRIIIIUGAkCBASBAgJAgQEgSsz5BtOmfz4LIGJ0mv72szeWLA/gU0fd1vPp9SfM2PEQkihAQBQoIAIUGAkCBASBBwsPq7+JNHirPPk5Py9T3siz/ZtPq7bmxa/2hGu2mpuxEJAoQEAUKCACFBgJAgQEgQcLCJ/nXLicv6bOb17VPGzlo9eq/r3nyw4DdGJAgQEgQICQKEBAFCggAhQcB6+nupaY3w0v4cZdNm1m8uiG6aES5+XE1T7dcfLDhiRIIAIUGAkCBASBAgJAgQEgQcTH8vNR2Q2OHZLar3L2By/5b6LYT6BHpRx60Re39DIyFBgJAgQEgQICQIEBIECAkCqveRrpt83KDu+tVev4B91zcMOroAIxIECAkChAQBQoIAIUGAkCDg66e/m851bVps3zHRXN8dqXgB159PaXqW5+jLMiJBgJAgQEgQICQIEBIECAkCft5cDtx0guq+yU30J8+Q3ffm1339bN/fGJEgQEgQICQIEBIECAkChAQBB6u/r+8rv69pPfLkKuOON6qr3xe5Pn/dcTzxx4gEEUKCACFBgJAgQEgQICQIWK/+Bo4YkSBASBAgJAgQEgQICQKEBAFCgoD/AJWVsFuxM1bJAAAAAElFTkSuQmCC)

## 主要技术栈
初始化项目代码：vue-cli-v2.96  
MVVM框架：Vue.js （版本2.x）  
状态管理：Vuex  
前端路由：Vue Router  
服务端通讯：axios、jsonp  
移动端滚动库：better-scroll  
技术语言：HTML、Stylus、ES6

## TODO
使用ssr优化体验

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```



