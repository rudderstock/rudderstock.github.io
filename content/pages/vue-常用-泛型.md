---
title: vue-常用-泛型
tags:
categories:
date: 2024-03-08
lastMod: 2024-03-13
---
# Props

  + ## ts props 变量声明和类型导出

    + 使用场景：用于在 `ts` 文件声明 `props`，对其进行约束，再配合 `ExtractPropTypes` 转换成规则的类型，进行类型导出。

```tsx
import { ComponentPropsOptions, ExtractPropTypes } from "vue";

const buildProps = <T extends ComponentPropsOptions>(data: T): Readonly<T> => {
	return data
}

export const ButtonProps = buildProps({
	name: {
		type: String,
		required: false,
	},
	text: {
		type: String,
		required: true,
	},
});

export type ButtonPropsType = ExtractPropTypes<typeof ButtonProps>;
```

    + 这样在声明 `props` 时，编辑器自动就会有相关代码的提示了，效果如下

![image.png](/assets/image_1709989041660_0.png)

![image.png](/assets/image_1710071196125_0.png)

    + `buildProps` 纯粹是用来进行约束和输出原始类型，给后面 `ExtractPropTypes` 使用

    + 如果直接声明：

```tsx
// ButtonProps 就只会是 ComponentPropsOptions
const ButtonProps: ComponentPropsOptions = {
  name: {
    type: String,
  }
}


```

    + 则会是

![image.png](/assets/image_1710302183283_0.png)

    + 通过 `buildProps`，类型则会是

![image.png](/assets/image_1710302360388_0.png)

  + ## PropType

    + 用于进行约束类型，例如：

```tsx
export default defineComponent({
  props: {
    book: Object as PropType<Book>,
    type: String as PropType<'primary' | 'warning'>
  }
})
```

    + 效果如下

![image.png](/assets/image_1710070997676_0.png)

  + ## 小结

    + `element-plus` 等一些组件库都支持导出对应的 `props` 类型，直接引入即可。

    + 但是如果碰到在 `.vue` 文件里声明 `props` 的组件，而且也没有导出对应 `.ts`，目前官方并未给出对应的泛型可以直接获取。

    + `.vue` 模板里引入还可以自己编写泛型进行获取，`.ts` 引入组件则根本无法获取对应的类型。。。。

## State

  + ## MaybeRef

    + `MaybeRef` 既可以是 `Ref` 类型，可以是基本数据类型，常用于封装 `hooks`

```tsx
type MaybeRef<T = any> = T | Ref<T>
```

    + 比如配合 `useQuery` 请求框架：

    + `api/user.ts`

```ts
export function useGetUserList(params: MaybeRef<UserPageListParams>) {
  return useQuery(['/user/list', params] as const, async ({ queryKey }) => {
    const [url, data] = queryKey;
    const result = await http.post<UserListResponse>(url, data);
    return result.data;
  });
}
```

    + `user.vue`

```ts
const params = ref({
	name: '你好',
})

const notRefParams = {
	name: '你好'
}

const { data } = useGetUserList(params); // 当数据发生变化后，会自动请求，例如 params.name = '不好'，则会自动进行请求
useGetUserList(notRefParams); // 也支持普通数据请求，只不过没有响应式
```

    + 


