# Использование Chrome DevTools - анализ открытия сайта при Slow параметрах

# Выполнил Дружинин Михаил ШРИ-2023

## Содержание <a name="contents"></a>

- [Описание](#about)
- [1.Вкладка Network](#network)
- [2.Вкладка Performance](#performance)
- [3.Вкладка Coverage](#coverage)

## Описание <a name="about"></a>

Тестировалась desktop-версию сайта https://www.gd.ru/articles/9039-finansovyy-kontrol при следующих параметрах :  замедление CPU 4x slowdown и эмуляция сети Slow 3G.

Данные на скриншотах (время запросов, размеры ресурсов и т.д.) могут незначительно отличаться от приложенных HAR-архива и профиля Performance.

## 1.Вкладка Network <a name="network"></a>

([вернуться к содержанию](#contents))

Профиль загрузки ресурсов при открытии страницы https://www.gd.ru/articles/9039-finansovyy-kontrol ([HAR-архив](3_2_Tooling_Slow/www.gd.ru.har)).

### Дублирование ресурсов.

Аналогично как и для случаю без троттлинга.
Присутствуют одинаковых запросы скриптов.

![code_same1](3_2_Tooling/1%20Network/double/1.png)

![code_same2](3_2_Tooling/1%20Network/double/3.png)

Наблюдаются одинаковых запросы на загрузку шрифтов.

![ttf_same1](3_2_Tooling/1%20Network/double/2.png)

Несколько раз идет запрос на загрузку библиотеки стилей bootstrap.

![css_same1](3_2_Tooling/1%20Network/double/4.png)

### Лишний размер ресурсов

Аналогично как и для случаю без троттлинга.
Многие картинки не до конца оптимизированы. Из изображений неоптимальными считаются файлы с разрешением выше необходимого для текущего размера экрана, а также загружаемые в не оптимальном формате (например `.jpg`, если можно использовать более компактный`.svg`)

![nonoptim_img](3_2_Tooling/1%20Network/big_size/2.png)

Из скриптов неоптимальными считаются не минифицированные и не сжатые файлы.

![nonoptim_scripts](3_2_Tooling/1%20Network/big_size/1.png)

### Медленно загружающиеся ресурсы

При заданных параметрах троттлинга, выделить какие-либо отдельные ресурсы, тормозящие процесс загрузки страницы, не представляется возможным. Ибо грузится все очень долго.

### Ресурсы, блокирующие загрузку

Аналогично как и для случаю без троттлинга.
Для проверки блокирующих ресурсов используем вкладку `Network request blocking`.
Обычно блокирующим ресурсом, будет любой js-скрипт, подключенные в секции head без атрибутов `async/defer`.

![block_load1](3_2_Tooling/1%20Network/block/1.png)

![block_load2](3_2_Tooling/1%20Network/block/2.png)

## 2.Вкладка Performance <a name="performance"></a>

([вернуться к содержанию](#contents))

Профиль загрузки страницы https://www.gd.ru/articles/9039-finansovyy-kontrol для экономии сжат в zip ([посмотреть](3_2_Tooling_Slow/Trace-20230615T131806.zip)).

Время от начала загрузки до события First Paint `~9020ms`.

![FP](3_2_Tooling_Slow/2%20Perfomance/FP.png)

Время от начала загрузки до события First Contentful Paint `~9020ms`.

![FCP](3_2_Tooling_Slow/2%20Perfomance/FCP.png)

Время от начала загрузки до события DOM Content Loaded `~38104ms`.

![DCL](3_2_Tooling_Slow/2%20Perfomance/DCL.png)

Время от начала загрузки до события Largest Contentful Paint `~48812ms`.

![LCP](3_2_Tooling_Slow/2%20Perfomance/LCP.png)

Время от начала загрузки до события Load `~69418ms`.

![Load](3_2_Tooling_Slow/2%20Perfomance/Load.png)

Общее время на обработку документа:

- Loading (`~286ms`)
- Scripting (`~10357ms`)
- Rendering (`~7386ms`)
- Painting (`~780ms`)

![Total time](3_2_Tooling_Slow/2%20Perfomance/2_4.png)

## 3.Вкладка Coverage <a name="coverage"></a>

([вернуться к содержанию](#contents))

Аналогично как и для случаю без троттлинга.
Вкладка `Coverage` после загрузки страницы https://www.gd.ru/articles/9039-finansovyy-kontrol выглядит следующим образом.
Общий объем скачанных ресурсов (JS + CSS) составил 4,3 Mb, реально использовано из них 1,5Mb (35%). Это свидетельствует о неоптимированности страницы.

![coverage](3_2_Tooling_Slow/3%20Coverage/3_1.png)

Объем неиспользованного CSS `~565 KB` (`~74%` от всего объема загруженного CSS `760kb`). Крайне низкий показатель.

![coverage_css](3_2_Tooling_Slow/3%20Coverage/3_2.png)

Некоторые CSS файлы после загрузки не используются вообще (100% unused).

![coverage_css2](3_2_Tooling/3%20Coverage/3_2_2.png)

Объем неиспользованного JavaScript - `2.4 MB` из загруженных `3.8Mb(`62%` от всего JS).

![coverage_js](3_2_Tooling_Slow/3%20Coverage/3_3.png)

Реальный объем неиспользуемого кода (на который может быть направлены возможная оптимизация) будет существенно меньше, т.к.:
- часть CSS/JS принадлежит сторонним ресурсам, например Яндекс.Метрики, Google.Analytics и т.п. 
- следует учесть, что учтен использованный CSS/JS на момент загрузки страницы. При взаимодействии пользователя с интерфейсом будет исполняться JS и объем неиспользованного кода в некоторой степени сократиться.
