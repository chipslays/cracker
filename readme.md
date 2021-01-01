# 🍪 Cracker

Автоматическое распознавание капчи в игровом сервисе Steam.

## Проблема

При написании библиотеки [Steam Client](https://github.com/aethletic/steam-client), я сталкивался с вводом капчи. В какой-то момент постоянный ввод капчи стал надоедать, поэтому было принято решение сделать распознование капчи.

## Решение

Для распознования используется [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) + отдельно обученная модель (языковой файл) на основе 5000 разных капч. 

Обучение модели сделано на основе [этого](https://github.com/guiem/train-tesseract) репозитория. 

**Процесс распознования следующий:** каждый раз ссылка на картинку с капчей генерирует один и тот же текст в течении некоторого времени, а потом умирает. Но текст на картинке всегда будет один и тот же, меняется только хаотичность букв и шума. Поэтому на основе этого, задается количество итераций распознования, например 10 итераций, после окончания цикла, выбираются те символы которые встречались чаще всего.

## Установка

```bash
$ composer require aethletic/cracker
```

## Использование

```php
use Cracker\Crack;

require 'vendor/autoload.php';

$cracked = (new Crack('https://steamcommunity.com/public/captcha.php?gid=387475048XXXXXXXXXXXXXXXX'))
    ->storage(__DIR__ . '/storage')
    ->data(__DIR__)
    ->model('steam')
    ->iterations(3)
    ->resolve(true);

print_r($cracked);

/** output */
Array
(
    [sortedChars] => Array
        (
            [0] => Array
                (
                    [M] => 3
                )
            [1] => Array
                (
                    [J] => 3
                )
            [2] => Array
                (
                    [X] => 3
                )
            [3] => Array
                (
                    [N] => 3
                )
            [4] => Array
                (
                    [P] => 3
                )
            [5] => Array
                (
                    [9] => 3
                )
        )
    [mostUsedChars] => Array
        (
            [0] => M
            [1] => J
            [2] => X
            [3] => N
            [4] => P
            [5] => 9
        )
    [captcha] => MJXNP9
    [time] => 2.0638
)
```

**Капча:** ![MJXNP9](https://raw.githubusercontent.com/aethletic/cracker/master/.github/captcha.png)

Как видно из результата, в `sortedChars` все 3 итерации распознали одни и те же символы.

Время выполнение (ключ `time`) заняло 2 секунды, т. е. чем больше итераций, тем больше времени занимает. 

Обученной мной модели достаточно 1 итерации (~0.6 секунды) для корректного распознования, но так как изображение модифицируется, в некоторых случаях бывают артефакты, когда буква залита или наоборот ее не видно, либо видно только маленький кусок, в таких случаях конечно есть погрешности.

> **Внимание!** Этот репозиторий не содержит обученную модель, вам необходимо самостоятельно собрать данные и обучить ее. Сделано это в соображениях безопасности и предотвращении распространения. Информация по обучению находится [здесь](https://tesseract-ocr.github.io/tessdoc/TrainingTesseract-4.00.html).

Вы можете использовать эту библиотеку следующем образом:

```php
$cracked = (new Crack($url))
    ->storage(__DIR__ . '/storage')
    ->model('eng')
    ->iterations(10)
    ->resolve(true);
```

Для модели `eng` нужно больше итерации и времени, но качество распознавания оставляет желать лучшего.





