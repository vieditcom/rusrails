# Запуск миграций

Rails предоставляет ряд задач rake для запуска определенных наборов миграций.

Самая первая команда Rake, относящаяся к миграциям, которую вы будете использовать, это `rake db:migrate`. В своей основной форме она всего лишь запускает метод `up` или `change` для всех миграций, которые еще не были запущены. Если таких миграций нет, она выходит.Она запустит эти миграции в порядке, основанном на дате миграции.

Заметьте, что запуск `db:migrate` также вызывает задачу `db:schema:dump`, которая обновляет ваш файл `db/schema.rb` в соответствии со структурой вашей базы данных.

Если вы определите целевую версию, Active Record запустит требуемые миграции (методы up, down или change), пока не достигнет требуемой версии. Версия это числовой префикс у файла миграции. Например, чтобы мигрировать к версии 20080906120000, запустите

```bash
$ rake db:migrate VERSION=20080906120000
```

Если версия 20080906120000 больше текущей версии (т.е. миграция вперед) это запустит метод `up` для всех миграций до и включая 20080906120000, но не запустит какие-либо поздние миграции. Если миграция назад, это запустит метод `down` для всех миграций до, но не включая, 20080906120000.

### Откат

Обычная задача это откатить последнюю миграцию. Например, вы сделали ошибку и хотите исправить ее. Можно отследить версию предыдущей миграци и произвести миграцию до нее, но можно поступить проще, запустив

```bash
$ rake db:rollback
```

Это запустит метод `down` последней миграции. Если нужно отменить несколько миграций, можно указать параметр `STEP`:

```bash
$ rake db:rollback STEP=3
```

это запустит метод `down` у 3 последних миграций.

Задача `db:migrate:redo` это ярлык для выполнения отката, а затем снова запуска миграции. Так же, как и с задачей `db:rollback` можно указать параметр `STEP`, если нужно работать более чем с одной версией, например

```bash
$ rake db:migrate:redo STEP=3
```

Ни одна из этик команд Rake не может сделать ничего такого, чего нельзя было бы сделать с `db:migrate`. Они просто более удобны, так как вам не нужно явно указывать версию миграции, к которой нужно мигрировать.

### Сброс базы данных

Задача `db:reset` удаляет базу данных, пересоздает ее и загружает в нее текущую схему.

NOTE. Это не то же самое, что запуск всех миграций. Оно использует только текущее содержимое файла schema.rb. Если миграция не может быть откачена,
'rake db:reset' может не помочь вам. Подробнее об экспорте схемы смотрите "Экспорт схемы":/rails-database-migrations/schema-dumping-and-you.

### Запуск определенных миграций

Если вам нужно запустить определенную миграцию (up или down), задачи `db:migrate:up` и `db:migrate:down` сделают это. Просто определите подходящий вариант и у соответствующей миграция будет вызван метод `up` или `down`, например

```bash
$ rake db:migrate:up VERSION=20080906120000
```

запустит метод `up` у миграции 20080906120000. Эта задача сперва проверят, была ли миграция уже выполнена, и ничего делать не будет, если Active Record считает, что она уже была запущена.

### Запуск миграций в различных средах

По умолчанию запуск `rake db:migrate` запустится в окружении `development`. Для запуска миграций в другом окружении, его можно указать, используя переменную среды `RAILS_ENV` при запуске команды. Например, для запуска миграций в среде `test`, следует запустить:

```bash
$ rake db:migrate RAILS_ENV=test
```

### Изменение вывод результата запущенных миграций

По умолчанию миграции говорят нам только то, что они делают, и сколько времени это заняло. Миграция, создающая таблицу и добавляющая индекс, выдаст что-то наподобие этого

```bash
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Некоторые методы в миграциях позволяют вам все это контролировать:

|Метод             |Назначение|
|------------------|----------|
|suppress_messages |Принимает блок как аргумент и запрещает любой вывод, сгенерированный этим блоком.|
|say               |Принимает сообщение как аргумент и выводит его как есть. Может быть передан второй булевый аргумент для указания, нужен отступ или нет.|
|say_with_time     |Выводит текст вместе с продолжительностью выполнения блока. Если блок возвращает число, предполагается, что это количество затронутых строк.|

Например, эта миграция

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    suppress_messages do
      create_table :products do |t|
        t.string :name
        t.text :description
        t.timestamps
      end
    end

    say "Created a table"

    suppress_messages {add_index :products, :name}
    say "and an index!", true

    say_with_time 'Waiting for a while' do
      sleep 10
      250
    end
  end
end
```

сгенерирует следующий результат

```bash
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

Если хотите, чтобы Active Record ничего не выводил, запуск `rake db:migrate VERBOSE=false` запретит любой вывод.