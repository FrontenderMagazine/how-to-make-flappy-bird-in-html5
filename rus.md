# Как сделать Flappy Bird на HTML5 с помощью Phaser

Flappy Bird небольшая милая игра с простой для понимания механикой, и я подумал, что
она будет превосходным примером для обучающего материала о играх на HTML5. Поэтому в
этой статье мы попробуем сделать более простую версию Flappy Bird написав лишь только
65 строчек JavaScript-кода.

[Взгляните на игру][1], которую мы собираемся сделать.

![][2]

> Замечание 1: вам необходимо знать немного JavaScriptб чтобы понять эту статью.

> Замечание 2: Если вас заинтересует фреймворк Phaser, то вы можете узнать о нём
> больше, прочитав мою короткую статью [быстрый старт c Phaser][3].


## Настройка

Вы должны скачать [базовый шаблон][4], что я сделал для этой статьи. В нём вы найдёте:

*   phaser.min.js — минифицированный фреймворк Phaser версии v1.1.5.
*   index.html — страничка на которой мы будем играть.
*   main.js — файл, в котором мы будем писать код.
*   assets/ — папка с двумя картинками (птичка и труба).

Разместите эти файлы на вашем сервере. Откройте index.html в вашем браузере, а main.js
— редакторе или IDE.

В файле main.js вы должны увидеть базовую структуру проекта на Phaser, которую мы
обсуждали в прошлой статьей.

    // инициализировать Phaser и создать игру на поле размером 400x490px 
    var game = new Phaser.Game(400, 490, Phaser.AUTO, 'game_div');  
    var game_state = {};
    
    game_state.main = function() { };  
    game_state.main.prototype = {
    
        preload: function() { 
            // загрузку всей необходимой статики
        },
    
        create: function() { 
            // настройка игры; вызывается сразу после preload
        },
    
        update: function() {
            // вызывается 60 раз в секунду
        },
    };
    
    game.state.add('main', game_state.main);  
    game.state.start('main');

Теперь мы собираемся реализовать функции `preload()`, `create()` и `update()` и
добавить новые недостающие для полноценной игры функции.


## Птичка!

Итак, начнём программировать! Давайте сначала сосредоточимся на птичке, которой можно
было бы управлять с помощью клавиши «пробела».

Всё достаточно просто и хорошо документировано, поэтому вам должно быть просто понять
код. Для лучшей читаемости я удалил инициализацию Phaser и код отвечающий за контроль
состояний (этот код вы могли видеть выше).

Сначала, мы допишем функции `preload()`, `create()` и `update()`.

    preload: function() {  
        
        // сменим фон игры
        this.game.stage.backgroundColor = '#71c5cf';
        
        // загрузим картинку Птички
        this.game.load.image('bird', 'assets/bird.png'); 
    },
    
    create: function() {  
        // покажем птичку на экране
        this.bird = this.game.add.sprite(100, 245, 'bird');
    
        // добавим гравитацию, заставив птичку падать
        this.bird.body.gravity.y = 1000;  
    
        // добавим функцию jump обработчиком на нажатие пробела
        var space_key = this.game.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
        space_key.onDown.add(this.jump, this);     
    },
    
    update: function() {  
        // если птичка вылетела за пределы (слишком высоко или слишком низко),
        // то необходимо вызвать функцию restart_game
        if (this.bird.inWorld == false)
            this.restart_game();
    },
    

И после этого добавим две новые функции.

    // поможем птичке подпрыгнуть
    jump: function() {  
        // добавляем вертикальное скорость птице
        this.bird.body.velocity.y = -350;
    },
    
    // начинаем игру заново
    restart_game: function() {  
        // запускаем состояние "main", которое рестартует игру
        this.game.state.start('main');
    },
    

Сохраните файл main.js с новым кодом и обновите index.html. Вы должны увидеть птичку
подпрыгивающую при нажатии проблела.


## Трубы

Игра Flappy Bird не будет такой интересной без труб, поэтому давайте добавим их.

Сначала надо загрузить изображение трубы в функции `preload()`.

    this.game.load.image('pipe', 'assets/pipe.png');          

