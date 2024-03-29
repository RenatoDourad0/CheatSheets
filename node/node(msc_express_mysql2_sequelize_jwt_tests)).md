# node.js
## express
### ambiente
  - `npm init -y` 
  - `npm i express express-async-errors@3.1 cors@2.8 body-parser morgan mysql2@2.3 sequelize dotenv@16.0.1 joi@17.6 jsonwebtoken@8.5 bcrypt`
  - `npm i -D sequelize-cli sequelize-test-helpers nodemon mocha@10.0 chai@4.3 chai-http@4.3 sinon@14.0 sinon-chai nyc@15.1` 
    - versões especificas somente para trybe
  - `npm init @eslint/config` - verificar plugins e regras no arquivo .eslintrc.json - [docs](https://eslint.org/docs/latest/user-guide/configuring/configuration-files)
    - eslint para trybe: 
      - `npm i eslint@6.8 eslint-config-trybe-backend@1.0 -D` 
      - ```
        // eslintrc.json
        {
        "env": {
          "es2020": true
        },
        "extends": "trybe-backend",
        "rules": {
          "sonarjs/no-duplicate-string": ["error", 5]
        }
        }
        ```
  - `touch .env` na raiz da aplicação
```js
MYSQL_HOST=<ip-ou-nome-host>
MYSQL_PORT=<prota> // 3306
MYSQL_USER=<nome-usuario>
MYSQL_PASSWORD=<senha-usuario>
MYSQL_DATABASE_NAME=<nome-banco-de-dados>
// somente mysql2
MYSQL_WAIT_FOR_CONNECTIONS=true
MYSQL_CONNECTION_LIMIT=10
MYSQL_QUEUE_LIMIT=0
```
  - `touch .env-example` - copia da estrutura acima para ser enviado ao gitHub como orientação de preenchimento
  - `touch .eslintignore` - node_modules, ./*.config.js
  - `git init`
  - `touch .gitignore` - node_modules, .env
  - criar pastas src e tests
    - em src a pasta middlewares, routes e db, essa ultima somente para mysql2
      - em routes arquivos com rotas usando o middleware routes 
      - em middlewares arquivos com funções que auxiliam as rotas 
      - (somente para mysql2) em db arquivo connection e arquivos com prepared statements (funções com chamadas a DB) para as rotas
    - em tests as pastas unit e integration
      - arquivos <nome>.test.js 
  - adicionar ao package.json
```js
"main": "src/server.js"
"scripts": {
"prestart": "npx sequelize db:drop && npx sequelize db:create && npx sequelize db:migrate",
"start": "node src/server.js",
"dev": "nodemon src/server.js",
"lint": "eslint --no-inline-config --no-error-on-unmatched-pattern -c .eslintrc.json .",
"lint:fix":"eslint --fix --ext .js,.jsx ." // adicionar extenções desejadas,
"tests":"mocha tests/**/*.test.js --exit",
"test:coverage": "nyc --all --include src/models --include src/services --include src/controllers mocha tests/unit/**/*.js --exit"
},
```
- em `/src` `npx sequelize-cli init` 
	- diretórios config, migrations, models e seeders serão gerados automaticamente
	- alterar /config/config.json para config.js
	- alterar linha 9 do arquivo src/models/index.js para apontar para o arquivo config.js
	- criar arquivo .sequelizerc na raiz da aplicação
- `git commit -am 'definição ambiente'`
	
### inicializando
```js
// src/config/config.js (sequelize)
require('dotenv').config();

const config = {
  username: process.env.MYSQL_USER,
  password: process.env.MYSQL_PASSWORD,
  database: process.env.MYSQL_DATABASE,
  host: process.env.MYSQL_HOST,
  dialect: 'mysql',
};

module.exports = {
  development: config,
  test: config,
  production: config,
};
	
// /.sequelizerc
const path = require('path');

module.exports = {
  'config': path.resolve('src', 'config', 'config.js'),
  'models-path': path.resolve('src', 'models'),
  'seeders-path': path.resolve('src', 'seeders'),
  'migrations-path': path.resolve('src', 'migrations'),
};
	
// src/app.js
const express = require('express');
require('express-async-errors');
const morgan = require('morgan');
const cors = require('cors');
const bodyParser = require('body-parser');
	
const port = process.env.PORT || 3001;
const app = express();
	
app.use(morgan('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(cors({
    origin: `http://localhost:${PORT}`,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Authorization'],
  }));
// ...
module.exports = app;
	
// src/server.js
const app = require('./app');
const connection = require('./db/connection'); // mysql2
require('dotenv').config();
  
const port = process.env.PORT || 3001;
app.listen(port, async () => {
  console.log(`API TrybeCash está sendo executada na porta ${port}`);
// codigo para testar conexão com db. Pode retirar depois de verificado, deixando somente a linha acima
  const [ result ] = await connection.execute('SELECT 1');
  if (result) {
    console.log('MySQL connection OK');
  }
});
	
// src/db/connection.js (mysql2)
const mysql = require('mysql2/promise');
const connection = mysql.createPool({
  host: process.env.MYSQL_HOST,
  port: process.env.MYSQL_PORT,
  user: process.env.MYSQL_USER,
  password: process.env.MYSQL_PASSWORD,
  database: process.env.MYSQL_DATABASE_NAME,
  waitForConnections: process.env.MYSQL_WAIT_FOR_CONNECTIONS,
  connectionLimit: process.env.MYSQL_CONNECTION_LIMIT,
  queueLimit: process.env.MYSQL_QUEUE_LIMIT,
});
module.exports = connection;

// tests/integration/foo.test.js
const app = require('../../src/app')
const chai = require('chai');
const chaiHttp = require('chai-http');
const sinon = require('sinon');
const connection = require('../../src/db/connection');

