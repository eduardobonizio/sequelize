# sequelize

Resumo para esse projeto funcionar:
  npm install
  npx sequelize db:migrate
  npx sequelize db:seed:all
  Configurar o .env (Para os dados do arquivo ./config/config.js)
  
Instalando o Sequelize:
  npm install sequelize
  npm install --save-dev sequelize-cli
  npm install mysql2

Instalando o express
  npm install express
  npm install nodemon
  npm install body-parser

Boas práticas:
  npm install dotenv

Resumo de como adicionar conteúdo ao banco de dados:
  Criar migration:
    npx sequelize migration:generate --name migration-name
  Editar o conteúdo de UP e DOWN da migration
    Aqui podemos definir os tipos, se poderá ser nulo ou não, primary key, foreign key, chave composta.
  Depois de definir as migrations:
    npx sequelize db:migrate

Testes:
  npm i mocha chai sinon chai-http -D

.sequelizerc
  Configuração de onde os arquivos do sequelize devem ficar.

Iniciando o Sequelize-cli:
  npx sequelize-cli init
  Pastas criadas pelo comando:
    config : contém um arquivo de configuração, que "fala" para o CLI como conectar-se com o nosso banco de dados;
    models : contém todos os modelos da nossa aplicação;
    migrations : contém todos os arquivos de migração da nossa aplicação;
    seeders : contém todos os arquivos de "seeds".
  Configurar conforme necessidade ./config/config.json:
    ex.: 
      {
        "development": {
        "username": "root",
        "password": "",
        "database": "orm_example",
        "host": "127.0.0.1",
        "dialect": "mysql"
        },

        // No resto do arquivo vamos encontrar as convenções para conectar o Sequelize em outros ambientes
      }
    Legenda:
      Usuário de acesso ao banco de dados;
      Senha de acesso ao banco de dados;
      Nome do banco de dados no qual queremos conectar;
      Host que estamos conectando - por ser local, utilizamos o 127.0.0.1 ;
      Dialect é, nada mais nada menos, qual banco estamos utilizando. Dito isso, passamos "mysql".
    Obs.: Isso é só um exemplo, o correto seria utilizar varáveis de ambiente.

Criando o banco de dados via CLI:
  //O nome será aquele definido no config.json
  npx sequelize db:create

  //Para checar se o banco foi criado
  mysql -u root -p
  show databases;

O Sequelize suporta os bancos MySQL , MariaDB , PostgreSQL , SQLite e Microsoft SQL Server .


Pasta Model:
  O sequelize cria o index.js, não apagar esse arquivo pois ele é utilizado para conexão com o banco de dados.
  Um model é uma abstração que representa uma linha na tabela em seu banco de dados e diz ao Sequelize várias coisas sobre essa entidade, como o nome da tabela no banco de dados e quais colunas ela possui (e seus tipos de dados).
  O model pode ser definido de duas formas:
    Chamando pela função sequelize.define(modelName, attributes, options)
    Estendendo Model como uma classe e chamando init(attributes, options)
  Template para criação:
    //Além de gerar o model, ele também gera uma migration que irá criar a tabela no banco de dados.
    //--name se refere ao nome da tabela
    //--attributes se refere ao nome das colunas e os tipos de dados que ela contém. Não é preciso definir todas as colunas neste      //comando, é possível adicioná-las direto no arquivo model.js gerado e na migration equivalente a este model.
    npx sequelize model:generate --name NomeDoModel --attributes nomeDoAtributo:string

    ex.:
      npx sequelize model:generate --name User --attributes fullName:string
      //Depois de rodar este comando, perceba que foi criado um arquivo user.js na pasta model, e na pasta migration foi criado o 
      //arquivo 20210310124202-create-user.js (os números, no início do nome do arquivo, significam a data e a hora de criação dele, 
      //seguindo o formato yyyy-MM-dd:hh:mm:ss ).

  Como não iremos trabalhar com classes, mas sim com a função sequelize.define(), temos que substituir o models/user.js por:
    const User = (sequelize, DataTypes) => {
      const User = sequelize.define("User", {
        fullName: DataTypes.STRING,
        email: DataTypes.STRING,
      });

      return User;
    };
    module.exports = User;
  //Observe que adicionamos o campo e-mail do tipo string, por aqui podemos adicionar quantas colunas forem necessárias.

