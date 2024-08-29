```vue
<script>
import {
  defineComponent,
  getCurrentInstance,
  reactive,
  toRefs,
  ref,
  computed,
  watch,
} from 'vue'
import { Login } from '@/api/login'   // 导入登录发送请求所需要的js文件
import { useRouter, useRoute } from 'vue-router'		// 导入路由组件
import { useApp } from '@/pinia/modules/app'	// 从pinia中导入useApp模块
export default defineComponent({
  name: 'login',
  setup() {
    const route = useRoute()	// 获取当前路由对象
    const state = reactive({
      model: {  // 登录表单默认的用户名和密码
        userName: 'admin',
        password: '123456',
      },
      submit: () => { // 登录方法
        state.loginForm.validate(async valid => {
          if (valid) {  // 校验数据的合法性，合法执行下述代码
            state.loading = true
            const { code, data, message } = await Login(state.model)  // 调用登录方法
            if (+code === 200) {  // 返回的状态码如果为200，给出登录成功提示信息
              ctx.$message.success({
                message: ctx.$t('login.loginsuccess'),	// 读取i18n中locals/zh-cn/login.js中的内容
                duration: 1000,
              })

              const targetPath = decodeURIComponent(route.query.redirect)	
              if (targetPath.startsWith('http')) {
                // 如果是一个url地址
                window.location.href = targetPath
              } else if (targetPath.startsWith('/')) {
                // 如果是内部路由地址
                router.push(targetPath)
              } else {
                router.push('/')  // 登录成功以后进行路由 , 查看src/router/index.js的路由配置
              }
              useApp().initToken(data)  // 保存后端返回给前端的数据
            } else {  // 登录失败给出错误提示信息
              ctx.$message.error(message)
            }
            state.loading = false
          }
        })
      },
    })
    return {
      ...toRefs(state),
    }
  },
})
</script>
```