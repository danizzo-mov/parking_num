# parking_num

В процедуру __whsync.arrivedtare_spimportfromjson__ значение поля ```parking_num``` не подаётся на вход, при этом вставка в таблицу ```wh.tare``` делается при помощи выражения:
```
COALESCE(ods.route_id, ds.parking_num)
```
на строке 94, где: 
-  таблица ```ods``` - это таблица ```lgx.officedeliverysettings```, причём в этой таблице нет иного значения ```delivery_type```, кроме ```"KBT"```, что логично, т.к. на строке 132 в JOIN-е есть условие ```ON ... AND ods.delivery_type = 'KBT'```, т.е. другие записи были бы отброшены;
-  таблица ```ds``` - это ```lgx.deliverysetting``` (5.3kk записей).
Если тара, указанная на входе, уже была занесена в таблицу ```wh.taretracking```, то происходит обновление существующей записи, и обновляется, в том числе, поле ```parking_num``` (строки 225-236). Аналогично, если запись о таре уже существует в таблице ```wh.tare```, то эта запись обновляется (строки 249-277), а поле ```parking_num``` обновляется с помощью ```parking_num = COALESCE(t.parking_num, 0)``` (строка 268), где таблица t - это временная таблица, полученная на основе входных данных.

Процедура __whsync.office_importfromjson_v2__ делает обновление в таблице ```wh.tare```, при этом изменяет исключительно поле ```parking_num```. Оно приравнивается к ```0``` (строка 79)
```
SET parking_num = 0
```
для доставляемой (доставленной?) тары, запись о которой существует и удовлетворяет условиям (строки 85-88).

Процедура __whsync.officedeliverysettings_importfromjson__ обновляет существующую запись в таблице ```wh.tare``` (строки 89-147) и, в том числе, обновляет значение ```parking_num```, если оно не было заполнено ранее такими значениями: 
```
parking_num = COALESCE(ods.route_id, ds.parking_num, 0),
```
где таблица ```ods``` - это ```lgx.officedeliverysettings```, таблица ```ds``` - это ```lgx.deliverysetting```.

Процедура __whsync.ordershippingsetting_importfromjson__ достаёт из таблицы ```lgx.deliverysetting``` из существующей записи поля ```src_office_id```, ```dst_office_id``` и ```parking_num```, а саму запись удаляет (строки 93-100). Затем в удалённую запись о доставке добавляет данные из входного запроса, в том числе, поле ```parking_num```, и происходит вставка в таблицу ```lgx.deliverysetting``` (строки 103-152). Наконец, на основании вставки в таблицу ```lgx.deliverysetting``` (и из других таблиц) обновляется запись в таблице ```wh.tare``` для заданий, готовых к отгрузке: ... AND t2.ready_to_go IS TRUE ...
(строка 194). Поле ```parking_num``` обновляется как и ранее: 
```
parking_num = COALESCE(ods.route_id, ds.parking_num, 0) (строка 159)
```
где таблица ```ods``` - это ```lgx.officedeliverysettings```, таблица ```ds``` - это ```lgx.deliverysetting```.


Процедура __whsync.taregoodsdeclaration_importfromjson__ делает вставку в таблицу ```wh.tare``` (строки 570-709), и, помимо прочего, для поля ```parking_num``` проставляется значение по прежнему принципу (строка 618): 
```
COALESCE(ods.route_id, ds.parking_num, 0)
где таблица ```ods``` - это ```lgx.officedeliverysettings```, таблица ```ds``` - это ```lgx.deliverysetting```. Если запись о таре уже существует: ```(ON CONFLICT(tare_id, tare_type)``` (строка 674)),то значение поля ```parking_num``` проставляется следующим образом (строки 703-705):
```
parking_num = CASE
				WHEN t.dt > excluded.dt THEN t.parking_num
                ELSE excluded.parking_num END
```
где таблица ```t``` - это таблица ```wh.tare```, т.е. оставляется существовавшее значение.

Процедура __whsync.taretracking_importfromjson__ обновляет запись о таре в таблице ```wh.tare``` (строки 347-409), и, в том числе, проставляет значение 
поля ```parking_num``` следующим образом (строка 374):
```
parking_num = COALESCE(CASE WHEN t.sm_id = 17 THEN ods.route_id END, ds.parking_num, 0)
```
где таблица ```t``` - это таблица ```wh.tare```, таблица ```ods``` - это таблица ```lgx.officedeliverysettings```, таблица ```ds``` - это таблица ```lgx.deliverysetting```.

Процедура __whsync.transitroute_importfromjson__ обновляет запись в таблице ```wh.tare``` (строки 78-135), причём изменяется всего 3 поля таблицы``` wh.tare```: 
```next_office_id```, ```parking_num``` и ```delivery_type```. Значение поля ```parking_num``` проставляется прежним образом:
```
parking_num = COALESCE(ods.route_id, ds.parking_num, 0)
```
где таблица ```ods``` - это таблица ```lgx.officedeliverysettings```, таблица ```ds``` - это таблица ```lgx.deliverysetting```.
