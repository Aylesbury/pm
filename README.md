# Демонстрационный экзамен
**Введение**
* [Загрузка данных excel в справочник](#Загрузка-данных-excel-в-справочник)


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
		СтрокаДанных.Код = ТабДок.ПолучитьОбласть("R" + Формат(НомерСтроки, "ЧГ=0") + "C" + 2).ТекущаяОбласть.Текст;
    КонецЦикла;
КонецПроцедуры
```
```1c
&НаКлиенте
Процедура ПрочитатьФайл_XLS()
	ПрочитатьФайл_XLS_НаСервере();
КонецПроцедуры
```
