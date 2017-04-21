# generator

```
// generator
function *gen() {
	const file1 = yield readFile('./file1.json', 'utf8')
	const file2 = yield readFile('./file2.json', 'utf8')
	console.log(file1, file2)
}
```

## 异步的解决方案
1. 使用callback
2. 包装成promise形式

## callback
1. 因为需要使用callback，所以`g.next().value`就应该是一个只接受callback的函数，所以就需要一个函数把多参数异步函数转化成只接受callback的函数 ===> thunk
	
	```
	// call
	const it = gen()
	it.next().value((error, data) => {
		it.next(data).value((error, data) => {
			it.next(data)
		})
	})
	```

2. thunk
	
	```
	const thunk = fn => (...arg) => cb => fn.call(null, ...arg, cb)
	```

3. 调用generator的时候不可能每次都手动调用，所以需要一个自动调用函数

	```
	const run = (gen) => {
		const it = gen()
		const next = (result) => {
			if(result.done) return result.value
			result.value((error, data) => {
				it.next(data)
				next()
			})
		}
		next(it.next())
	}
	```

4. 所以最后使用thunk函数的调用方式

	```
	// generator
	function *gen() {
		const file1 = yield readFileThunk('./file1.json', 'utf8')
		const file2 = yield readFileThunk('./file2.json', 'utf8')
		console.log(file1, file2)
	}
	// run
	run(gen)
	```

## promise
1. 首先，需要把异步操作包装成异步形式

	```
	const readFileAsync = (filePath) => Promise.resolve({ then(onFulfilled, onRejected) {
		fs.readFile(filePath, 'utf8', (err, data) => {
			if(err) return onRejected(err)
			onFulfilled(data)
		})
	}})
	```

2. 自动调用的函数也必须改成promise形式

	```
	const run = gen => {
		const it = gen()
		const next = (result) => {
			if(result.done) return result.value
			result.value.then((data) => {
				it.next(data)
				next()
			})
		}
		next(it.next())
	}
	```

## async
1. 是generator的语法糖，不需要自动调用函数，但是只支持promise形式

	```
	(async () => {
		const file1 = await readFileAsync('./file1.json')
		const file2 = await readFileAsync('./file2.json')
		console.log(file1, file2)
	})()
	```