const { expect, use } = chai;
use(chaiHttp);
```

### definições
- sintaxe
```js
app.get('/', (req, res, next) => {
  try {
  res.status(200).json({ message: 'Olá Mundo!' }));
} catch(e) {
  res.status(500).json({message: e.message});
}})
```
  
- sintaxe rotas
  - `'/ab?cd'` o caractere anterior ao `?` (b) é opcional
  - `'/ab+cd'` o caractere anterior ao `+` (b) pode se repetir
  - `'/ab*cd'` o `*` representa um conjunto de caracteres aleatórios
  - `'/a(bc)?de'` o `( )` agrupa caracteres

- parâmetros
  - parâmetro `res`
    - res.download()	Solicita que seja efetuado o download de um arquivo
    - res.end()	Termina o processo de resposta
    - res.json()	Envia uma resposta JSON
    - res.jsonp()	Envia uma resposta JSON com suporta ao JSONP
    - res.redirect()	Redireciona uma solicitação
    - res.render()	Renderiza um modelo de visualização
    - res.send()	Envia uma resposta de vários tipos
    - res.sendFile	Envia um arquivo como um fluxo de octeto
    - res.sendStatus()	Configura o código do status de resposta e envia a sua representação em sequência de caracteres como o corpo de resposta
  - parâmentro `req`
    - req.params acessa os paramentros da requisição
    - req.query acessa as queries da requisição
    - req.path acessa o caminho da requisição
    - req.body acessa o corpo da requisição
    - req.headers acessa os valores passados no header da requisição
    
- requisição GET por query
  - `req.query` 
  - `/rota?variavel1=valor&variavel1=valor&variavelN=valor`
    - `/rota` é a rota a se buscar
    - `?` é o inicio da query
    - `variavel1=valor` são os valores a serem passados
    - `&` é o separador entre valores
    - dado disponivel na chave `query` sempre em formato de string

- requisição GET por parametros
  - `req.params`
  - `/rota/:variavelX/:varivelZ` ex: /user/3345/active
    - `/rota` é a rota a se buscar
    - `/:` o indicador de que um parametro será passado
    - `variavelN` o valor a ser recebido
    - dado disponivel na chave `params` sempre em formato de string
      
- requisições com body
  - desestruturar as propriedades do body e depois re-estruturar. (para não trazer mais informações do que o necessário)
```js
app.put('<route>', async (req, res) => {
  try {
    const { id } = req.params;
    const { name, brandId } = req.body;
    // ...
  } catch (e) {
    if (...) return res.status(404).json({ message: ""});
    return res.status(500).end();
  }
}) 
```

- middlewares
  -  podem ser criados (função) ou instalados (pacote) e aplicados globalmente ou na chamada da rota
  - ex: `express.json(), express.static(<path>), morgan('dev'), cors()`
    - static: permite servir arquivos estáticos em um caminho automaticamente
    - morgan: produx logs da aplicação
    - cors: permite definir politica de cors. Conexão entre front e back 
  - middlewares globais podem ser definidos em `app.use(<middleware>)`. Somente as rotas abaixo dessa declaração passarão por esse middleware
  - middlewares locais são definidos como uma função que tem acesso aos paramentros `req, res, next` e podem fazer uma lógica e responder a requisição ou  passar para o proximo middleware com `next()`
  -  Express já vem com um middleware de erro padrão pronto para lidar com a maior parte dos casos comuns através de um retorno em html. Este middleware só entra em ação se a requisição chegar ao final do código sem ser respondida. Caso queira reconfigurar a mensagem de resposta basta definir no final do código um app.use com uma chamada de rota com a mensagem desejada. Ex. `app.use((req, res) => res.sendStatus(404))`
```js
// global
app.use(express.json());

// local
function meuMiddleware(req, res, next) {
// ...
next();
};
app.get('<rota>', meuMiddleware, (req, res) => {
// ...
});
```

  - middleware de erro
    - sempre devem vir depois de rotas e outros middlewares
    - só recebem requisições se algum middleware lançar um erro ou chamar next(err)
    - sempre devem receber quatro parâmetros
    - O Express utiliza a quantidade de parâmetros que uma função recebe para determinar se ela é um middleware de erro ou um middleware comum
    - pode ser encadeado passando `err` como parametro de `next()`, neste caso o express encaminha a requisição para o próximo middleware de erro ao invez de seguir o fluxo normal da rota
```js
app.use((err, _req, res, _next) => {
  res.status(500).json({ message: `Algo deu errado! Mensagem: ${err.message}` });
  
// ou

app.use((err, _req, _res, next) => {
  console.error(err.stack);
  // passa o erro para o próximo middleware
  next(err);
});

app.use((err, _req, res, _next) => {
  res.status(500).json({ message: `Algo deu errado! Mensagem: ${err.message}` });
});
	
// ou
	
// middlewares/error.middleware.js
module.exports = (err, _req, res, _next) => {
  if (err.isJoi) { // erro do joi tem essa propriedade
    return res.status(422).json({
      error: { message: err.details[0].message },
    });
  }

  if (err.statusCode) {
    return res.status(err.statusCode).json({
      error: { message: err.message },
    });
  }

  console.error(err);

  return res.status(500).json({
    error: {
      message: `Internal server error: ${err.message}`,
    },
  });
};
	
// /app.js
app.use(middlewares.error)
```
  - middleware de rota
    - `const router = express.Router()`
    - o Router é um middleware que “agrupa” várias rotas em um mesmo lugar
    - os caminhos das rotas podem ser definidos relativos ao app.js (ex: `app.use('/teams', teamsRouter)` e em teams.js router.get('/')) ou como rotas absolutas no arquivo de rotas (ex: `app.use(teamsRouter)` e em teams.js router.get('/teams'))
    - no exemplo abaixo uma rota definida com router.get('/:id') na verdade se torna acessível por meio de /teams/:id
```js
// src/app.js

// ...
// importa arquivo com rotas
const teamsRouter = require('./routes/teamsRouter');
// ...
const app = express();
// ...
// monta o router na rota /teams
app.use('/teams', teamsRouter);
// ...
module.exports = app;

// src/routes/teamsRouter

// cria um router middleware
const router = express.Router();
// middlewares podem ser usados da mesma forma
router.use(<middleware>);
// o path é relativo à rota em que o router foi montado ou seja a rota desse GET é `/teams`
router.get('/', (req, res) => res.json(teams));  
//  ...
module.exports = router;
```
- arquivo barrel para middlewares de rota
  - arquivo que agrega os routes com objetivo de melhorar a organização do código
  - criar um index.js na pasta routes
  - importar todos os arquivos routers
  - exportar um router com todos unidos
  - importar e usar no app somente esse router
```js
// routes/index.js

const express = require('express');

const router = express.router();

const router1 = require('./router1')
const router2 = require('./router2')

router.use(router1);
router.use(router2);

module.exports = router;
```

### MYSQL2
- sintaxe prepared statements
    - arrow function que utiliza a função execute do objeto connection
    - `connection.execute()` método ASSINCRONO que recebe uma string (utiliando craze para permitir quebra de linha) como primeiro argumento, um array como segundo e retorna uma promisse
    - ? representa placeholders 
    - a ordem em que as dados são passadas no array deve ser igual a ordem que as chaves foram passadas na string
    - quando utilizada deve ser chamada com await
    - prepared statements previne ateques de sql injection
```js
// src/db/peopleDB.js

const connection = require('./connection');
  
const findById = async (id) => connection.execute('SELECT * FROM people WHERE id = ?', [id]);
  
const insert = async (person) => {
  const [{ insertId }] = await connection.execute(
    `INSERT INTO people 
        (first_name, last_name, email, phone) VALUES (?, ?, ?, ?)`,
      [person.firstName, person.lastName, person.email, person.phone],
    );
  const [[personWithId]] = await findById(insertId);
  return personWithId;
  }
const findAll = async () => await connection.execute('SELECT * FROM people');

module.exports = { insert, findAll, findById };
  
// src/routes/peopleRoutes.js

const peopleDB = require('../db/peopleDB');
// ...
router.post('/', (req, res) => {
  const person = req.body;
  try {
    const result = peopleDB.insert(person);
// ...
    }
});
  
router.get('/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const [[result]] = await peopleDB.findById(id);
// ...
    }
});
```
- retorno em array
  - deve-se sempre desestruturar o retorno
    - se o retorno for um objeto desestruturar em 2 arrays `const [[result]] = await peopleDB.findById(id);` ou em um array e um objeto para tirar uma propriedade do objeto `const [{ affectedRows }] = await connection.execute(...)`
    - se o retorno for um array desestruturar em um array `const [result] = await peopleDB.insert(person);`
	
### sequelize
- um ORM (object relational map) - permite um mapeamento estrutural entre as entidades do banco de dados e os objetos que as representam no código
- gera um código mais consiso, de mais fácil manutenção
- o sequelize utiliza os métodos de mapeamento `data mapper` (a classe que representa os registros NÃO conhece nada do banco. Ou seja existe uma interface que liga a estrutura do banco de dados com a aplicação sem que estes se comuniquem diretamente) ou o `active record` (a classe que representa os registros conhece os recursos necessários para realizar transaçoes no banco - a DB e a aplicação estão mais entrelasadas)
- migrations
	- representa as alterações em estruturas de tabelas do banco de dados. Basicamente um controle de versão
	- a função up constroi o novo recurso e a função down desfaz a up
	- o comando `npx sequelize migration:generate --name <nome>` cria o arquivo de migration, aonde o nome deve ser igual o do model porém no plural e minúsculo
	- o comando `npx sequelize db:migrate` executa as migrations
	- o comando `npx sequelize db:migrate:undo` desfaz a ultima migration
	- para reverter ATÉ uma migration específica `npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js`
	- funções de migration - createTable, addColumn, updateColumn, dropTable, removeColumn ...
```js
	'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      fullName: {
        type: Sequelize.STRING,
	field: 'full_name', // o nome do campo na DB
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
	default: Sequelize.NOW(),
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
	default: Sequelize.NOW(),
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Users');
  }
};
```
- model
	- na pasta models criar arquivos no formato `<nome>.model.js` aonde o nome deve ser no singular e com inicial maiúscula
	- usar a função define do sequelize
	- caracterizar os nomes das propriedades e os tipos de dados a serem usados
	- outra forma de se criar models é atravéz da cli `npx sequelize model:generate --name <nome> --attributes fullName:string,email:string` que gera um arquivo model e um arquivo migration.
	- no model propriedades padrão como created_at, updated_at e id podem ser omitidas porém é recomendado explicitar a declaração 
```js
// src/models/user.model.js