Затем надо создать группу труб в функции `create()`. Мы собираемся обрабатывать
большое количество труб в игре, поэтому будет проще использовать группу объектов.
Она будет содержать 20 труб, с которыми мы вольны делать всё, что душе угодно.

    this.pipes = game.add.group();  
    this.pipes.createMultiple(20, 'pipe');      

Теперь нам требуется функция добавляющая трубу в игру. По умолчанию, трубы в группе
«мертвы» и не показываются. Итак, мы берём неиспользованную трубу, показываем её и
следим за тем, чтобы она автоматически убивалась, когда становится невидимой. Таким
образом нам всегда будет хватать труб.

    add_one_pipe: function(x, y) {  
        // получаем первую неиспользованную трубу из группы
        var pipe = this.pipes.getFirstDead();
    
        // позиционируем трубу на игровом поле
        pipe.reset(x, y);
    
        // добавим скорость трубе, чтобы она двигалась влево
        pipe.body.velocity.x = -200; 
    
        // убивам трубу, когда она становится невидимой
        pipe.outOfBoundsKill = true;
    },

Предыдущая функция покажет нам только одну трубу, но нам нужны шесть с промежутками
между ними. Так давайте напишем соответствующую функцию.

    add_row_of_pipes: function() {
        var hole = Math.floor(Math.random()*5) + 1;
    
        for (var i = 0; i < 8; i++) {
            if (i !== hole && i !== (hole + 1)) {
                this.add_one_pipe(400, i*60 + 10);
            }
        }
    },

Теперь, чтобы трубы появлялись, надо вызывать функцию `add_row_of_pipes()` каждые 1.5
секунды. Для этог мы можем добавить соответствующий таймер в функцию `create()`.

    this.timer = this.game.time.events.loop(1500, this.add_row_of_pipes, this);    

И наконец добавим строчку кода, останавливающую таймер в начало функции
`restart_game()`.

    this.game.time.events.remove(this.timer);    

Теперь вы можете сохранить файл и попробовать поиграть. Наша затея медленно
превращается в настоящую игру.


## Подсчёт очков и столкновения

Последняя вещь, что нам необходимо добавить в игру это добавить очки и научиться
обрабатывать столкновения. И это очень просто сделать.

Добавьте эти строчки в функцию `create()`, чтобы показать счёт в левом верхнем игру.

    this.score = 0;
    var style = { font: "30px Arial", fill: "#ffffff" };
    this.label_score = this.game.add.text(20, 20, "0", style);

А это добавьте в `add_row_of_pipes()`, чтобы инкрементировать счёт каждый раз, когда
новая создаётся новая труба.

    this.score += 1;
    this.label_score.content = this.score;

Эту строчку, в свою очередь, надо добавить в функцию `update()`, чтобы вызвать
`restart_game()` каждый раз, когда птичка встречается с роковой трубой.

    this.game.physics.overlap(this.bird, this.pipes, this.restart_game, null, this);

Эгей, мы закончили! Я вас поздравляю, теперь у вас есть HTML5 клон игры Flappy Bird.
Вы можете [скачать исходники][5] с гитхаба.


## Что дальше?

Игра работает, но она немного скучная. В следующей статье мы узнаем как сделать её
интереснее с помощью звуков, анимации, меню и прочего. Поэтому, если вы неподписаны на
рассылку, то обязательно подпишитесь, чтобы узнать, когда будет готова следующая
статья (форма подписки доступна на странице оригинальной статьи, — прим. переводчика).

Также вы можете взглянуть на мой чэллендж (!!!) «по одной HTML5-игре в неделю» на сайте [lessmilk.com][6].



 [1]: http://www.lessmilk.com/flappy_bird/
 [2]: img/FB-1.png
 [3]: http://blog.lessmilk.com/make-html5-games-with-phaser-1/
 [4]: https://github.com/lessmilk/phaser-tutorials/blob/master/2-flappy_bird/basic_template.zip?raw=true
 [5]: https://github.com/lessmilk/phaser-tutorials/blob/master/2-flappy_bird/flappy_bird.zip?raw=true
 [6]: http://www.lessmilk.com