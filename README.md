# parking_num

Процедура ```rpt.taredeliverytask_recalcbyoffice(office_id, delay_ss)``` делает вставку в таблицу ```rpt.taredeliverytask``` и вставляет, в том числе, поле ```parking_num```. Если значения нет, то проставляется значение ```0```. Эта функция вызывается, например, процедурой ```wh.parking_getbyoffice```. Но значение проставляется на основании таблицы ```wh.tare```, поэтому если  произойдут изменения в ```wh.tare```, то поменяется и запись в таблице ```rpt.taredeliverytask```.

Триггер ```wh.taregupd()``` делает вставку в таблицу ```rpt.tarenextstat``` и вставляет, в том числе, поле ```parking_num```. Однако вставка происходит не из таблицы, а из некого хранилища ```"NEW"```. Например:
```
INSERT INTO rpt.tarenextstat AS t(office_id, parking_num, ...)
SELECT NEW.current_office_id,
	   NEW.parking_num, ...
```
Надо разобраться, что такое ```NEW```, т.к. в такой вставку нет ```FROM```. Возможно, это поле - это какая-то особенность триггеров, т.к. дальше фигурирует ```OLD```. Затем вставка (или обновление) делается в таблицу ```rpt.tarenextstatcourier```, и вставляется, в том числе, поле ```parking_num```.

В процедуре ```whsync.arrivedtare_spimportfromjson``` значение поля ```parking_num``` не подаётся на вход, а вставляется при помощи
```COALESCE(ods.route_id, ds.parking_num)``` на строке 94, где
-  таблица ```ods``` - это ```lgx.officedeliverysettings```, причём в этой таблице нет иного значения ```delivery_type```, кроме ```"KBT"```, и всего 148 записей;
-  таблица ```ds``` - это ```lgx.deliverysetting``` (5.3kk записей).