const UserModel = (sequelize, DataTypes) => {
const User = sequelize.define('User', {
	id: {
	  type: DataTypes.INTEGER,
	  primaryKey: true,
	  autoIncrement: true,
	},
	fullName: DataTypes.STRING,
	email: DataTypes.STRING,
	}, {
	sequelize,
	modelName: 'User',
	tableName: 'Users',
	underscored: true, // transformação automática dos camel case do model para kebab case nos campos com field declarado na migration
	timestamps: true, // permite desativar as propriedades default created_at e updated_at (devem ser omitidas da migration em caso de false)
	createdAt: 'foo', // permite renomear as propriedades default created e updated at
	updatedAt: 'bar',
	});
	return User;
};

module.exports = UserModel;
```
- seeders
	- um seeder é usado para, basicamente, alimentar o banco de dados com informações necessárias para o funcionamento mínimo da aplicação
	- tambem possui as funções up e down
	- comando para criar o arquivo `npx sequelize seed:generate --name <nome>`
	- comando para executar as seeds `npx sequelize db:seed:all` ou `npx sequelize db:seed --seed <caminho-do-arquivo>` para executar uma seed específica
	- comando para desfazer a ultima seed `npx sequelize-cli db:seed:undo` e para desfazer todas seeds `npx sequelize db:seed:undo:all`
	- comando para desfazer seed especifica `npx sequelize-cli db:seed:undo --seed name-of-seed-as-in-data`
```js
'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => queryInterface.bulkInsert('Users',
    [
      {
        fullName: 'Leonardo',
        email: 'leo@test.com',
        // usamos a função CURRENT_TIMESTAMP do SQL para salvar a data e hora atual nos campos `createdAt` e `updatedAt`
        createdAt: Sequelize.literal('CURRENT_TIMESTAMP'),
        updatedAt: Sequelize.literal('CURRENT_TIMESTAMP'),
      },
      {
        fullName: 'JEduardo',
        email: 'edu@test.com',
        createdAt: Sequelize.literal('CURRENT_TIMESTAMP'),
        updatedAt: Sequelize.literal('CURRENT_TIMESTAMP'),
      },
    ], {}),

  down: async (queryInterface) => queryInterface.bulkDelete('Users', null, {}),
};
```
- requisições
	- [docs](https://sequelize.org/docs/v6/core-concepts/model-querying-basics/)
	- Os models são os responsaveis por realizar operações na db. São chamados nos services, deve-se importar o arquivo index da pasta models e desestruturar pelo nome dos models
	- funções assíncronas nativas do sequelize como `<model-name>. findAll(), findByPk(id), findOne({ where: { id, email } }), findAll({ where: { id, email }, order: [ ['name', 'ASC'] ] }), create({ fullName, email }), update({ fullName, email }, { where: { id } }), destroy({ where: { id } })`
	- para ordenação usar a chave order do objeto de configuração. Recebe um array com outros arrays que representam a ordem de prioridade na ordenação. Na primeira posição a coluna a ser usada e na segunda a forma de ordenaçÃo (ASC, DESC)
	- o retorno das funções do sequelize possui metadados do mesmo. Para remove-los usar a propriedade dataValues do retorno
	- para remover proriedades usar a chave attributes, um objeto com a propriedade excludes que por sua vez é um array
	- a chave where permite filtar resultados
```js
// src/services/user.service.js

const { User } = require('../models/');

const getAll = async () => {
   const users = await User.findAll();
   return users;
};

const getById = async (id) => {
  const user = await User.findByPk(id);
  return user;
};

const getByIdAndEmail = async (id, email) => {
  const user = await User.findOne({ where: { id, email }, attributes: { exclude: [ 'email'] } });
  return user;
};
	
const getByAuthor = async (author) => {
  const books = await Book.findAll({ where: { author }, order: [ ['title', 'ASC'] ] });
  return books;
};

const createUser = async (fullName, email) => {
  const newUser = await User.create({ fullName, email });
  return newUser;
};

const updateUser = async (id, fullName, email) => {
  const [affectedRows] = await User.update({ fullName, email }, { where: { id } }); 
  return affectedRows;
};

const deleteUser = async (id) => {
  const user = await User.destroy({ where: { id } });
  return user;
};
	
const findAuthor = async () => {
	return Author.findAll({
	    include: {
	      model: Book, // inclui modelo com o qual tem associação
	      attributes: [],
	    },
	    attributes: [
	      ['name', 'author'], // renomeia atributo name como author
	      [sequelize.fn('COUNT', sequelize.col('books.id')), 'totalBooks'], // sequelize fn. Funções de agregação
	    ],
	    group: 'authors.name', // agrupa resultado pela coluna name
	    order: [['totalBooks', 'DESC']],
	    raw: true, // retorna somente os dados da entidade. Sem informações do banco
	  });
};

