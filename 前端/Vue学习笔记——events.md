## keycodes  

默认的keycodes：
	
	// src/compiler/codegen/events.js
	const keyCodes: { [key: string]: number | Array<number> } = {
	  esc: 27,
	  tab: 9,
	  enter: 13,
	  space: 32,
	  up: 38,
	  left: 37,
	  right: 39,
	  down: 40,
	  'delete': [8, 46]
	}

可以通过给键盘事件添加特定的keycodes:  

	<input @keydown.left="fn" /> // 敲击键盘“←”  
	
另外可以通过Vue.config添加自定义的keycodes:  

	Vue.config.keyCodes = {
		append: 65 // 键盘敲击"A"
	}  
	
	<input @keydown.append="fn" /> // 敲击键盘“A” 