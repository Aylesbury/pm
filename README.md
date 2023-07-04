**Ссылки**
* [Загрузка данных excel в справочник](#Загрузка-данных-excel-в-справочник)
* [Вывод суммы прописью](#Пример-вывода-суммы-прописью)
* [Подсчет итоговой суммы в табличном документе и рассчет](#Код-подсчета-итоговой-суммы-в-табличном-документе)
* [Код подстановки суммы в табличную часть](#Код-для-вставки-суммы-со-справочника-в-табличную-часть-документа)

# Загрузка данных excel в справочник
**Перед загрузкой файла необходимо перевести формат ячеек в** `R1C1`
1. Необходимо в списке конфигуратора найти `Обработка` затем создать ее и назвать `ЗагрузкаДанныхИзФайла`
2. Затем создать реквизит который будет содержать путь файла `ПутьКФайлу`, тип данных строка, длинна 256.
3. Далее создать табличную часть для предварительнных данных `ДанныеФайла` и в данной табличной части необходимо создать столько реквизитов сколько находится в таблице excel.
4. После этого необходимо создать форму обработки, в коммандной панели необходимо убрать галочку на автозаполнении чтобы убрать кнопку `еще`, так же и для табличной части.
5. Далее необходимо нажать на `ДанныеФайла` и убрать галочку `Изменять состав строк` чтобы пользователь не смог добавить новые строки
6. Переходим в свойства элемента `ПутьКФайлу` и на закладке `Использование` у свойства `Кнопка Выбора` ставим Да.
7. На вкладке `Команды` создаем команду и называем `ЗагрузитьФайл` и затем переносим на коммандную панель табличной части.
8. Создаем еще одну команду `ЗаписатьДанные` и так же переносим в табличную часть.
9. Переходим в свойства `ПутьКФайлу` и затем события и создаем событие `Начало выбора` на клиенте, далее вставляем код ниже.
```1c
&НаКлиенте
Процедура ПутьКФайлуНачалоВыбора(Элемент, ДанныеВыбора, СтандартнаяОбработка)
	Проводник = Новый ДиалогВыбораФайла(РежимДиалогаВыбораФайла.Открытие);
	Проводник.Заголовок = "Выберите файл";
	Фильтр = "Файл Excel (*.xls)|*.xls";
	Проводник.Фильтр = Фильтр;
	Оповещение = Новый ОписаниеОповещения("ПослеВыбораФайла", ЭтотОбъект);
	Проводник.Показать(Оповещение);
КонецПроцедуры
```
```1c
&НаКлиенте
Процедура ПослеВыбораФайла(ВыбранныеФайлы, ДополнительныеПараметры) Экспорт
	Если ВыбранныеФайлы = Неопределено Тогда 
		Возврат;
	КонецЕсли;
	Объект.ПутьКФайлу = ВыбранныеФайлы[0]
КонецПроцедуры
```
10. Далее правой кнопкой мыши по кнопке `ПрочитатьФайл` нажать на `<Действие команды>` и создать ее на клиенте, затем указать код ниже.
```1c
&НаКлиенте
Процедура ПрочитатьФайл(Команда)
	Объект.ДанныеФайла.Очистить();
	ПрочитатьФайл_XLS();
КонецПроцедуры
```
```1c
&НаСервере
Процедура ПрочитатьФайл_XLS_НаСервере()
	ТабДок = Новый ТабличныйДокумент;
	
	Попытка
		ТабДок.Прочитать(Объект.ПутьКФайлу, СпособЧтенияЗначенийТабличногоДокумента.Значение);
	Исключение
		Сообщение = Новый СообщениеПользователю;
		Сообщение.Текст = "Не удалось прочитать указанный файл по причине: " + ОписаниеОшибки();
		Сообщение.Сообщить();
		Возврат;
	КонецПопытки;
	
	КоличествоСтрок = ТабДок.ВысотаТаблицы;
	
	Для НомерСтроки = 2 По КоличествоСтрок Цикл 
		СтрокаДанных = Объект.ДанныеФайла.Добавить();
		СтрокаДанных.Наименование = ТабДок.ПолучитьОбласть("R" + Формат(НомерСтроки, "ЧГ=0") + "C" + 1).ТекущаяОбласть.Текст; // Данные строки необходимо подставить под ваши требования изменяя `СтрокаДанных.НазваниеПоляТабличной части и после "C" номер столбца`
		СтрокаДанных.Кодс = ТабДок.ПолучитьОбласть("R" + Формат(НомерСтроки, "ЧГ=0") + "C" + 2).ТекущаяОбласть.Текст;
    КонецЦикла;
КонецПроцедуры
```
```1c
&НаКлиенте
Процедура ПрочитатьФайл_XLS()
	ПрочитатьФайл_XLS_НаСервере();
КонецПроцедуры
```
11. Далее для загрузки прочитанных файлов необходимо создать у кнопки `ЗаписатьДаные` процедуру `На клиенте и сервере` и добавить код ниже.
```1c
&НаКлиенте
Процедура ЗаписатьДанные(Команда)
	ЗаписатьДанныеНаСервере();
КонецПроцедуры
```
```1c
&НаСервере
Процедура ЗаписатьДанныеНаСервере()
	Для каждого СтрокаДанных Из Объект.ДанныеФайла Цикл
		Повторение = Справочники.ТестовыйСправочник.НайтиПоРеквизиту("Кодс", Число(СтрокаДанных.Кодс)); // Число(СтрокаДанных.Кодс) - преобразует в число для точной проверки данных.
		Если Повторение <> Справочники.ТестовыйСправочник.ПустаяСсылка() Тогда
			Продолжить;
		КонецЕсли;
		// Код выше проверяет на уникальность данные, если в "Кодс" будет данные которые есть в справочнике то они не добавятся снова.
		НоваяЗапись = Справочники.ТестовыйСправочник.СоздатьЭлемент();
		НоваяЗапись.Наименование = СтрокаДанных.Наименование;
		НоваяЗапись.Кодс = СтрокаДанных.Кодс;
		
		НоваяЗапись.Записать();
	КонецЦикла;
	Сообщить("Добавление записей завершено!");
КонецПроцедуры
```
**Создание загрузки данных из файла Excel и выгрузку данных в справочник выполнено.**

# Пример вывода суммы прописью
```1c
 Сообщить(ЧислоПрописью(1234.56, "Л=ru_RU;ДП=Истина", "доллар,доллара,долларов,м,цент,цента,центов,м,2")); // 1234.56 заменить значением переменной
```

# Код подсчета итоговой суммы в табличном документе

*При измении количества*
```1c
&НаКлиенте
Процедура СписокТоваровКоличествоПриИзменении(Элемент)
	СтрокаТаблицы = Элементы.СписокТоваров.ТекущиеДанные;
	СтрокаТаблицы.Сумма = СтрокаТаблицы.Количество * СтрокаТаблицы.Цена
КонецПроцедуры
```
*При измении количества*
```1c
&НаКлиенте
Процедура СписокТоваровКоличествоПриИзменении(Элемент)
	СтрокаТаблицы = Элементы.СписокТоваров.ТекущиеДанные;
	СтрокаТаблицы.Сумма = СтрокаТаблицы.Количество * СтрокаТаблицы.Цена
КонецПроцедуры
```

*Для вывода итоговой суммы документа как реквизит необходимо выбрать итоговую функцию `Объект.СписокТоваров.ИтогСумма`, или вставить код ниже*
```1c
&НаКлиенте
Процедура ПередЗаписью(Отказ, ПараметрыЗаписи)
	Объект.Итого = 0;
	Для каждого СтрокаДокумента Из Объект.СписокТоваров Цикл
		Объект.Итого = Объект.Итого + (СтрокаДокумента.Количество * СтрокаДокумента.Цена);
	КонецЦикла;
КонецПроцедуры
```

# Код для вставки суммы со справочника в табличную часть документа
```1c
&НаКлиенте
Процедура СписокТоваровНаименованиеПриИзменении(Элемент)
	ТекДанные = Объект.СписокТоваров.НайтиПоИдентификатору(Элементы.СписокТоваров.ТекущаяСтрока);
	Если ЗначениеЗаполнено(ТекДанные.Наименование) Тогда
		ТекДанные.Цена=ПолучитьЦенуНаСервере(ТекДанные.Наименование);
	КонецЕсли;
КонецПроцедуры

&НаСервереБезКонтекста
Функция ПолучитьЦенуНаСервере(Номенклатура)
	Возврат Номенклатура.Цена
КонецФункции
```