module.exports = {
  getAll,
  getById,
  getByIdAndEmail,
  createUser,
  updateUser,
  deleteUser,
};
```
- transações
	- Uma transação simboliza uma unidade de trabalho indivisível executada do banco de dados de forma independente de outras transações. Ou seja, um conjunto de operações aonde ou todo o trabalho é feito, ou nada é feito
	- Uma transação de banco de dados relacional, por definição, deve ser atômica (Se algo falhar, a transação inteira falha - indivisível), consistente (todas as regras do banco de dados devem ser respeitadas), isolada (uma transação não pode interferir em outra transação) e durável (uma vez finalizada, os dados devem ser armazenados de forma permanente), mais conhecida pela sigla ACID
	- Transações aumentam a confiabilidade da sua aplicação, já que respeitam o princípio da atomicidade, evitando popular o banco de dados de forma inconsistente
	- Unmanaged transactions
		- é preciso indicar manualmente a circunstância em que uma transação deve ser finalizada ou revertida, ou seja, executar o commit ou rollback
	- Managed transactions
		- o próprio Sequelize gerencia as transações e determina em tempo de execução, quando deve finalizar ou reverter uma transação
```js
// src/services/employee.service.js
const { Employee, sequelize } = require('../models');
// unmaneged transaction
const insert = async ({ firstName, lastName, age, city, street, number }) => {
  const t = await sequelize.transaction(); // cria a transação
  try {
    const employee = await Employee.create(
      { firstName, lastName, age },
      { transaction: t }, // vincula a operação a transação
    );

    await Address.create(
      { city, street, number, employeeId: employee.id },
      { transaction: t },
    );

// Se chegou até essa linha, quer dizer que nenhum erro ocorreu.
// Com isso, podemos finalizar a transação usando a função `commit`.
    await t.commit(); // finaliza a transação
    return employee;
  } catch (e) {
// Se entrou nesse bloco é porque alguma operação falhou.
// Nesse caso, o sequelize irá reverter as operações anteriores com a função rollback, não sendo necessário fazer manualmente
    await t.rollback(); // reverte a transação
    console.log(e);
    throw e; // Jogamos o erro para a controller tratar
  }
};
	
// managed transaction

const insert = async ({ firstName, lastName, age, city, street, number }) => {
  try {
    const result = await sequelize.transaction(async (t) => { // passa a transação como argumento
      const employee = await Employee.create({
        firstName, lastName, age
      }, { transaction: t }); // vincula a transação

      await Address.create({
        city, street, number, employeeId: employee.id
      }, { transaction: t });
      return employee;
    });
    return result;
// Se chegou até aqui é porque as operações foram concluídas com sucesso,
// não sendo necessário finalizar a transação manualmente.
// `result` terá o resultado da transação, no caso um empregado e o endereço cadastrado
  } catch (e) {
// Se entrou nesse bloco é porque alguma operação falhou.
// Nesse caso, o sequelize irá reverter as operações anteriores com a função rollback, não sendo necessário fazer manualmente
    console.log(e);
    throw e; // Jogamos o erro para a controller tratar
  }
// };
```
- relacionamentos
	- 1:1 / 1:N
		- migration
			- além das propriedades descritas acima adicionar as propriedades `references, onUpdate e onDelete` a chaves extrangeiras quando construindo a migration
				- `onUpdate e onDelete` podem ter os valores
					- `cascade`: ao se alterar ou excluir uma linha em uma tabela, a linha na tabela que usa a chave extrangeira também será alterada ou excluida
					- outro: `setNull`, ...
				- `references` é um objeto com as propriedades `model e key`. `model` referencia o nome da tabela da chave extrangeira e `key` referencia o nome da coluna da chave extrangeira
		- model
			- utilizar a função `associate()` do model tanto do lado que vai receber a chave extrangeira quanto do lado que vai fornecer os dados. Ela recebe como parametro `models`, disponibilizando para as funções de associação
			- funções de associação `hasOne bolongsTo hasMany bolongsToMany`
			- as funções de associação recebem dois argumentos, o primeiro é o model da chave extrangeira e o segundo um objeto com as chaves foreingKey e as. Aonde foreignKey representa o nome da chave extrangeira e as um apelido para aquela associação que será usado no retorno da query para os valores da tabela extrangeira
			- O lado que vai receber informação (tem uma coluna recebendo foreing key) usa as funções belongsTo ou bolongsToMany do model e o lado que vai fornecer informações (emprestar sua chave primaria) usa as funções hasOne ou hasMany 
			- deve-se pensar qual das tabelas irá emprestar sua primary key e qual vai ter uma coluna recebendo uma foreign key. Ou seja, a tabela que tem uma coluna recebendo foreign keys deve declarar de forma explicita essa coluna e sua referência no model/migration além de usar as funções de associação belongs com a chave foreignKey referenciando a sua própria chave extrangeira. Já a tabela que empresta sua primary key não declara nenhuma coluna extra no seu model/migration e somente usa as funções de associação do grupo has com a chave foreignKey referenciando a chave extrangeira do model sendo relacionado
		- requisições
			- [docs](https://sequelize.org/docs/v6/core-concepts/model-querying-finders/)	
			- A grande diferença quando vamos fazer uma requisição que necessite da utilização de uma association com o Sequelize, é o campo include. Se incluso o sequelize utilizará o eager loading para fazer a requisição, retornando os dados já unidos (join feita). Se omitido o sequelize fará uma requisição normal, trazendo somente a chave extrangeira.
			- o campo `include` é um objeto com as propriedades `model` e `as` ou um array com objetos que tenha as propriedades `model` e `as`, aonde model é o model da chave extrangeira e as deve ser igual a que declaramos no momento da criação da associação no respectivo model.
```js
// src/migrations/[timestamp]-create-Employees.js

module.exports = {
  up: async (queryInterface, Sequelize) => {
    return queryInterface.createTable('Employees', {
     { ... },
      age: {
        allowNull: false,
        type: Sequelize.INTEGER,
      },
      addressId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        // Configuram o que deve acontecer ao atualizar ou excluir um funcionário
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
        field: 'address_id', // o nome do campo na DB
        // Informa que o campo é uma Foreign Key (Chave estrangeira)
        references: {
          // Informa a tabela da referência da associação
          model: 'Address',
          // Informa a coluna da tabela da referência da associação
          key: 'id',
        },
      },
    });
  },

  down: async (queryInterface, _Sequelize) => {
    return queryInterface.dropTable('Employees');
  },
};
	
// src/migrations/[timestamp]-create-Addresses.js

module.exports = {
  up: async (queryInterface, Sequelize) => {
    return queryInterface.createTable('Addresses', {
     id: {
	allowNull: false,
	autoIncrement: true,
	primaryKey: true,
	type: Sequelize.INTEGER,
      },
     city: {
        allowNull: false,
        type: Sequelize.STRING,
      },
      street: {
        allowNull: false,
        type: Sequelize.STRING,
      },
      number: {
        allowNull: false,
        type: Sequelize.INTEGER,
      },
    });
  },

  down: async (queryInterface, _Sequelize) => {
    return queryInterface.dropTable('Addresses');
  },
};
	
// src/models/employee.model.js

module.exports = (sequelize, DataTypes) => {
	const Employee = sequelize.define('Employee', {
			id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
			firstName: DataTypes.STRING,
			lastName: DataTypes.STRING,
			age: DataTypes.INTEGER,
			addressId: {
				type: Sequelize.INTEGER,
				field: 'address_id',
				foreignKey: true,
			},
		},
		{
			timestamps: false,
			tableName: 'employees',
			underscored: true,
	});

	Employee.associate = (models) => {
		Employee.belongsTo(models.Address,
			{ foreignKey: 'addressId', as: 'addresses' });
		};

	return Employee;
};
	
