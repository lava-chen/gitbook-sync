创建`DraggableTopBar`

1. 在全局css中添加
```css
header {

-webkit-app-region: drag;

}
```
2. 创建component
```tsx
export const DraggableTopBar = () => {

return <header className="absolute inset-0 h-8 bg-transparent" />

}
```

```css
.absolute {  
	position: absolute;  
}
  
.inset-0 {  
	top: 0px;  
	right: 0px;  
	bottom: 0px;  
	left: 0px;  
}
.h-8 {  
	height: 2rem /* 32px */;  
}
```

3. 添加在APP组件中即可