# Ответы на тестовое задание от FUNBOX

# level I

## Q1

**Чем отличается proc от lambda?** 

Хоть proc и lambda принадлежат одному классу Proc, у них есть пара отличий

lambda всегда проверяет количество данных аргументов, в то время как proc'у плевать на это с высокой колокольни

```ruby
lam = lambda { |fir, sec| puts fir, sec }
lam.call("fir")
=> ArgumentError (wrong number of arguments (given 1, expected 2))
prc = Proc.new { |fir, sec| puts "#{(fir.nil?) ? 'fir is not given' : fir } and #{(sec.nil?) ? 'sec is not given' : sec }" }
prc.call
=> fir is not given and sec is not given
```

Также, вызов метода return из тела lambda не прерывает исполнения блока внешней функции, proc же игорирует все после его return'a 

```ruby
def lambr
	lam = lambda { return }
	lam.call
	puts "it will be shown"
end
lambr()
=> it will be shown
def procr
	prc = Proc.new { return }
	prc.call
	puts "it will never be shown"
end
procr()
=>
```

## Q2

**Чем отличается && от and?**

по сути, только приоритетом выполнения: оператор && имеет более высокий приоритет, нежели and

пример
```ruby
a = 220
b = 440
a || b and nil
=> nil
a || b && nil
=> 220
```

## Q3

**Какие плюсы и минусы Ruby по сравнению с другими современными языками
программирования вы выделяете для себя?**

## Вот реальные плюсы, которые я выделил именно для себя:
- Синтаксис близкий к нормальному человеческому языку(без шуток, я показывал коды некоторых своих проектов друзьям, далеким от программирования, и они отдаленно, но понимали, что происходит в моем коде)
- Огромное множество гемов на любой чих и на любую, даже самую очевидную потребность
- Довольно дружелюбное коммьюнити, которое не будет закидывать камнями, если ты не понимаешь как вывести свой заветный "hello world"
- Полностью объектно-ориентированный язык
## Минусы:
- Global Interpreter Lock, а значит никакого нормального многопотока(JRuby в помощь)
- Стандартный интерпретатор жрет как не в себя
- Также довольно медлительный интерпретатор
- Сложно найти работу

## Q4

**В каком случае вы бы стали использовать Ruby on Rails для разработки web приложения, а в каком — другой фреймворк?**

RoR я бы использовал, если бы мне нужен был производительный, безопасный и динамичный сайт, с мейлерами, простым тестированием и множеством библиотек, с помощью которых я бы мог максимально быстро поставить готовый проект заказчику. Во всех других случаях можно обойтись и golang'ом, поставляющим статичные html документы пользователю

## Q5

Вы запустили большой и сложный ruby скрипт на вашем любимом дистрибутиве
Linux, и в процессе работы он намертво «зависает». Как найти причину сбоя? 

Для начала надо убить процесс через htop
```
$ sudo emerge htop
```
```
$ htop
```
**Дальше нам следует залоггировать каждый подозрительный момент в нашей программе и проверять места, с которых наша программа умирает**

```ruby
require 'logger'

logger = Logger.new(STDOUT)
logger.level = Logger::WARN

logger.warn("Some bad stuff is going on here")
while true do
 p "hehee"
end
```

## Q6

**Вы запустили большой и сложный ruby скрипт на вашем любимом дистрибутиве
Linux, и в процессе работы он умирает, исчерпав память, хотя, казалось бы, не должен. Как найти причину сбоя?**

Для начала, нам следует просмотреть все места, где обрабатываются объекты классов ```Array```, ```Hash``` и ```String```. Каждый шаг цикла или каждое действие над ними следует залоггировать, к примеру, с ```get_process_mem```

```ruby
require "get_process_mem"

def log_mem(desc)
  	mb = GetProcessMem.new.mb
  	puts "#{ desc } - MEMORY USAGE(MB): #{ mb.round }"
end

r = Random.new
arr = []
log_mem(0)
10000.times do |m|
	log_mem(m)
	arr <<	r.rand(10000) 
end
```

## Q7

**Некоторые проекты (Rails и не только) мы запускаем под JRuby. Как вы думаете, почему?** 

- JRuby быстрее, ведь он работает на великолепной JVM
- Так как больше мы не имеем проблем с GIL, мы получаем нормальный многопоток
- Есть возможность интеграции с Java кодом

## Q8
 
**Какие плюсы и минусы библиотеки ActiveRecord в Rails вы знаете? Какие альтернативы существуют? В чем их плюсы и минусы? Какие из них вы использовали?**

## Плюсы:
* Количество документации
* Большинство библиотек интегрированы именно с AR
## Минусы:
* Медлительность: Тот же Sequel выполняет запросы в раза 1.5-2 быстрее AR
* Для написания сложных запросов все равно приходится все делать через голый SQL
* Невозможно создать больше одного объекта за один запрос
* Пусть мы и имеем Ares, запросы все равно будут выглядеть отвратительно и кто-то, кроме тебя вряд ли сможет понять писанину, которую ты любезно оставишь в своем проекте

Из реально стоящий альтернатив можно отметить Sequel, с которым я имел возможность работать только в парочке тестовых проектов. Он использует протокол расширенного запроса PG, который делает его значительно быстрее AR и сильно снижает жор ресурсов на сервере

# Level II
## Q1
**Есть Rails приложение, развернутое на N серверах. В приложение нужно добавить загрузку и хранение аватаров.**
* Какую библиотеку использовать?
* Куда складывать аватарки и почему?
* Как и зачем нужно обрабатывать загружаемые аватарки?
* Какие проверки загружаемого файла и когда (например, на клиенте, в Nginx или Rails) лучше всего производить?
* Как отдавать аватарки пользователю? Какие плюсы и минусы описанного подхода?
* Какие еще проблемы и вопросы могут возникнуть при реализации такой функциональности? 
## Ответы 
* Я бы использовал библиотеку CarrierWave вместо записи в байтовом формате напрямую в базу данных. Так мы сможешь снизить загрузку на нашу драгоценную DB и избежать записи лишней информации, никак не связанной с изображением.
* Аватарки можно складывать в каком-нибудь облачном хранилище по типу Amazon S3. Так мы сможем получить доступ к нашим файлам с любого из n серверов
* Обрабатывать аватарки можно с помощью ImageMagick на серверной стороне, чтобы сжимать изображания перед сохранением и избежать лишнего использования диского пространства
* С помощью виртуального атрибута, который мы присваем с помощью Uploader'a у CarrierWave. Единственная проблема - это то, что он будет пытаться присвоить url динамически и не всегда это делает так, как надо. Решить можно с помощью одной линии в конфиге нашего аплоадера. Но зато мы имеет безопасную и быструю систему раздачи аватарок нашим пользователям
```ruby
def uid
  '58406672-c227-4cec-a846-04c443e70f33'
end
def store_dir
  "uploads/#{uid}"
end
```
* Из минусов:
	* Проблемы с поддержкой китайского
	* При апгрейде с 1.3.1 на 2.0.0 у некоторых ломалось приложение

* Больше минусов, как по мне, нет ;)

## Q2 

Есть большое приложение с десятками тысяч тестов. Как лучше всего организовать код и тесты, чтобы добиться максимальной скорости прогона тестов? 

Каждому 