// src/models/address.model.js

module.exports = (sequelize, DataTypes) => {
  const Address = sequelize.define('Address', {
    id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
    city: DataTypes.STRING,
    street: DataTypes.STRING,
    number: DataTypes.INTEGER,
  },
  {
    timestamps: false,
    tableName: 'addresses',
    underscored: true,
  });

  Address.associate = (models) => {
    Address.hasOne(models.Employee,
      { foreignKey: 'addressId', as: 'employees' });
  };

  return Address;
};
	
// src/services/employee.service.js

const { Address, Employee } = require('../models/');

const getAll = async () => {
  const users = await Employee.findAll({
    include: [{ model: Address, as: 'addresses' }, { model: Foo, as: 'bar' }],
  });

  return users;
};

module.exports = { getAll };
```
-
	- N:N
		-  pode ser visto também como dois relacionamentos um para muitos (1:N) ligados por uma tabela intermediária, chamada de tabela de junção
		- A tabela de junção possui dois campos compondo uma chave primária composta
		- Para se definir o relacionamento atarvés de uma tabela intermediaria o model da mesma deve:
		- migration
			- a tabela de ligação deve ser construida com as duas chaves como primayKey e foreign key (propriedade references) ao mesmo tempo
			- as outras tabelas devem ser contruidas de forma normal, sem foreignKey
		- model
			- as tabelas que participam do relacionamento mas não são a de ligação são declaradas sem colunas de foreignKey e sem funções de associação
			- jã a tabela de ligação deve ser declarada sem chaves (o sequelize interpreta que as associações são as chaves) e com funções de associação que referenciam as duas outras tabelas do relacionamento
				- ambas as associações são do tipo belongsToMany e devem ter em seu objeto de configuração as propriedades foreignKey (referencia a chave do próprio modelo de ligação que representa a foreignKey do modelo no qual a função belongsToMany é chamada), otherKey (a outra chave, referencia a chave do próprio modelo de ligação que representa a foreignKey do modelo do primeiro argumento da função belongsToMany), as (apelido da associação usado no retorno da query) e through (representa o modelo que faz a ligação entre as duas, ou seja, o modelo da tabela de associação)
		- requisições
			- requisições N:N são feitas nos models que representam as entidades, e não no model da tabela intermediaria
			- usar o `through: { attributes: [] } }` para que o retorno da query não traga os valores da tabela intermediária (por exemplo uma query que busca livros de um usuário pelo seu id sem o through retornaria em cada livro o id do usuário, o que é reduntante. Com o through essa informação não e retonada)
```js
// src/models/User.js
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define('User', {
    id: { type: DataTypes.INTEGER, primaryKey: true },
    firstName: DataTypes.STRING,
    lastName: DataTypes.STRING,
    age: DataTypes.INTEGER,
  },
    {
      timestamps: false,
      underscored: true,
    });

  return User;
};
	
// src/models/Book.js
module.exports = (sequelize, DataTypes) => {
  const Book = sequelize.define('Book', {
    id: { type: DataTypes.INTEGER, primaryKey: true },
    name: DataTypes.STRING,
    releaseYear: DataTypes.INTEGER,
    totalPages: DataTypes.INTEGER,
  },
    {
      timestamps: false,
      underscored: true,
    });

  return Book;
};
	
// src/migrations/20221206132302-create-users-books - tabela intermediária
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users_books', { // define as duas chaves como foreign keys (propriedade references) e primary keys(prop primaryKey)
      userId: {
        type: Sequelize.INTEGER,
        field: 'user_id',
        references: {
          model: 'users',
          key: 'id',
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
        primaryKey: true,
      },
      bookId: {
        type: Sequelize.INTEGER,
        field: 'book_id',
        references: {
          model: 'books',
          key: 'id',
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
        primaryKey: true, // AS DUAS COLUNAS TEM O PRIMARY KEY EXPLICITO (PRIMARY KEY COMPOSTA)
      },
    });
  },

  down: async (queryInterface, _Sequelize) => {
    await queryInterface.dropTable('users_books');
  },
};
	
// src/models/UserBook.js - tabela intermediária
module.exports = (sequelize, _DataTypes) => {
  const UserBook = sequelize.define('UserBook',
    {}, // não é preciso definir as chaves nem as propriedades foreign e primary key, o sequelize já define sozinho
    {
      timestamps: false,
      underscored: true, 
      tableName: 'users_books'
    },
  );
  UserBook.associate = (models) => {
    models.Book.belongsToMany(models.User, { // associar os dois models que usam a tabela de ligação (no caso Book e User) em relação 1:N
      as: 'users', //alias
      through: UserBook, // o model da tabela de ligação
      foreignKey: 'bookId', // se refere ao model em que belongsToMany é chamado
      otherKey: 'userId', // e refere ao model com o qual estamos criando a associação
    });
    models.User.belongsToMany(models.Book, {
      as: 'books',
      through: UserBook,
      foreignKey: 'userId',
      otherKey: 'bookId',
    });
  };
  return UserBook;
};
	
// src/services/userBook.service.js
const { User, Book } = require('../models');

const getUsersBooksById = (id) => User.findOne({
  where: { id },
  include: [{ model: Book, as: 'books', through: { attributes: [] } }], // a propriedade through com attributes vazio é necessaria para que não se retorne dentro da chave books todos os seus users
});

module.exports = {
  getUsersBooksById,
};

```
### arquitetura MSC
#### model
  - É a camada mais próxima da base de dados
  - responsavel por realizar operaçoes com a DB
	- quando utilizando sequelize, seus models são a camada model da arquitetura 
	- mysql2:
		- em `src` criar pasta `models`
			- um arquivo *.model.js para cada entidade (tabela) da DB
			- um arquivo index.js que importa todos os models e a connection e os exporta juntos (barrel)
		- em `tests/unit` criar a psta `models`
			- para fazer testes unitarios na camada model é preciso mockar a connection
				- mockar retornos das chamadas ao connection com stub do sinon
			- criar a pasta mocks com arquivos de extensão `*.model.mocks.js`
			- criar os arquivos de teste com extensão *.model.test.js
		- padrões comuns
			- para inserts e updates a operação na DB retorna as chaves `insertId` e `affectedRows` respectivamente no formato [{}]. Já select retorna um array com os resultados no formato [[ ]]
```js
// create
const insert = async (travel) => {
  const columns = Object.keys(snakeize(travel))
    .map((key) => `${key}`)
    .join(', ');
  const placeholders = Object.keys(travel)
    .map((_key) => '?')
    .join(', ');
  const [{ insertId }] = await connection.execute(
    `INSERT INTO travels (${columns}) VALUE (${placeholders})`,
    [...Object.values(travel)],
  );
  return insertId;
};
  
// read
const findTravelById = async (travelId) => {
  const [[result]] = await connection.execute(
    'SELECT * FROM travels WHERE id = ?',
    [travelId],
  );
  return camelize(result);
};
  
// update
const update = (data) => {
  const [{affectedRows}] = await connection.execute(
    'UPDATE travels SET driver_id = ? WHERE id = ?',
    [driverId, travelId],
  );
};
  
