实现sidebar的方式

# 方法A

创建一个`AppLayout`组件

```tsx
export const Sidebar = ({ className, children, ...props }:ComponentProps<'aside'>) => {
	return (
		<aside className={twMerge('w-[250px] mt-10 h-[100vh + 10px] overflow-auto',className)} {...props}>
			{children}
		</aside>
)}
```


```css
.mt-10 {  
margin-top: 2.5rem /* 40px */;  
}
```

**`rem` 单位的定义**:
- `rem` 是相对于根元素（`<html>`）的字体大小来计算的。
- 如果根元素的字体大小设置为 16px，那么 1 `rem` 就等于 16px。
- 如果根元素的字体大小设置为 20px，那么 1 `rem` 就等于 20px。

 **`overflow-auto` 值**:
- `overflow-auto` 表示只有当内容超出容器的大小时，才会显示滚动条。
- 如果内容在垂直方向上超出容器的高度，会显示垂直滚动条。
- 如果内容在水平方向上超出容器的宽度，会显示水平滚动条。
