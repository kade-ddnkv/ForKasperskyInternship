# ForKasperskyInternship

[ссылка_на_тз](https://drive.google.com/file/d/1pBc_mRXTsE-OOpGEeA0PP7bf7iG7PjqE/view?usp=sharing)

### Ссылки на репозитории двух реализованных приложений:

[scan_service](https://github.com/kade-ddnkv/ScanService_ForKasperskyInternship)

[scan_util](https://github.com/kade-ddnkv/ScanUtil_ForKasperskyInternship)


# Инструкция по использованию:

Для запуска приложения, насколько я понимаю, нужен установленный IIS EXPRESS.

### Запустить scan_service.exe  
Он откроется в отдельном окне.  
Для закрытия нужно нажать Ctrl+C или просто закрыть окно.  
(это также будет указано при запуске).

### Запустить утилиту scan_util.exe (через командную строку с аргументами)  
Поддерживаются две команды:  
- `scan_util.exe scan %userprofile%\Documents`
- `scan_util.exe status 1234`

Любой некорректный ввод обрабатывается (по идее), и на консоль выводится соответствующее сообщение.

### Пример запуска программы
Заранее извиняюсь за неудобоваримое название программы, однако не знаю, как это корректно исправить.

[Пример](https://imgur.com/a/27P5FkZ)

Место хранения программ версии release (по крайней мере при использовании VisualStudio):  
scan_service.exe = ...\ScanService_ForKasperskyInternship\ScanService_ForKasperskyInternship\bin\Release\net5.0\ScanService_ForKasperskyInternship.exe  
scan_util.exe = ...\ScanUtil_ForKasperskyInternship\ScanUtil_ForKasperskyInternship\bin\Release\net5.0\ScanUtil_ForKasperskyInternship.exe

---

## Примечания по решению:
**Использовались NuGet-пакеты:**
- Newtonsoft.Json - в scan_util
- Swashbuckle.AspNetCore - в scan_service

Я пытался установить статический порт для веб-приложения, но столкнулся с трудностями. По идее, программа должна работать на дефолтных портах 5000 и 5001.

## Алгоритм:

При **создании задачи** новая задача на сканирование директории добавляется в очередь на выполнение. При помощи ContinueWith она присоединяется к последней поставленной задаче.  
*(Примечание: если не использовать такую очередь, все становится хуже.
Рассмотрим поставленные одновременно 3 задачи (1 задача = 20 секунд).
При параллельном выполнении без очереди время ожидания их выполнения = 60, 60, 60 (проверено на практике), 
тогда как при последовательном ожидание выполнения будет 20, 40, 60, что, очевидно, лучше, так как результат одной задачи будет доступен раньше.)*
*Конечно, можно найти случаи, в котороых такое решение проиграет, например, сканируются 2 директории: сначала большая, потом маленькая. 
Тогда маленькая будет обязана ждать окончания сканирования большой. Но, мне кажется, это решение вполне эффективно.*

Далее в задаче для проверки каждого файла запускается параллельная задача.

Далее было два решения:
- В файле параллельно запускать проверку для каждой строки (многопоточность).
- Или пройтись по строкам в файле сверху вниз (один поток).

Как показал замер времени на примере, многопоточное решение эффективнее. Поэтому в релизе используется именно оно.
Хотя это и немного странно, ведь 1) Тратится время на создание параллельной задачи, 2) Ядра должны быть все время заняты, т.к. много файлов.

*Замер времени на примере (128 файлов с 10000 строками по 10000 символов в каждом = ~12 ГБ):*  
- *параллельно - 24 секунды в среднем.*
- *в один поток - 34 секунды в среднем.*  

### Характеристики системы, на которой делались проверки:  
(Процессор: Intel(R) Core(TM) i5-8265U CPU @ 1.60GHz, 1800 МГц, ядер: 4, логических процессоров: 8)