// delete
```

#### service
  - É a camada intermediária, entre base de dados e requisições
  - responsavel por realizar validações de regras de negócio
  - desacoplamento busca reduzir custo de manutenção (tempo)
  - em `src` criar pasta `services`
    - um arquivo *.service.js para cada entidade
    - um arquivo index.js que importa todos os services e os exporta juntos (barrel)
    - criar pasta validations
      - criar arquivo schemas.js aonde se declara as validações (joi)
      - criar arquivo validationsInputValues.js aonde se inicializa as validações
  - em `tests/unit` criar a psta `services`
    - para fazer testes unitarios na camada services é preciso mockar a camada model
      - mockar retornos das funções do model (ao invés do connection) com stub do sinon 
    - criar a pasta mocks com arquivos de extensão `*.service.mocks.js` 
    - criar os arquivos de teste com extensão *.service.test.js
```js
// /*.service.js
	
// com sequelize
	const { User } = require('../models/');
	const updateUser = async (id, fullName, email) => {
		const [updatedUser] = await User.update(
			{ fullName, email },
			{ where: { id } },
		);
		return updatedUser;
	};
	
// sem sequelize
	const updateProductById = async (id, name) => {
  const { type, message } = validateNewProduct(name);
  if (type) return { type, message };
  const affectedRows = await productsModel.updateProductById(id, name);
  if (affectedRows === 0) return { type: NOT_FOUND, message: 'Product not found' };
  const product = await productsModel.getProductsById(id);
  return { type: null, message: product };
	};
	
// /validations/schema.js
  
	const Joi = require('joi');

	const productSchema = Joi.object({
		name: Joi.string().min(5).required(),
	});

	const saleSchema = Joi.array().items({
		productId: Joi.number().integer(),
		quantity: Joi.number().integer().min(1).messages({
			'number.min': '"quantity" must be greater than or equal to 1',
		}),
	});
	
// /validations/validationsInputValues.js
	
	const validateNewProduct = (name) => {
  const { error } = productSchema.validate({ name });
  if (error) return { type: INVALID_VALUE, message: error.message };
  return { type: null, message: '' };
	};
  
// /tests/unit/services/*.service.test.js
	
	describe('testes unitários - rota /products - services', function () {
		describe('endpoint PUT /products/:id - updateProductById()', function () {
			afterEach(sinon.restore);

			it('em caso de sucesso', async function () {
				const product = { id: 1, name: 'abacaxi' };
				sinon.stub(productsModel, 'updateProductById').resolves(1)
				sinon.stub(productsModel, 'getProductsById').resolves(product);
				sinon.stub(validateNewProduct, 'validateNewProduct').resolves({ type: null, message: '' });

				const resolves = await productsService.updateProductById(1, 'abacaxi');

				expect(resolves).to.be.deep.equal({ type: null, message: product });
			});

			it('em caso de falha - nome inválido', async function () {
				sinon.stub(validateNewProduct, 'validateNewProduct').resolves({ type: null, message: "\"name\" length must be at least 5 characters long" });

				const resolves = await productsService.updateProductById(1, 'a');

				expect(resolves).to.be.deep.equal({ type: 'INVALID_VALUE', message: "\"name\" length must be at least 5 characters long" });
			});
		});
	});
```
  
#### controler
  - É a camada responsavel por controlar o recebimento de requisições, fazer validações de dados (que não envolvem regras de negócio) e chamar os serviços necessários para responder a requisição ou retornar erro
  - Existem dois tipos de erros:
    - erros originados pelo cliente → occorrem quando as validações falham. Devem ser respondidos com informações sobre o erro.
    - erros originados na api → devem ser respondidos com status 500 e mensagem genérica para não expor informações sobre a construção da api.
  - em src criar a pasta utils
    - criar o arquivo errorMap.js em que se mapeia a propriedade type dos objetos de erro a um status específico
  - em scr criar a pasta routers
    - criar um arquivo de middleware de rota no formato *.router.js para cada grupo de rotas similar
    - criar um arquivo barrel index.js aonde se importa todos os middlewares de rota e os exporta em um router unico (ou exportar em um objeto unico)
    - em app.js chamar a função use() para cada router criado, passando a rota base
  - em src criar a pasta controllers
    - Aqui ficaram as funções callback do segundo parametro das chamadas de rotas
    - tal organização permite fazer testes unitários na camada controller sem a necessidade de inicializar o express.
    - criar um arquivo *.controller.js para cada middleware de rota
        - as funções desses arquivos se comportam da mesma forma que quando estavam na definição da rota. Podem receber os paramentros (err, req, res, next) e retornam uma resposta para requisição
    - criar arquivo barrel index.js e importar todos controllers, exportando em um objeto único
  - em src criar a pasta middlewares
    - middlewares são usados nas chamadas de rotas para validação da existencia das informações necessárias na requisição para execução do serviço a ser chamado
    - um arquivo para cada função de validação
    - em caso de falha respo nder a requisição com o erro e em caso de sucesso chamar next()
  
  - em `tests/unit` criar as pastas controllers e middlewares
    - na pasta controllers criar a pasta mocks e os arquivos *.controller.test.js
      - na pasta mocks criar os arquivos *.controller.mock.js
      - para testar uma função controller deve-se criar os objetos req e res a serem passados como parametro e mockar a chamada ao serviço realizada nela. Além de os retornos das funções de resposta res.status e res.message
      - as verificações são feitas com base nas funções de resposta res.status e res.message, chcando a estrutura que foi passada em suas execuções
      - uso do sinonChai permite fazer asserções mais diretas nos parâmetros de chamada do res.status e res.json (to.have.been.calledWith() ou to.have.been.called)
      - middlewares são testados separadamente seguindo a mesma lógica e acrescentando um mock sem valor de retorno para next().
  - em test/integration criar a pasta controllers
    - criar a pasta mocks e arquivos *.controller.test.js
    - similar aos testes unitários porem o que é mockado são as chamadas a DB (connection.execute() da camada model)
```js
// src/utils/errorMap.js
const errorMap = {
  TRAVEL_NOT_FOUND: 404,
  DRIVER_NOT_FOUND: 404,
  INVALID_VALUE: 422,
  TRAVEL_CONFLICT: 409,
};

const mapError = (type) => errorMap[type] || 500;

module.exports = {
  errorMap,
  mapError,
};

// src/routers/bar.router.js
const express = require('express');
const { bar } = require('../controllers/foo.controller');

const router = express.Router();

router.get('/foo', bar);
{ ... }
module.exports = router;
  
// src/controllers/foo.controller.js
const { fooService } = require('../services');
const errorMap = require('../utils/errorMap');

const bar = async (_req, res) => {
  const { type, message } = await fooService.getAll();
  if (type) return res.status(errorMap.mapError(type)).json(message);
  res.status(200).json(message);
};

module.exports = {
  bar,
};
  
// tests/unit/controllers/driver.controller.test.js
const chai = require('chai');
const sinon = require('sinon');
const sinonChai = require('sinon-chai');

const { expect } = chai;
chai.use(sinonChai);

const { driverService } = require('../../../src/services');
const driverController = require('../../../src/controllers/driver.controller');

