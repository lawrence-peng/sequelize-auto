# Sequelize-build

[![Greenkeeper badge](https://badges.greenkeeper.io/sequelize/sequelize-auto.svg)](https://greenkeeper.io/)

[![Build Status](http://img.shields.io/travis/sequelize/sequelize-auto/master.svg)](https://travis-ci.org/sequelize/sequelize-auto) [![Build status](https://ci.appveyor.com/api/projects/status/bf9lb89rmpj6iveb?svg=true)](https://ci.appveyor.com/project/durango/sequelize-auto) [![Dependency Status](https://david-dm.org/sequelize/sequelize-auto.svg)](https://david-dm.org/sequelize/sequelize-auto) [![Code Climate](https://codeclimate.com/github/sequelize/sequelize-auto/badges/gpa.svg)](https://codeclimate.com/github/sequelize/sequelize-auto) [![Test Coverage](https://codeclimate.com/github/sequelize/sequelize-auto/badges/coverage.svg)](https://codeclimate.com/github/sequelize/sequelize-auto/coverage)

Automatically generate models for [SequelizeJS](https://github.com/sequelize/sequelize) via the command line.

## Install

    npm install -g sequelize-build

## Prerequisites

You will need to install the correct dialect binding globally before using sequelize-build.

Example for MySQL/MariaDB

`npm install -g mysql`

Example for Postgres

`npm install -g pg pg-hstore`

Example for Sqlite3

`npm install -g sqlite`

Example for MSSQL

`npm install -g tedious`

## Usage

    [node] sequelize-auto -h <host> -d <database> -u <user> -x [password] -p [port]  --dialect [dialect] -c [/path/to/config] -o [/path/to/models] -t [tableName] -C

    Options:
      -h, --host        IP/Hostname for the database.   [required]
      -d, --database    Database name.                  [required]
      -u, --user        Username for database.
      -x, --pass        Password for database.
      -p, --port        Port number for database.
      -c, --config      JSON file for Sequelize's constructor "options" flag object as defined here: https://sequelize.readthedocs.org/en/latest/api/sequelize/
      -o, --output      What directory to place the models.
      -e, --dialect     The dialect/engine that you're using: postgres, mysql, sqlite
      -a, --additional  Path to a json file containing model definitions (for all tables) which are to be defined within a model's configuration parameter. For more info: https://sequelize.readthedocs.org/en/latest/docs/models-definition/#configuration
      -t, --tables      Comma-separated names of tables to import
      -T, --skip-tables Comma-separated names of tables to skip
      -C, --camel       Use camel case to name models and fields
      -n, --no-write    Prevent writing the models to disk.
      -s, --schema      Database schema from which to retrieve tables

## Example

    sequelize-auto -o "./models" -d sequelize_auto_test -h localhost -u my_username -p 5432 -x my_password -e postgres

Produces a file/files such as ./models/Users.js which looks like:

    /* jshint indent: 2 */

    module.exports = function(sequelize, DataTypes) {
      return sequelize.define('Users', {
        id: {
          type: DataTypes.INTEGER(11),
          allowNull: false,
          primaryKey: true,
          autoIncrement: true
        },
        username: {
          type: DataTypes.STRING,
          allowNull: true
        },
        touchedAt: {
          type: DataTypes.DATE,
          allowNull: true
        },
        aNumber: {
          type: DataTypes.INTEGER(11),
          allowNull: true
        },
        bNumber: {
          type: DataTypes.INTEGER(11),
          allowNull: true
        },
        validateTest: {
          type: DataTypes.INTEGER(11),
          allowNull: true
        },
        validateCustom: {
          type: DataTypes.STRING,
          allowNull: false
        },
        dateAllowNullTrue: {
          type: DataTypes.DATE,
          allowNull: true
        },
        defaultValueBoolean: {
          type: DataTypes.BOOLEAN,
          allowNull: true,
          defaultValue: '1'
        },
        createdAt: {
          type: DataTypes.DATE,
          allowNull: false
        },
        updatedAt: {
          type: DataTypes.DATE,
          allowNull: false
        }
      }, {
        tableName: 'Users',
        freezeTableName: true
      });
    };


Which makes it easy for you to simply [Sequelize.import](http://docs.sequelizejs.com/en/latest/docs/models-definition/#import) it.

## Configuration options

For the `-c, --config` option the following JSON/configuration parameters are defined by Sequelize's `options` flag within the constructor. For more info:

[https://sequelize.readthedocs.org/en/latest/api/sequelize/](https://sequelize.readthedocs.org/en/latest/api/sequelize/)

## Programmatic API

```js
var SequelizeAuto = require('sequelize-build')
var auto = new SequelizeAuto('database', 'user', 'pass');

auto.run(function (err) {
  if (err) throw err;

  console.log(auto.tables); // table list
  console.log(auto.foreignKeys); // foreign key list
});

With options:
var auto = new SequelizeAuto('database', 'user', 'pass', {
    host: 'localhost',
    dialect: 'mysql'|'mariadb'|'sqlite'|'postgres'|'mssql',
    directory: false, // prevents the program from writing to disk
    port: 'port',
    additional: {
        timestamps: false
        //...
    },
    tables: ['table1', 'table2', 'table3']
    //...
})

width template:

'use strict'

const path = require('path')
const SequelizeAuto = require('sequelize-build')

const modelsDir = path.join(__dirname, './models')

const auto = new SequelizeAuto('database', 'user', 'pass', {
  host: 'localhost',
  port: 3306,
  dialect: 'mysql',
  template: {
    base: {
      directory: modelsDir+'/base',
      templatePath:  path.join(__dirname, 'base_model.js')
    },
    model:{
      use:true,
      directory: modelsDir,
      templatePath:  path.join(__dirname, 'model.js')
    }
  },
  directory: modelsDir,
  omitNull: true,
  // 指定生成的 models 文件的缩进格式以匹配 ESlint 规则
  spaces: true,  // 使用空格缩进
  indentation: 4,  // 使用 4 空格缩进
  additional: {
    timestamps: true,
    createdAt: '"created_at"',
    updatedAt: '"updated_at"',
    deletedAt: '"deleted_at"',
    paranoid: true
  }
})

console.log('> syncing models...')
auto.run(err => {
  if (err) {
    console.error(err)
  } else {
    console.log('> done.')
  }
})


```

## Template engine

* template engine use handlebars more see [handlebars](http://handlebarsjs.com/)

Example

  base_model.js

```js
module.exports = `/* jshint indent: 4 */
module.exports = function(app) {
    const DataTypes = app.Sequelize
    return {{{fields}}}
}`
```

 model.js

```js
module.exports = `/* jshint indent: 4 */
const columns = require('./base/{{tableName}}')
module.exports = app => {
  return app.model.define('{{tableName}}',columns(app), {
    tableName: '{{tableName}}',
    timestamps: true,
    paranoid: true
  })
}`
```

## Testing

You must setup a database called `sequelize_auto_test` first, edit the `test/config.js` file accordingly, and then enter in any of the following:

    # for all
    npm run test

    # mysql only
    npm run test-mysql

    # postgres only
    npm run test-postgres

    # postgres native only
    npm run test-postgres-native

    # sqlite only
    npm run test-sqlite

## Projects Using Sequelize-Auto

* [Sequelizer](https://github.com/andyforever/sequelizer)
