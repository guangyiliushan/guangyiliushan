# Vue3 实践规范(vite)

## src 目录

- components 目录：存放所有的 Vue 组件。按照功能模块划分子目录，例如user/components存放与用户相关的组件，product/components存放与产品相关的组件。
- views 目录：存放页面级别的组件，这些组件通常是路由视图的内容。例如，HomeView.vue、AboutView.vue。
- store 目录：用于存放 Pinia 的相关文件。每个模块（如用户模块、产品模块）可以有自己独立的.ts文件，用于定义 store。
- router 目录：存放 Vue Router 相关的配置文件，如index.ts，用于定义路由和路由守卫。
- types 目录（可选）：如果有复杂的类型定义，可以将它们集中放在这个目录中，方便管理和复用。
- assets 目录：存放图片、样式文件（如 CSS、SCSS）等静态资源。