Migration:
  Uma migration é uma versão do banco de dados, a cada atualização que ocorre no banco - como adição de coluna - ele gera uma nova migration. As migrations tem as funções UP e DOWN que servem para subir a nova migration (executar as mudanças no banco de dados) ou restaurar um antiga (Up e Down respectivamente).

  Como adicionamos o campo "email" no passo anterior, temos que mudar a migration para adicionar esse campo.
  Temos que abrir o arquivo correspondente na pasta ./migrations/ e adiciona o seguinte trecho em UP (Dados que serão inseridos no DB):
    email: {
      type: Sequelize.STRING
    },
  
  E então podemos executar a migration com:
    npx sequelize db:migrate
  Ou reverter a migration com:
    //O undo faz perder os dados da tabela
    npx sequelize db:migrate:undo
    //Reverter até uma migration específica
    npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
  Para verificar o que foi criado:
    mysql -u root -p
    show databases;
    USE orm_example;
    SHOW TABLES;
    SHOW COLUMNS FROM users;
  
  Foreign Key:
    Dentro da migration podemos adicionar uma chave que será a FK, essa chave deve ficar na migration da tabela que pegará a informação da outra.
    Alguma opções da FK:
      field: o nome que a FK receberá nessa tabela
      references.model : Indica qual tabela nossa FK está referenciando.
      references.key : Indica qual coluna da tabela estrangeira deve ser utilizada para nossa foreign key .
      onUpdate e onDelete : Configura o que deve acontecer ao atualizar ou excluir um usuário. Nesse caso, todos os produtos daquele usuário serão alterados ou excluídos.
    Exemplo de uma FK:
      employeeId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
        field: 'employee_id',
        references: {
          model: 'Employees',
          key: 'id',
        },
      },

Alterando uma tabela já existente:
  Rodar o comando:
    //Isso irá gerar um novo arquivo na pasta migration com um código contendo o UP e o DOWN
    //Devemos alterar o UP e o DOWN para adiciona e remover (respectivamente) a nova coluna que queremos adiciona nessa migration
    npx sequelize migration:generate --name add-column-phone-table-users
  Editar o UP e o DOWN dessa nova migration:
    up: async (queryInterface, Sequelize) => {
      await queryInterface.addColumn('Users', 'phone_num', {
        type: Sequelize.STRING,
      });
    },

    down: async (queryInterface, Sequelize) => {
      await queryInterface.removeColumn('Users', 'phone_num');
    }
  Rodar a migration:
    npx sequelize db:migrate
  Adicionar o novo campo ao User model para ele saber que o campo existe:
    phone_num: DataTypes.STRING,

  Além de adicionar ou remover colunas, o objeto queryInterface também permite que você altere a estrutura de uma coluna como seu tipo, valor default entre outros detalhes assim como o ALTER TABLE também permite. Você pode consultar esse link da documentação do Sequelize para ver como utilizar esse recurso. https://sequelize.org/master/manual/query-interface.html

Seeds:
  São responsáveis por alimentar o banco de dados.
  Muito parecido com uma migration
  Ex.:
    Quando criamos um banco de dados mas queremos colocar algumas informações nele para testar no front-end ou consulta a API.
  Criando a seed:
    npx sequelize seed:generate --name users
  Modificar a seed com os dados que desejamos.
  Ex.:
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
  Para adicionar os dados ao banco:
    npx sequelize db:seed:all
  Para reverter:
    npx sequelize db:seed:undo:all
  Para reverter uma seed específica
    npx sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
  Para verificar o que foi criado:
    mysql -u root -p
    show databases;
    USE orm_example;
    SHOW TABLES;
    SHOW COLUMNS FROM users;
    SELECT * FROM users;

Exemplos:
  Exemplos de consultas ao banco com um servidor express:
    const { User } = require('../models');
    const users = await User.findAll();
    const user = await User.findByPk(id);
    const user = await User.findOne({ where: { id, email }});

  Exemplo de inserção:
    const { User } = require('../models');
    const newUser = await User.create({ fullName, email });

  Exemplo de atualização de dado:
    const { fullName, email } = req.body;
    const { id } = req.params;

    const [updateUser] = await User.update(
      { fullName, email },
      { where: { id } },
    );

  Exemplo de como deletar:
    const { id } = req.params;
    const deleteUser = await User.destroy(
      { where: { id } },
    );

Boas práticas:
  Mudar o config/config.json para usar o dotEnv
    Renomear o arquivo para config.js
    Mudar o conteúdo do arquivo para (conforme está na pasta)
    Modifique a linha 8 do arquivo models/index.js para apontar para o arquivo config.js :
      const config = require(__dirname + '/../config/config.js')[env];
