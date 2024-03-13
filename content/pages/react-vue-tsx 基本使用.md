---
title: react-vue-tsx 基本使用
tags:
categories:
date: 2024-03-10
lastMod: 2024-03-12
---
tags:: vue, react

> 本文主要面向 `react` 开发人员快速上手 `vue` 的 `.tsx` 组件开发

# Tsx

  + 在 `vue` 项目中，如果不太熟悉、不想使用 `vue` 模板语法的开发方式，可以通过 `vite` 相关插件，使用 `tsx` 进行组件编写

  + 优点：

    + 可以使用更加灵活的 `jsx` 来进行开发
logseq.order-list-type:: number

    + 可以单个文件里可以输出多个小组件
logseq.order-list-type:: number

    + 更加灵活的`js`配置，不需要忍受 `.vue` 模版里的的魔法配置
logseq.order-list-type:: number

    + 减少 `all in one`（页面组件代码 4000 + 行） 概率，文件内可以各种变量导出，重复内容也可以很方便的进行管理
logseq.order-list-type:: number

    + 使用 `model.confirm` 等函数方式调用组件，可以很方便的个性化自定义标签，不需要频繁塞进 `template` 里，再声明 `ref` 来进行各种控制
logseq.order-list-type:: number

  + 缺点：

    + 无法使用 `css scope`，可以使用 `css-modules` 或者其他方式来进行样式隔离，习惯了 `:deep` 等方式的开发人员可能会不太适应
logseq.order-list-type:: number

    + 其他成员难维护，未接触过 `react` 框架的 `vue` 开发人员，大概率无法接受和学习开发，需要靠你自己维护或者培训他们
logseq.order-list-type:: number

    + `.tsx` 引用自己项目创建的 `.vue` 的组件，没有相关类型声明，需要自己手动或者通过其他手段添加声明类型
logseq.order-list-type:: number

    + `vue` 重点还是放在 `.vue` 组件，后续 `.tsx` 更新优化力度不会太大
logseq.order-list-type:: number

    + `ref.value` 这个 `.value` 使用时还是有点烦，绑定时经常需要注意
logseq.order-list-type:: number

## Babel 设置

  + 安装相关插件

```shell
npm i @vitejs/plugin-vue-jsx -D
```

  + `vite.config.ts`

```ts
// vite.config.js
import vueJsx from '@vitejs/plugin-vue-jsx'

export default {
  plugins: [
    vueJsx({
      // options are passed on to @vue/babel-plugin-jsx
    }),
  ],
}
```

## 创建组件

  + 有以下几种方式创建组件

```tsx
// 选项语法
function defineComponent(
  component: ComponentOptions
): ComponentConstructor

// 函数语法 (需要 3.3+)
function defineComponent(
  setup: ComponentOptions['setup'],
  extraOptions?: ComponentOptions
): () => any


// 函数组件
export const Button = props => {
  return <div>{props.name}</div>;
};
```

  + ### defineComponent

    + 可以通过创建 `props` 变量，进行管理和复用

    + 选项语法和函数语法只是写法不一样，看个人习惯

```tsx
const ButtonProps = {
	name: {
		type: String,
		default: 'hello'
	}
}

// 函数语法
export default defineComponent((props, { emit }) => {
  return () => {
    return <div>{props.name}</div>;
  };
}, {
  props: ButtonProps
});

// 选项语法
export const Button = defineComponent({
	props: ButtonProps,
	setup(props, ctx) {
		return () => {
			return <div></div>
		}
	}
})
```

  + ### 函数组件

    + 该方式无法函数内创建 `State`，能不用就不用，只适用于非常简单的组件，例如下面示例：

```tsx
import { ref } from 'vue'

const RcButton = () => {
	const count = ref(0)
	function onClick() {
		count.value++
	}
	return (
		<div>
			<div> 数字 : {count.value}</div>
			<button onClick={onClick}>点我+1</button>
		</div>
	)
}
```

    + 点击按钮后，视图仍然显示为 0，点击事件无任何作用

  + ## 是否能使用 `React.FC<Props>` 方式创建函数组件

    + 是否能像 `const component: FC<Props> = (props) => <div></div>` 快速生成组件

    + 只能说，看起来可以，但基本上不行，还是要写传统的 `Props` 变量来进行约束，如下所示：

    + ## defineComponent

      + 看起来似乎可以传定义过去，而且使用时，编辑器也有相关提示

```ts
import { defineComponent } from 'vue'

interface ButtonProps {
	type: 'primary' | 'ghost',
	content: string
}

export default defineComponent<ButtonProps>((props) => {
	return () => {
		return <div>{props.content}</div>
	}
})
```

![image.png](/assets/image_1710075884899_0.png)

      + 但组件使用后，传的值并没有传到 `props` 上，而是 `attrs` 上，只是看起来像生效了而已

![image.png](/assets/image_1710076082940_0.png)

    + ## FunctionalComponent

      + 虽然可以通过泛型来快速定义，`props` 也有值，编辑器有类型提示，但无法定义 `State`，基本上无用

```tsx
interface ButtonProps {
	type: string
}
const RcButton: FunctionalComponent<ButtonProps> = (props, { attrs }) => {
	console.log('props', props)
	console.log('attrs', attrs)
	return (
		<div>
			<div>{props.type}</div>
		</div>
	)
}
```

      + `props` 和 `attrs` 传值一样，将就的话怎么说也能用

![image.png](/assets/image_1710076451064_0.png)

## 基本使用

  + 如下所示：简单的点击按钮，数字加一

```tsx
import { defineComponent, type PropType } from 'vue'
import { useVModel } from '@vueuse/core'

export default defineComponent({
	props: {
		type: String as PropType<'primary' | 'warning'>,
		modelValue: {
			type: Number,
			required: true
		}
	},
	emits: ['update:modelValue'],
	setup(props, { emit }) {
      
		const count = useVModel(props, 'modelValue', emit, {
			defaultValue: 0
		})
		function onButtonClick() {
			count.value += 1
		}

		return () => {
			return (
				<div>
					<div>数字是 {count.value}</div>
					<button onClick={onButtonClick}>点我+1</button>
				</div>
			)
		}
	}
})
```

  + 使用下来，结构上大概可以分三个区域：

![image.png](/assets/image_1710077628671_0.png)

  + 第一区域：用于定义 `props`、`emit`、`slots` 等相关外部属性；
logseq.order-list-type:: number

  + 第二区域：用于定义声明 `state` 组件内部状态，比如创建各种变量、`ref`、`watch`、`computed` 等相关操作，一些性质类似于 `React.Component` 的 `constructor`，可以定义类型，只执行一次，不过能添加 `watch`，生命周期等相关操作；
logseq.order-list-type:: number

  + 第三区域：可以理解为 `render` 的函数，只要发生渲染，就会执行；
logseq.order-list-type:: number

  + 

  + 刚开始接触 `.tsx` 可能很难绕过来弯，初次上手，可以 `setup` 下当成 `constructor` 来使用，应该能稍微好上手一点。

## 后续

  + 


