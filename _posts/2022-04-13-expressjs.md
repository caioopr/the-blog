


# Express.js
categories: [node , express, javascript, backend]


## O que é ?
O Express.js é um framework NodeJs que disponibiliza recursos para construção de servidores web, facilitando a criação do back-end.
	
- Vantagens em relação ao NodeJs "puro":
			1. Pouco código 
			2. Liberdade para estruturar a aplicação
			3. facilidade de integração com o front-end

## Na prática

### Static Files 
Como exemplo vamos renderizar uma barra de navegação composta pelos seguintes arquivos:  index.html (estrutura), styles.css (estilização), main.js (funcionalidades) e logo.svg (logo).

Primeiro vamos ver como com o Node *vanilla*, **sem framework**. Então, com o projeto já iniciado:

1. Iniciamos o servidor: 

```js 
	const http = require('http') //built-in
	
	const server = http.createServer((req,res) => {
		res.end('Hellooo')
	}
	server.listen(5000)
```

2. Lemos os arquivos que compõem a barra de navegação com a função readFileSync:

```js 
	const http = require('http')
	const {readFileSync} = require('fs') // built-in
	
	const homePage = readFileSync('./navbar-app/index.html')
	const homeStyles = readFileSync('./navbar-app/styles.css')
	const homeImage = readFileSync('./navbar-app/logo.svg')
	const homeLogic = readFileSync('./navbar-app/main.js')
	
	const server = http.createServer((req,res) => {
		res.end('Hellooo')
	}
									 
	server.listen(5000)
```

3. Agora definimos o que o servidor vai responder quando a requisição desses arquivos for feita:

```js
	const http = require('http')
	const {readFileSync} = require('fs')

	const homePage = readFileSync('./navbar-app/index.html')
	const homeStyles = readFileSync('./navbar-app/styles.css')
	const homeImage = readFileSync('./navbar-app/logo.svg')
	const homeLogic = readFileSync('./navbar-app/browser-app.js')
  	
	// criando o servidor
	const server = http.createServer((req,res) => {
		
	const url = req.url; // a url que vem na requisição

	//home page
	if (req.url === '/') {
		res.writeHead(200,{'content-Type': 'text/html'});
		res.write(homePage);
		res.end();
	}
	//CSS
	else if(req.url === '/styles.css') {
		res.writeHead(200,{'content-Type': 'text/css'});
		res.write(homeStyles);
		res.end();
	}
	//Logo
	else if(req.url === '/logo.svg') {
	res.writeHead(200,{'content-Type': 'image/svg+xml'});
	res.write(homeImage);
	res.end();
	}

	//JS
	else if(req.url === '/main.js') {
		res.writeHead(200,{'content-Type': 'text/javascript'});
		res.write(homeLogic);
		res.end();
	}
		
	//erro 404, no caso do arquivo não existir
	else {
		res.writeHead(404,{'content-Type': 'text/html'});
		res.write('<h1>Page not found</h1>');
		res.end();
	}
})

server.listen(5000)	
```

Agora vamos ver como é utilizando o **Express**:

1.  criamos o servidor: 

```js
	const express = require('express')
	const app = express()
	
	app.listen(5000,() => {
		console.log('server is litening on port 5000...')
	})
```

2. definimos os arquivos que serão renderizados utilizando um middleware do Express:
```js
	const express = require('express')
	const app = express()
	
	//nesse caso todos os arquivos da barra de navegação estão em uma pasta nomeada 'public'
	//passa os arquivos estáticos para o middleware
	app.use(express.static('./public'))

	//nesse caso o all esta sendo usado para cobrir o erro de todos os metodos
	app.all('*', (req, res) => {
	res.status(404).send('Resource not found')
	})

	
	app.listen(5000,() => {
		console.log('server is litening on port 5000...')
	})
```

4. Se o index.html não estiver na mesma pasta dos outros arquivos da barra de navegação:

```js
	const express = require('express')
	const path = require('path') //built-in
	const app = express()
	
	//o index.html não esta na pasta public
	//passa os arquivos estáticos para o middleware
	app.use(express.static('./public'))
	
	app.get('/',(req,res) => {
		res.sendFile(path.resolve(__dirname,'./outrapasta/index.html'))
	})
	//nesse caso o all esta sendo usado para cobrir o erro de todos os metodos
	app.all('*', (req, res) => {
	res.status(404).send('Resource not found')
	})

	
	app.listen(5000,() => {
		console.log('server is litening on port 5000...')
	})
```

### Express Routes
Vamos supor que estamos fazendo um e-commerce e queremos criar uma rota para acessar todos os produto e outra para adicionar um produto. Sem o express as rotas seriam chamadas  de modo similar ao primeiro exemplo, mas com o express seria assim:

1. No arquivo app.js criamos o servidor, importamos as rotas que iremos criar e definimos o nome das rotas que iremos utilizar  :
```js

	const express = require('express')
	const app = express()
	
	//rotas 
	const productsRouter = require('./routes/prducts')
	
	// json parser
	app.use(express.json());
	
	// prefixo da rota products e o nome do Router que usa esse prefixo
	app.use("/api/products", productsRouter)

	// erro para bad request
	app.all('*', (req, res) => {
	res.status(404).send('Resource not found')
	})
	
	app.listen(5000,() => {
		console.log('server is litening on port 5000...')
	})
```

2. Criamos um arquivo separado com o nome desejado (no caso 'products.js' na pasta routes) e definimos o que vai acontecer quando a url que está sendo passada para router for chamada com métodos específicos:
```js
	const express = require('express')
	const router = express.Router()
	
	//controllers
	// funções que executam quando a rota "/api/products", definida no app.js é chamada com o método get ou post
	const getAllProducts = async (req,res) => {
		res.status(200).send('all products')
	}
	const createProduct = async (req,res) => {
		res.status(200).send('all products')
	}
	
	// nesse caso o route aponta para "/api/products/"
	router.route('/').get(getAllProducts).post(createProduct)

	module.exports = router
``` 

### Middlewares
Os middlewares são funções que excutam quando uma requisição é feita antes de uma resposta ser dada. Eles recebem como parametro a requisição, a resposta e o next, que indica o que fazer a seguir (a response).
	- req => middleware =>res
Para demonstrar, vamos pegar o exemplo anterior, mas agora a rota só pode ser
se o usuário estiver autenticado:

1. Primeiro criamos o middleware:
```js
	// middleware/authentication.js
	const authMiddleware = (req,res,next) => {
		try{
			metododeAutenticacao(req)
			next() // acaba o middleware e passa para a resposta
		} catch(err){
			throw new Error
		}
		 // end middleware
}
	module.exports = authMiddleware
``` 

2. Agora importamos o middleware e o passamos antes do router ser chamado no app.js:
```js
	const express = require('express')
	const app = express()
	const authMiddleware = require('./middleware/authentication') 
	//rotas 
	const productsRouter = require('./routes/prducts')
	
	app.use(express.json());
	// o express passa o os parametros automaticamente
	app.use("/api/products", authMiddleware,productsRouter)

	
	app.all('*', (req, res) => {
	res.status(404).send('Resource not found')
	})
	
	app.listen(5000,() => {
		console.log('server is litening on port 5000...')
	})
```