describe('Teste de unidade do driverController', function () {
  it('Buscando as viagens em aberto quando não tem nenhuma viagem cadastrada', async function () {
    const res = {};
    const req = {};

    res.status = sinon.stub().returns(res);
    
    res.json = sinon.stub().returns();

    sinon
      .stub(driverService, 'getWaitingDriverTravels')
      .resolves({ type: null, message: [] });

    await driverController.openTravel(req, res);

    expect(res.status).to.have.been.calledOnceWith(200);

    expect(res.json).to.have.been.calledWith([]);
		expect(res.json).to.have.been.called;

  });
});
  
// tests/integration/controllers/passenger.controller.test.js
const chai = require('chai');
const sinon = require('sinon');
const chaiHttp = require('chai-http');
const sinonChai = require('sinon-chai');

const { expect } = chai;
chai.use(sinonChai);
chai.use(chaiHttp);

const app = require('../../../src/app');

const connection = require('../../../src/models/connection');

const {
  happyTravelDB,
  happyPassengerDB,
  happyTravelResponse,
} = require('./mocks/passenger.controller.mock');

describe('Teste de integração de passengers', function () {
  it('Criação de uma nova viagem com sucesso', async function () {
    sinon
      .stub(connection, 'execute')
      .onFirstCall()
      .resolves([[happyPassengerDB]])
      .onSecondCall()
      .resolves([{ insertId: 42 }])
      .onThirdCall()
      .resolves([[happyTravelDB]]);

    const response = await chai
      .request(app)
      .post('/passengers/1/request/travel')
      .send({
        startingAddress: 'Rua AAAA',
        endingAddress: 'Rua BBB',
      });

    expect(response.status).to.be.equal(201);

    expect(response.body).to.be.deep.equal(happyTravelResponse);
  });
  afterEach(sinon.restore);
});
```
  
### testes com mocha
  - recomenda-se não usar arrow function, e sim declarar atraves da palavra function
  - funções `beforeEach() e afterEach()` para setup e teardown
  - AAA (triple A) - técnica para escrita de testes - arrange / act / assert
	- chai auxilia nas chamadas para testes de integração e nas acerções
```js
describe('Foo', function () {
  beforeEach();
  afterEach(sinon.restore);
  it('Foo', async function () {
    const output = <output-esperado>;
    const response = await chai
      .request(app)
      .put(<route>)
      .send(<dados-do-body>);
    expect(response.status).to.be.equals(200);
    expect(response.body).to.have.property(<property>);
    expect(response.body).to.deep.equal(output);
    expect(response.affectedRows).to.be.equal(1);
    expect(response.insertId).to.be.equal(1);
    expect(response).to.be.a('array');
  });
})
```
#### mocks com Sinon
- Stubs são objetos que podemos utilizar para simular interações com dependências externas ao que estamos testando de fato
- o escopo da chamada ao sinon deve ser o mais local o possível (dentro do it) pois se mais de um teste usa aquela função haverá conflito entre os retornos
- `sinon.stub(<modulo>, '<função>').resolves(<valor-retornado>)`
- `sinon.restore()` para teardown do stub (aftereach)
- atenção para a formatação do retorno mockado. Deve ser identico ao retorno real (atenção especial em stubs do connection.execute() com os retornos em array)
```js
  it('Testando o cadastro de uma pessoa ', async function () {
    sinon.stub(connection, 'execute').onFirstCall().resolves([{ insertId: 42 }]);

    const response = await chai
      .request(app)
      .post('/people')
      .send(
        {
          firstName: 'Luke',
          lastName: 'Skywalker',
          email: 'luke.skywalker@trybe.com',
          phone: '851 678 4453',
        },
      );

    expect(response.status).to.equal(201);
    expect(response.body).to.
      deep.equal({ message: 'Pessoa cadastrada com sucesso com o id 42' });
  });
```

#### coverage com nyc
  - `"test:coverage": "nyc --all --include src/models --include src/services --include src/controllers mocha tests/unit/**/*.js --exit"` é o script do package.json para executart teste de cobertura

#### testes com sequelize-test-helpers
  - [docs](https://www.npmjs.com/package/sequelize-test-helpers)
	- quando o objetivo é fazer um teste unitario do service (mokar o model) ou fazer teste do model na mão, deve-se importar o model desestruturando a referencia ao models/index.js. Quando se deseja utilizar o model com sequelize-test-helpers deve-se importar o arquivo do model em sí
	- para usar o banco de dados em ambiente de testes é necessário inicializar o mesmo para o ambiente de teste ```NODE_ENV=test npx sequelize-cli db:create
NODE_ENV=test npx sequelize-cli db:migrate
NODE_ENV=test npx sequelize-cli db:seed:all```
```js
// teste na mão
const { expect } = require('chai');
const { Book } = require('../../models');

describe('O model de Book', function () {
  it('possui a propriedade "title"', function () {
    const book = new Book();
    expect(book).to.have.property('title');
  });

  it('possui a propriedade "author"', function () {
    const book = new Book();
    expect(book).to.have.property('author');
  });

  it('possui a propriedade "pageQuantity"', function () {
    const book = new Book();
    expect(book).to.have.property('pageQuantity');
  });

  it('possui a propriedade "publisher"', function () {
    const book = new Book();
    expect(book).to.have.property('publisher');
  });
});

// teste com sequelize-test-helpers
const {
  sequelize,
  dataTypes,
  checkModelName,
  checkPropertyExists,
} = require('sequelize-test-helpers');

const UserModel = require('../../../src/models/user.model');

describe('O model de User', () => {
  const User = UserModel(sequelize, dataTypes);
  const user = new User();

  describe('possui o nome "User"', () => {
    checkModelName(User)('User');
  });

  describe('possui as propriedades "fullName" e "email"', () => {
    ['fullName', 'email'].forEach(checkPropertyExists(user));
  });
});
```
### leitura e escrita de arquivos com fs
  -  terceiro parâmentro flag opcional
      - `w` para escrita
      - `wx` lança errro caso o arquivo exista
  - leitura
```js
const fs = require('fs').promises;
const path = require('path');

async function readData() {
  try {
    const json = await fs.readFile(path.resolve(__dirname, './meu-arquivo.txt'), 'utf-8');
    const data = JSON.parse(data);
    return data;
    console.log('arquivo lido com sucesso');
  } catch (err) {
    console.error(`Erro ao ler o arquivo: ${err.message}`);
  }
}
```
  - escrita
```js
 const fs = require('fs').promises;
 const path = require('path');
 
 async function updateFile(id, newData) {
  try {
    const oldFile = await readData();
    // operação com arquivo antigo
    // const index = oldFile.<prop>.findIndex(...);
    // oldFile.<prop>[index] = { id, ...newData };
    const newFile = JSON.stringify(oldFile);
    await fs.writeFile(path.resolve(__dirname, './meu-arquivo.txt'), newFile);
    console.log('Arquivo escrito com sucesso!');
  } catch (err) {
    console.error(`Erro ao escrever o arquivo: ${err.message}`);
  }
}
 ```
### JWT e bcrypt
- O JWT (JSON Web Token) é um token gerado a partir de dados “pessoais” que pode ser trafegado pela internet ao fazer requisições para APIs e afinscom o objetivo de autenticar e verificar permissões de usuários
- bcrypt adiciona uma camada de segurança as senhas salvas no banco de dados através de um hash. Quando criado o usuário (no service user) faz a incriptação da senha e quando autenticado usa a função de comparação nativa do bcrypt como demostrado abaixo
- Autenticaço !== autorização
	- Autenticação é utilizada para verificar sua identidade, realizada por meio de informações confidenciais como email e senha.
	- Autorização verifica as permissões de uma pessoa para acessar ou executar determinadas operações.
- a signature gerada pelo JWT consite em uma string criptografada a partir de tres partes. Header que informa os metadados da operação, payload que contém os dados da requisição (body por exemplo) e secret, que é a chave do JWT
	- Header contém as propriedades `alg, typ`, alg sendo o tipo do algoritimo de encriptação (por exemplo 'HS256') e typ o tipo do token, no caso 'JWT'. Além de outras propriedades como `expiresIn`
	- Payload contém os dados da entidade, como nome, id e permissões
	- Signature consiste no header e no payload codificados em base64, usando o algoritmo definido no header
	- gerando a estrutura `(Header em base64).(Payload em base64).(Signature em base64)`
- uma vez o usuário tentando fazer login e seus dados sendo validados é gerado o token que será retornado ao usuário
	- a função sign do JWT recebe três argumentos nesta ordem, o objeto payload (os dados da entidade), o secret para criptografar, e header (objeto de configuração)
```js
require('dotenv/config');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const { UserService } = require('../services');

/* Sua chave secreta. É com ela que os dados do seu usuário serão encriptados.
   Em projetos reais, armazene-a numa variável de ambiente e tenha cuidado com ela, pois só quem tem acesso
   a ela poderá criar ou alterar tokens JWT. */
const secret = process.env.JWT_SECRET || 'seusecretdetoken';

const isBodyValid = (username, password) => username && password;

module.exports = async (req, res) => {
  try {
    const { username, password } = req.body;

    if (!isBodyValid(username, password)) {
      return res.status(401).json({ message: 'É necessário usuário e senha para fazer login' });
    }
  
    const user = await UserService.getByUsername(username);
  
    if (!user || user.password !== password) {
      return res.status(401).json({ message: 'Usuário não existe ou senha inválida' }); 
    }
    // ou se estiver usando bcrypt
    // if (!user || !bcrypt.compareSync(password, user.password)) {
    //   return res.status(401).json({ message: 'Usuário não existe ou senha inválida' }); 
    // }

    /* Criamos uma config básica para o nosso JWT, onde:
    expiresIn -> significa o tempo pelo qual esse token será válido;
    algorithm -> algoritmo que você usará para assinar sua mensagem
                (lembra que falamos do HMAC-SHA256 lá no começo?). */

    /* A propriedade expiresIn aceita o tempo de forma bem descritiva. Por exemplo: '7d' = 7 dias. '8h' = 8 horas. */
    const jwtConfig = {
      expiresIn: '7d',
      algorithm: 'HS256',
    };

    /* Aqui é quando assinamos de fato nossa mensagem com a nossa "chave secreta".
      Mensagem essa que contém dados do seu usuário e/ou demais dados que você
      quiser colocar dentro de "data".
      O resultado dessa função será equivalente a algo como: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjp7ImlkIjozLCJ1c2VybmFtZSI6Iml0YWxzc29kaiIsInBhc3N3b3JkIjoic2VuaGExMjMifSwiaWF0IjoxNjM4OTc1MTMyLCJleHAiOjE2Mzk1Nzk5MzJ9.hnpmu2p61Il8wdQfmUiJ7wiWXgw8UuioOU_D2RnB9kY */
    const token = jwt.sign({ data: user.dataValues }, secret, jwtConfig); // dataValues e uma propriedade do sequelize que retorna os dados sem metadados do sequelize

    /* Por fim, nós devolvemos essa informação ao usuário. */
    res.status(200).json({ token });
  } catch (err) {
    return res.status(500).json({ message: 'Erro interno', error: err.message });
  }
};
```
- para autorizar o acesso do usário deste ponto para frente o token deve ser enviado pelo mesmo. Para tal criamos uma função que será usada como middleware para as nossas requisições, validando todas as rotas em que nós solicitarmos autenticação
```js
// src/auth/validateJWT.js
const jwt = require('jsonwebtoken');

require('dotenv/config');
const { UserService } = require('../services');

/* Mesma chave privada que usamos para criptografar o token.
   Agora, vamos usá-la para descriptografá-lo.
   Numa aplicação real, essa chave jamais ficaria hardcoded no código assim,
   e muitos menos de forma duplicada, mas aqui só estamos interessados em
   ilustrar seu uso ;) */
const secret = process.env.JWT_SECRET || 'seusecretdetoken';

module.exports = async (req, res, next) => {
  /* Aquele token gerado anteriormente virá na requisição através do
     header Authorization em todas as rotas que queremos que
     sejam autenticadas. */
  const token = req.header('Authorization');

  /* Caso o token não seja informado, simplesmente retornamos
     o código de status 401 - não autorizado. */
  if (!token) {
    return res.status(401).json({ error: 'Token não encontrado' });
  }

  try {
    /* Através o método verify, podemos validar e decodificar o nosso JWT. */
    const decoded = jwt.verify(token, secret);
    /*
      A variável decoded será um objeto equivalente ao seguinte:
      {
        data: {
          userId: 1
        },
        iat: 1656616422,
        exp: 1657221222
      }
    */

    /* Caso o token esteja expirado, a própria biblioteca irá retornar um erro,
       por isso não é necessário fazer validação do tempo.
       Caso esteja tudo certo, nós então usamos o serviço de usuário para obter seus dados atualizados */

    const user = await UserService.getByUserId(decoded.data.userId);

    /* Não existe um usuário na nossa base com o id informado no token. */
    if (!user) {
      return res.status(401).json({ message: 'Erro ao procurar usuário do token.' });
    }

    /* O usuário existe! Colocamos ele em um campo no objeto req.
       Dessa forma, o usuário estará disponível para outros middlewares que
       executem em sequência */
    req.user = user;

    /* Por fim, chamamos o próximo middleware que, no nosso caso,
       é a própria callback da rota. */
    next();
  } catch (err) {
    return res.status(401).json({ message: err.message });
  }
};
	
// /app.js
const express = require('express');
const bodyParser = require('body-parser');
const routes = require('./routes');

/* Aqui, importamos nossa função que valida se o usuário está ou não autenticado */
const validateJWT = require('./auth/validateJWT');

const app = express();

app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

const apiRoutes = express.Router();

apiRoutes.get('/api/posts', validateJWT, routes.getPosts);
apiRoutes.post('/api/users', routes.createUsers);
apiRoutes.get('/api/users', routes.getUsers);
apiRoutes.post('/api/login', routes.login);

app.use(apiRoutes);

module.exports = app;
	
// services/user.service.js
const { User } = require('../models');

const create = async ({ username, password }) => {
	const salt = bcrypt.genSaltSync(10)
	const hashedPassword = bcrypt.hashSync(password, salt);
	const user = await User.create({ username, password: hashedPassword })
	const { password: _password, ...userWithoutPassword } = user;
	return userWithoutPassword;
}
```
- Uma requisição usando JWT tem o formato abaixo
```bash
 GET /foo/bar HTTP/1.1
 Host: www.exemplo.com
 Authorization: Bearer (Header em base64).(Payload em base64).(Signature em base64)
```
