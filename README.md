# IBEX API GUIDE 0.1.2

Документация находится в разработке.

## Содержание

1. [Общая информация](#общая-информация)
    - [Запросы](#запросы)
    - [Базовый URL сервера](#базовый-url-сервера)
    - [Авторизация](#авторизация)
    - [Ответы](#ответы)
2. [Работа с заказами](#работа-с-заказами)
    - [Статусы заказа](#статусы-заказа)
    - [Расчёт стоимости заказа](#расчёт-стоимости-заказа)
    - [Добавление заказа](#добавление-заказа)
    - [Получение информации о заказе](#получение-информации-о-заказе)
    - [Узнать состояние заказа](#узнать-состояние-заказа)
    - [Получение списка заказов](#получение-списка-заказов)
3. [Тарифы](#тарифы)
    - [Получение списка тарифов](#получение-списка-тарифов)
4. [Экономика](#экономика)
    - [Получение баланса](#получение-баланса)
5. [Карты](#карты)
    - [Геокодинг](#Геокодинг)
    - [Обратный геокодинг](#Обратный-Геокодинг)
    - [Поиск](#Поиск)
    - [Построение путей](#Построение-путей)


## Общая информация

REST API IBEX работает по протоколу HTTP и представляет собой набор методов для совершения запросов. Ответы сервера приходят в формате JSON.


### Запросы

Для взаимодействия с API используются POST или GET запросы на URL нужного метода с помощью добавления метода к базовому URL сервера.


Если в теле запроса передаются параметры, то они должны быть закодированы в одном из двух форматов: 
- `application/x-www-form-urlencoded` и с соответствующим заголовком: `Content-Type: application/x-www-form-urlencoded`
- `JSON` с заголовком `Content-Type: application/json`.

Пример запроса:

```
POST /ibex-0.1.7/order/calculate HTTP/1.1
Host: ibex24.ru
Authorization: Bearer kwA-NipcVyGAm2vTdiRmL30fDPjdONBq
Content-Type: application/x-www-form-urlencoded

time=15%3A04&tariff=2&porterAmount=2&paymentType=0
```


### Базовый URL сервера

https://ibex24.ru/ibex-0.1.7/ - тестовый сервер

https://ibex24.ru/ibex-1.1.7/ - боевой сервер


### Авторизация

Для использования запросов, требующих авторизации, необходимо получить Bearer токен у менеджера и добавить соответствующий http-заголовок Authorization к запросу, например: 

`Authorization: Bearer F1akwPiQT-qn_oJIvsksjxQNgVqJhTE0`


### Ответы

Сервер возвращает ответ со стандартным HTTP-кодом.

Поле | Описание
--- | ---
success | true в случае успешного запроса и false в противном случае.
data | Содержит в себе запрашиваемую информацию в случае, если запрос успешен, и данные об ошибке в противном случае. 


#### Пример ответа
```
{
    "success": true,
    "data": {
        "id": 3708,
        "status": 0,
        "price": 4307
    }
}
```


#### Ответ с ошибкой

В случае ответа с ошибкой поле data содержит поля:

Поле | Описание
--- | ---
status | Повторяет HTTP-код ответа
name | Описание HTTP-кода ответа
message | Описание ошибки



##### Пример ответа с ошибкой
```
{
    "success": false,
    "data": {
        "name": "Bad Request",
        "message": "укажите тариф",
        "status": 400
    }
}
```


---


## Работа с заказами


### Статусы заказа

Код статуса | Описание статуса
------------- | -----------------
-2 | Черновик
-1 | Не доступен
0, 8 | Ожидает распределения 
1 | Назначен исполнителю
2 | В процессе доставки
3, 4 | Завершён
5 | На погрузке
6 | На разгрузке
7 | Отменён
9 | Исполнитель на точке



### Расчёт стоимости заказа

POST `order/calculate`

#### Параметры запроса

Параметр | Описание  
---- | ----
points | Массив точек пути, каждая точка должна иметь следующие параметры:<ul><li>`lat` - широта точки, число от 0 до 180,</li><li>`lng` - долгота точки, число от 0 до 180,</li><li>`text` - текстовое описание адреса, строка не более 256 символов</li></ul>|
date | Дата заказа в формате ДД.ММ.ГГГГ, учитывается только вместе с полем `time`, _опциональный параметр_|
time | Время заказа в формате ЧЧ:ММ, учитывается только вместе с полем `date`, _опциональный параметр_|
isDateNow | Признак срочного заказа, 0 или 1. По умолчанию заказ считается срочным. Если одновременно указаны и `isDateNow`, и `date` и `time`, то `isDateNow` имеет больший приоритет, _опциональный параметр_.|
tariff | Идентификатор тарифа|
porterAmount | Число грузчиков, число 0, 1 или 2; 0 - по умолчанию, _опциональный параметр_|
carParams | Массив параметров транспорта, может иметь следующие поля: <ul><li>`is_isotherm` - признак изотермы, 0 или 1,</li><li>`is_refrigerator` - признак рефрижератора, 0 или 1</li></ul>_опциональный параметр_|


#### Поля ответа

Поле | Описание  
---- | ----
price | Стоимость заказа
token | Токен расчета стоимости
distance | Дистанция пути заказа, м
duration | Время пути заказа (без учета остановок), с
polyline | Кодированный оценочный маршрут заказа, алгоритм кодирования - https://developers.google.com/maps/documentation/utilities/polylinealgorithm, опциональное поле

#### Пример ответа

```
{
    "success": true,
    "data": {
        "price": 3225,
        "token": "8wp27jnYL7zWly2E2ZYPtYpxKsrnBvRT",
        "distance": 44414,
        "duration": 3248,
        "polyline": "}uxrImagcFgCGM?E@CBCBABCB?BABAFAFALG|@I|@AJAFABCDA@C@C@C?cEa@[ECAI?C?E?C?EBA@ABCDADCHERQ`BAHCJEJABCDA@C?C?C?EAq@OmFyAc@Ko@Qi@OSCQAKAM?u@@c@@IAGAQEy@[GCu@_@eB}@??g@eBaAkDiA}DIUKSKSkAyB_ByCKUGO{CgH]y@m@gB[m@U]QQeCaC??l@wFHw@bAoLd@iF@I@GDWBQDSNg@J[b@kAj@oAXk@`@u@rA}BjA_BDGPUHMJODIFMBIDOFQBSBO?M?O?IAKCMCKCKQq@]iAKc@AQg@oBQs@??Qq@_@aB??UcA[qAU}@Mm@ESw@}C??Eo@?M?C?E?E@C?E@C@GBEBGJWv@[PIHAHCHAJ?J?NBL@JBNFh@TV^DJDN@J@H?J?J?LCNENELQj@??qAz@qAt@}@f@gAh@iAf@oAf@eAV{DdA}GrBKB{@XgA^eA^i@NMFQHc@Pk@T??sB`AoAj@a@TSHsBrAcAr@}AlAg@`@ONsArAu@n@wB~BkDdEmI~LYb@}AbCaAxAc@f@c@`@c@n@wDzFS`@KPmCbE_AnA_AtAmCnDoB~CcAzAyAvBa@h@QRMNQTOVc@p@c@p@cBhCmBxCSXgA~Aa@l@g@r@]l@iGdJm@`AmDdFqBtCQ`@yAxBs@`AqBdCo@t@[^g@j@c@^u@jAgBdBsAjASNyAjAqA~@w@h@wAz@aClAoAf@UH]NSHg@VyUpJSH_FrBiAd@uH|CYL[LeA`@iCfAoAf@kDxAeAb@oHzC{@ZsBz@cA\\MFiCfAwH~Ck@Xk@VqH|CeGdC_A^uInDeDrAeDtAoFvB}NbG_C`A_A^qCjA}B~@}@\\_A^g@T}@\\}@\\oAd@gA\\A@cBf@eAXQFSFk@Lk@Lg@LuAXk@JuAXi@Hk@Hc@DeALkALiAJs@D{@FaBJeAD_ABaABgABuB?qEGw@C_DEiA@gCAk@Ag@CmJEe@Cc@A_@?u@?iDC}ROUAqCCcA@kB@y@CO?mDCmAAoAA[?w@A??CSCECCCECCCCECCAEAKGGEIKIOGOEMCMCMCOAO?Q?K@O@MBIBMDQFKFMFKFGHIHEHEHCF?F?F@DBFFLNJNHLFLTh@JVNR??NnAdAfHFf@BPHh@PxAFb@P~ARpBPzBB^F|@P`CDh@Bb@Bj@Db@@RBVDh@lAnIpBtMn@~Dx@bFL~@?@V~AVvAt@zDVvANv@Jd@bAnFdCdNFb@Fb@Fd@D`@Hz@H~@\\dEBNFl@@JHn@BTFf@Hp@H^Jf@Pp@Rt@Z~@b@pAb@jANZt@~AfBjDrAtCf@dAv@|Af@`AT`@FLV\\XZ^Zf@`@f@^PNVRj@d@p@t@^b@Z`@b@p@NVb@p@Tb@|@`Bv@|AlAnCP`@LXb@z@f@|@j@~@l@~@j@bARb@@DTj@N`@j@~A\\bAHVfAfDzAtEBJr@tB\\hAd@|ABHT|@Lp@^nBH\\NdAJv@B`@Dv@?^@\\Af@AZCf@E`@K~AQtBGp@AZCVGfACb@?f@Ax@?hA@`A@v@@ZFv@Fp@Fd@L`ATvAL~@Hb@Dh@B^@V@T@d@HtCDpAFvALxCD|@DfADz@F`AD^N`BFp@J|AFv@LdADd@Jp@Hh@Fb@PbAh@vC^rBTbBF\\Fb@DVTbB^nCFZPrARbBDXHl@DRHj@Hj@Hb@Nr@HZFVLf@Rp@Xv@FRbAvCbAvCxAdEHTdAfDx@hCDPRn@?FHp@@TDV@TFh@Bx@?@BVDdB@~@T~KF~EFlFH`G@t@@v@LjG`@pZ`@tTBz@?@B~AHbCF~@H~@Fj@Hr@H`@Jh@HZJ\\Nf@Rj@Zt@Xj@dAlB|AfCtBnD@@|AdCz@vAv@rATZT^d@v@PVVf@Xd@NXJRFRFPDRF^BPDTDd@BXDb@Dh@HpAFlAF`ADnADzABbAFpCBp@@r@DlAHzAPlCDp@Dl@B\\FbAN~B`@rGDj@VnEr@jK`C~\\BVp@pJHtBH`JFpEDnALhBPxBJlARpBLdBA`@?XAL?PAXC`@Gz@En@q@rJSbCKbCCb@E`@QxB_@fFC\\MzBEn@K`CIjBEjB@h@@ZJjBVtDx@fKFr@Dd@B^B^D`@^~EJhBBXXxBJ|@PfALx@H`@ZhBNx@TpAjAxGDR\\pBNx@VvAFZf@zC|A`JLp@^nBPt@VtALn@\\|AZdBLd@Pv@VlAL^Lj@\\bA^jAPh@Vx@X~@b@rAHZZz@Rl@f@pAx@tBh@tAb@hAXl@Pb@N\\d@`AJVRb@P`@JVNb@\\pA`@xATfANr@N~@f@pDn@fFj@nEn@hFHt@DXBRLx@Hf@BJ@NNnAp@~F\\pC\\vCNdBZfD@H^~E`@zHDtAJlEX`FL~ALjBd@vDb@~DFb@NrABP~@nG@DDZFXFTNx@f@`C|@bEBFNv@n@tC`@`BJVVrAn@lEb@|DBLNpBLpAF~@F|@NlBN|B@P@V@\\Dr@T`FHfDJjGBfABl@Bl@@\\@\\Bl@B^@Z@\\@P?B@NDlADlA?NTjI\\~KFtAn@hU@P?RPpG@~@@v@?j@@jAAh@AXCZI`AEr@MzAMfBQjCIbAIdAAZE`@g@vGI|AM`CE\\Cb@MhBIbAATM|AQfCOzBWvDGbAE`BGtAAPM`HCnACpAEtCAhAAf@Af@Az@EpCEzCAJCzAAvA?n@@t@@b@@NRbCLfAn@zFLpAVzBD`@Jr@\\zC|@`IL~@@NF\\Hd@DHJTJLLLpA~@d@Xf@XRL~@j@n@`@DBlAt@^\\XZT`@FLDLHVJd@\\hBf@zCDRTxARhA@HNdAJn@Hj@Hd@Hj@Hj@Jh@Hj@Ln@Jp@DZNz@TxAXnBPlBJtAFfAL|CHjBFlBNlCB^LjCLxABXBVXlCFf@NvAb@nDnAxLPhBLnALrA\\nDXnCZlCDVNbAt@tEV~AvAnIRnAJn@Fj@Bd@Dt@DdAVzI@f@@f@@r@LxD?DLjDNnEZnHRlBNnDDlA@\\@^@`@?Z?Z?pAAv@An@KfIAv@A~@C`A?jAE`DAvAM|LAbBEzDA`AAjB@t@BhFBxC@~CAtB?|@A~A?~D?L?jE?r@Al@AnCCbDAbA@|ABvF?fA@zCBfE@pA@z@Bf@FrA@`@FdAFp@JhA@L@JDb@D`@??DVHl@BTLbAR~Ad@|DhA`JP`Bb@lDlAhK\\pC\\rCFd@Fb@P`AJj@vA`Gb@dB\\zAfAnEj@~B\\nA\\vARt@Nb@FRHPN`@JTTd@R`@LTHPT^DFZh@@@Zh@r@jAZh@PZHJl@`AHL`@p@jApBZf@z@vAdAfBj@dAf@`A??LXNVLTJNJLHJDDFFPPXRHFNJ`CzARPJJJNJTN^DTBPBV@BBXB\\@T?X@P?j@Ad@CtBAVAb@C^Ch@Gx@AR?@Ab@Cd@?n@@bC?PFvD@r@BxB?DDjBHpC?FNvGF~B@rA@^A\\C\\Cd@Kr@Mp@Qz@g@`C_@`BADERCVAFANAVCjBGnBGjAEj@KbCCjACbB@z@@|DB`DBxDBxCBpH@rH?tKCjLFlE?~@?lD?XAvBM~JC|@E`EGjFA~@E`EEbE?J?f@KlFA~B?L?n@AzAEfDAtAG|EEjCMzLIjJOxMShN?v@O~L?\\Ad@CZAd@Ir@CXEVERERGPENGNIXS`@q@vAsArCkBxDmBbEaC|EMXaC~EoB~D[n@Wj@s@xA{@hBa@~@K^Mj@U`AOnACTE\\Et@I`ACh@]|EGx@Gp@APC`@Cp@??O?OEKGKGoDoDsGqG??|@sEx@eGNuAHgABw@?o@Ca@ASCUG[AEOc@Se@m@gAcAaBS[ACAECIAI?KAO@C?G@G@IjBcF??yBuCi@k@wC{CiAkAeAiAgCgCeBgB}@y@wCmCuAsAw@u@IM???]Bq@DYDUXaBh@oC^iB\\qAXcA??hHvD`@V??_@xCYzAu@nDADuAzG??|@x@dBfBfCfCdAhAhAjAvCzCh@j@xBtC??kBbFAHAF?FAB@N?J@HBH@D@BRZbA`Bl@fARd@Nb@@DFZBT@RB`@?n@Cv@IfAOtAy@dG}@rE??rGpGnDnDJFJFNDN???Bq@Ba@@QFq@Fy@\\}EBi@HaADu@D]BUNoATaALk@J_@`@_Az@iBr@yAVk@Zo@nB_E`C_FLY`C}ElBcEjByDrAsCp@wARa@HYFODOFQDSDSDWBYHs@@e@B[@e@?]??~@nADBD@D?DABCHCb@_@HEHARA`@L^DVCLELGRMHMDE@I"
    }
}
```


### Добавление заказа

POST `order/add`

#### Параметры запроса

Параметр | Описание  
---- | ----
points | Массив точек пути, каждая точка должна иметь следующие параметры:<ul><li>`lat` - широта точки, число от 0 до 180;</li><li>`lng` - долгота точки, число от 0 до 180;</li><li>`text` - текстовое описание адреса, строка не более 256 символов;</li><li>`contacts` - массив контактных лиц на точке заказа:<ul><li>`phone` - номер телефона контактного лица, 11 цифр;</li><li>`name` - имя, _опциональный параметр_.</li></ul></li></ul> |
date | Дата заказа в формате ДД.ММ.ГГГГ, учитывается только вместе с полем `time`, _опциональный параметр_|
time | Время заказа в формате ЧЧ:ММ, учитывается только вместе с полем `date`, _опциональный параметр_|
isDateNow | Признак срочного заказа, 0 или 1, опциональный параметр. По умолчанию заказ считается срочным. Если одновременно указаны и `isDateNow`, и `date` и `time`, то `isDateNow` имеет больший приоритет, _опциональный параметр_.|
tariff | Идентификатор тарифа.|
porterAmount | Число грузчиков, число 0, 1 или 2; 0 - по умолчанию, _опциональный параметр_.|
carParams | Массив параметров транспорта, может иметь следующие поля: <ul><li>`is_isotherm` - признак изотермы, 0 или 1,</li><li>`is_refrigerator` - признак рефрижератора, 0 или 1</li></ul>_опциональный параметр_|
token | Токен расчета стоимости, который вернул сервер при запросе `order/calculate`. Заказ будет создан по соответствующей токену стоимости при аналогичных данных расчета (в случае "срочного заказа" интервал между расчетом стоимости и созданием заказа не должен превышать 600 секунд). Если не указан, то цена будет рассчитана в момент создания заказа. _Опциональный параметр_.
paymentType | Способ оплаты: <ul><li>0 - наличные;</li><li>1 - банковская карта;</li><li>4 - счет в системе;</li></ul> обязателен, если не указан параметр `paymentMethodId`.
paymentMethodId | Идентификатор способа оплаты, обязателен, если не указан параметр `paymentType`.
comment | Комментарий к заказу, _опциональный параметр_.

#### Поля ответа

Поле | Описание  
---- | ----
id | Идентификатор созданного заказа
status | Статус
price | Стоимость

#### Пример ответа

```
{
    "success": true,
    "data": {
        "id": 3708,
        "status": 0,
        "price": 4307
    }
}
```

### Получение информации о заказе

GET `order/get?id=XXXX`

#### Параметры запроса

Параметр | Описание  
---- | ----
id | Идентификатор заказа

#### Поля ответа

Поле | Описание  
---- | ----
base | Основная информация о заказа 
price_data | Информация о стоимости
points | Массив точек заказа
contacts | Массив контактных лиц получателя
comment | Комментарий клиента к заказу, опциональное поле
driver | Данные водителя, опциональное поле
car_params | Данные параметров авто, указанные клиентом в заказе, опциональное поле

##### Описание поля `base`

Поле | Описание  
---- | ----
id | Идентификатор заказа
status | Статус
time_created | Время создания
time_order | Время заказа
address_from | Текстовый адрес начальной точки
address_from_lat | Широта начальной точки
address_from_lng | Долгота начальной точки
address_to | Текстовый адрес конечной точки
address_to_lat | Широта конечной точки
address_to_lng | Долгота конечной точки
tariff_id | Идентификатор тарифа
driver_id | Идентификатор исполнителя
payment_type | Способ оплаты
payment_method_id | Идентификатор способа оплаты
porter_amount | Число грузчиков
price | Стоимость

##### Описание поля `price_data`

Поле | Описание  
---- | ----
tariff_name | Название тарифа
total | Стоимость заказа

##### Описание элемента массива поля `points`

Поле | Описание  
---- | ----
id | Идентификатор точки
lat | Широта
lng | Долгота
text | Текстовый адрес

##### Описание элемента массива поля `contacts`

Поле | Описание  
---- | ----
contact | Контакт лица на точке: <ul><li>`name` - имя;</li><li>`phone` - телефон;</li></ul>
point_id | Идентификатор точки, к которой относится контакт

##### Описание поля `driver`

Поле | Описание  
---- | ----
id | Идентификатор исполнителя
name | Имя
phone | Номер телефона
license_plate | Номер авто
photo | Фото исполнителя, кодированное в base64, jpg/jpeg 100x100, опциональное поле

##### Описание поля `car_params`

Поле | Описание  
---- | ----
is_isotherm | Признак изотермы, 0 или 1
is_refrigerator | Признак рефрижератора, 0 или 1

#### Пример ответа

```
{
    "success": true,
    "data": {
        "base": {
            "id": 3329,
            "status": 2,
            "time_created": "2018-07-31 15:03:49",
            "address_from": "Москва, Варшавское шоссе, 28А",
            "address_from_lat": 55.68286,
            "address_from_lng": 37.6175119,
            "address_to": "Москва, улица Новый Арбат, 6",
            "address_to_lat": 55.75323,
            "address_to_lng": 37.59644,
            "time_order": "2018-07-31 15:03:49",
            "tariff_id": 1,
            "tariff_type": 0,
            "payment_type": 0,
            "payment_method_id": null,
            "price": 1911,
            "porter_amount": 2
        },
        "comment": "Велосипед и пара кресел",
        "price_data": {
            "tariff_name": "Small",
            "total": 1911
        },
        "car_params": {
            "is_refrigerator": 1,
            "is_isotherm": 0
        },
        "points": [
            {
                "lat": 55.68286,
                "lng": 37.6175119,
                "text": "Москва, Варшавское шоссе, 28А",
                "id": 153557
            },
            {
                "lat": 55.709488,
                "lng": 37.62941,
                "text": "Большой Староданиловский пер., 3с3, Москва, Россия, 115191",
                "id": 153558
            },
            {
                "lat": 55.75323,
                "lng": 37.59644,
                "text": "Москва, улица Новый Арбат, 6",
                "id": 153559
            }
        ],
        "driver": {
            "id": 333,
            "name": "Иван",
            "phone": "79123456789",
            "license_plate": "A123BC",
            "photo": "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"

        },
        "contacts": []
    }
}
```

### Узнать состояние заказа

GET `order/locate?id=XXXX`

#### Параметры запроса

Параметр | Описание  
---- | ----
id | Идентификатор заказа

#### Поля ответа

Поле | Описание  
---- | ----
driver | Локация исполнителя
order | Краткая информация о заказе

##### Описание поля `driver`

Поле | Описание  
---- | ----
lat | Широта координаты исполнителя
lng | Долгота координаты исполнителя
id | Идентификатор исполнителя

##### Описание поля `order`

Поле | Описание  
---- | ----
status | Статус заказа
price | Стоимость

#### Пример ответа

```
{
    "success": true,
    "data": {
        "driver": {
            "lat": 55.755833,
            "lng": 37.61777,
            "id": 0
        },
        "order": {
            "status": 2,
            "price": 3044
        }
    }
}
```

### Получение списка заказов

GET `order/get-all`

Возвращает массив заказов.

##### Описание элемента массива

Поле | Описание  
---- | ----
id | Идентификатор заказа
status | Статус
time_created | Время создания
time_order | Время заказа
address_from | Текстовый адрес начальной точки
address_from_lat | Широта начальной точки
address_from_lng | Долгота начальной точки
address_to | Текстовый адрес конечной точки
address_to_lat | Широта конечной точки
address_to_lng | Долгота конечной точки
tariff_id | Идентификатор тарифа
driver_id | Идентификатор исполнителя
payment_type | Способ оплаты
payment_method_id | Идентификатор способа оплаты
porter_amount | Число грузчиков
price | Стоимость

#### Пример ответа

```
{
    "success": true,
    "data": [
        {
            "id": 3329,
            "status": 1,
            "time_created": "2018-07-31 15:03:49",
            "address_from": "Москва, Варшавское шоссе, 28А",
            "address_from_lat": 55.682861,
            "address_from_lng": 37.617511,
            "address_to": "Москва, улица Новый Арбат, 6",
            "address_to_lat": 55.75323499,
            "address_to_lng": 37.5964430,
            "time_order": "2018-07-31 15:03:49",
            "customer_id": 352,
            "driver_id": 333,
            "tariff_id": 1,
            "payment_type": 0,
            "payment_method_id": null,
            "price": 1,
            "porter_amount": 2
        },
        {
            "id": 3470,
            "status": 8,
            "time_created": "2019-02-28 16:14:01",
            "address_from": "AAAA",
            "address_from_lat": 55.892445,
            "address_from_lng": 37.40228,
            "address_to": "CCCC",
            "address_to_lat": 55.7554360,
            "address_to_lng": 37.59141499,
            "time_order": "2019-02-28 16:14:00",
            "customer_id": 352,
            "driver_id": null,
            "tariff_id": 6,
            "payment_type": 0,
            "payment_method_id": null,
            "price": 5379,
            "porter_amount": 2
        }
    ]
}
```

## Тарифы

### Получение списка тарифов

GET `data/get-tariffs`

Возвращает массив тарифов.

##### Описание элемента массива

Поле | Описание  
---- | ----
id | Идентификатор тарифа
name | Название
description | Текстовое описание
length | Длина авто, м
width | Ширина, м
height | Высота, м
weight | Грузоподъёмность, т

#### Пример ответа

```
{
    "success": true,
    "data": [
        {
            "id": "1",
            "name": "Малый",
            "description": "Малый транспорт для перевозки нескольких коробок, велосипеда или иного не очень габаритного груза.",
            "length": "1.6",
            "width": "1.2",
            "height": "1.2",
            "weight": "0.4"
        },
        {
            "id": "2",
            "name": "Стандартный",
            "description": "Полноценный грузовик, которого хватит для перевозки небольшого дивана, комода или даже переезда.",
            "length": "2.7",
            "width": "1.7",
            "height": "1.7",
            "weight": "1.0"
        {
            "id": "3",
            "name": "Большой",
            "description": "Самый привычный и надёжный транспорт. Если у вас есть сомнения в том, хватит ли \"Стандартного\", смело заказывайте это авто.",
            "length": "3.5",
            "width": "2.1",
            "height": "2.1",
            "weight": "1.7"
        }
    ]
}
```

## Экономика

### Получение баланса

GET `payment/get-balance?type=XXXX`

#### Параметры запроса

Параметр | Описание  
---- | ----
type | main - для основного баланса, promo - для баланса подарочных баллов

Возвращает баланс в рублях.

#### Пример ответа

```
{
    "success": true,
    "data": 5100
}
```

## Карты

### Геокодинг

GET `maps/geocode?address=XXXX`

#### Параметры запроса

Параметр | Описание  
---- | ----
address | текстовый адрес

#### Поля ответа

Поле | Описание  
---- | ----
lat | Широта
lng | Долгота

#### Пример запроса

`https://ibex24.ru/ibex-1.1.8/maps/geocode?address=тверская+15`

#### Пример ответа

```
{
    "success": true,
    "data": {
        "lat": "58.44126955",
        "lng": "36.8208705494901"
    }
}
```

### Обратный геокодинг

GET `maps/reverse-geocode?lat=XXX&lng=YYY`

#### Параметры запроса

Параметр | Описание  
---- | ----
lat | Широта
lng | Долгота

Возвращает текстовый адрес точки.

#### Пример запроса

`https://ibex24.ru/ibex-1.1.8/maps/reverse-geocode?lat=55.555&lng=37.234`

#### Пример ответа

```
{
    "success": true,
    "data": "поселение Марушкинское, Новомосковский административный округ, Москва, ЦФО, 108809, Россия"
}
```

### Поиск

GET `maps/search?text=XXX&point=YYY`

#### Параметры запроса

Параметр | Описание  
---- | ----
text | Поисковой запрос
point | Широта и долгота точки, разделенные запятой. Относительно этой точки будут выданы результаты поиска. _Опциональный параметр_, по умолчанию - центр Москвы.

Возвращает массив поисковых результатов.

##### Описание элемента массива

Поле | Описание  
---- | ----
latlng | Координаты точки: <ul><li>`lat` - широта;</li><li>`lng` - долгота.</li></ul>
text | Текстовый адрес точки

#### Пример запроса

`https://ibex24.ru/ibex-1.1.8/maps/search?text=мастерк&point=57.473182,40.823245`

#### Пример ответа

```
{
    "success": true,
    "data": [
        {
            "latlng": {
                "lat": 57.473182,
                "lng": 40.823245
            },
            "text": "Мастерково, Костромская область, 157820"
        },
        {
            "latlng": {
                "lat": 55.708355,
                "lng": 37.6575264
            },
            "text": "улица Мастеркова, Даниловский район, Москва, 115280"
        },
        {
            "latlng": {
                "lat": 55.92407485,
                "lng": 37.888883119261
            },
            "text": "Мастерком, улица Гайдара, Королёв, Московская область, 141065"
        },
        {
            "latlng": {
                "lat": 55.7085403,
                "lng": 37.6585375
            },
            "text": "Горздрав, улица Мастеркова, 3, Даниловский район, Москва, 115280"
        }
    ]
}
```

### Построение путей

GET `maps/routes?points=XXX&time=YYY`

#### Параметры запроса

Параметр | Описание  
---- | ----
points | массив адресов, широта и долгота адресов разделяются запятыми, адреса - точками с запятой
time | время для построение маршрута в формате timestamp: ГГГГ-ММ-ДД чч:мм:сс, _опциональный параметр_

#### Поля ответа

Поле | Описание  
---- | ----
polyline | Кодированный путь
distance | Дистанция пути, м
duration | Время пути, с

#### Пример запроса

`https://ibex24.ru/ibex-1.1.8/maps/routes?points=55.9232527065549,37.9232527065549;55.8870321634492,37.96861220898438;55.98709609064341,37.11105617187502&time=2019-10-15+15:38:01`

#### Пример ответа

```
{
    "success": true,
    "data": {
        "points": null,
        "polyline": "eqitIg|mfF^cEzByXx@iKt@uIl@wIt@eJn@cJb@gGv@yK?OCSEY?IBMFILAZ@zDl@l@Dd@DTNd@\\XNtBr@p@^v@`@z@^p@P`ABL@RBhCjAxDjCl@^bGbEpCjBDBnBrAHFbBlAnAz@DBJHj@`@fErCl@VF@n@L|@FlAHfEZvBLnAJP@b@BRCTOLWLe@Dq@NgCdDsj@Ds@@OFiAxAeVZeFDq@FgAPyC`@qGHe@HUPA|EtALBZJB@`HnBbEfAvBl@TFHB~Cx@\\BRC\\MRWH]Fa@Hq@`@{G`AgP?KbAwNLyBtBn@p@h@b@~@Xt@d@xA|@~Ch@tBf@zB~@tFh@zBzExMFRHu@ReDN{Bp@iMd@}HHuAZSr@KtEqAzBm@d@NpAhAJNNrAJvADhBBrB?H?|@Al@KzAEZAh@ShBCd@OdBEf@?N@RRp@R\\Vr@Zb@Zd@RIn@kC~@_EZ}@~CyHpAcDlGcP\\{@sA}BBm@r@oBnIySxBsFaCqE??`CpEyBrFoIxSs@nBCl@rA|B]z@mGbPqAbD_DxH[|@_A~Do@jCSH[e@[c@Ws@S]Sq@AS?ODg@NeBBe@RiB@i@D[J{A@m@?}@?ICsBEiBKwAOsAKOqAiAe@O{Bl@uEpAs@J[RItAe@|Hq@hMOzBSdDIt@GS{EyMi@{B_AuFg@{Bi@uB}@_De@yAYu@c@_Aq@i@uBo@MxBcAvN?JaAfPa@zGIp@G`@I\\SV]LSB]C_Dy@ICUGwBm@cEgAaHoBCA[KMC}EuAQ@ITId@a@pGQxCGfAEp@[dFyAdVGhAANEr@eDrj@OfCEp@Md@MVUNSBc@CQAoAKwBMgE[mAI}@Go@MGAm@WgEsCQAKFINI\\Gb@c@`FANSlCk@pHAPCZEj@gC`[AXg@zFCX_AtKaAxLg@`GALmAlNYlDC^lCWN@LDFPD\\b@hVBbABpATnM@h@~@|`@B~BAf@GnBSpESnCe@bHo@dEYjBiAnHCNKr@[|Be@vCkAvGoBxLaBhKSbAQ~@kAfEkAbDe@tAeCbHYx@_@jAKd@]|A}@hFMfAw@~HIrCDbBP|CF~@RnBBNnCfLnB~HzAxF^vAtA`FTp@\\z@d@hBFj@@j@AvCAx@f@BhBLhAJz@HRBJ@hBRd@BT^JNDJJXJ\\Hd@D^OnGC~AGlCGhDCtAChAMnGATC~AInFEtBKxH?^@f@QjDKvFCtCAvAA~FIlBC`AGxAq@bNG|BDpABr@@v@FzI@`AH~LJ|@?H?nC?bA@fC?hBRbBn@`DXrAHZ\\bC\\hCBbCAbE?^?`CUxCcB|MEVa@zC}@zFGb@G^Gb@Ed@Cf@DnAFfAFvABd@HbBXhA|BrJThABN@L?N?TO`FE|AU~HYjIOtECl@AP?BUfHEv@Ad@ARChAWx@SlHAXCd@CPATCd@EfA?T?RAR@Rj@z_@NfJ?j@OrJ[jUCtAGdEA^Q|KAd@SzJApAAjDE~V?vABxAFtAJpAPpAPfAPdARdAPt@Pl@Tf@P\\TZRR\\VpAp@NJRPLPNVNZnBbFbArDz@fD`@bB`@fB\\bBXvAl@~Cp@hEp@pEj@hEj@dERvAT|AXdBZfBZ~A^hB`@lBVdAJ^^xAJd@xDvNj@zBx@jCf@xAt@hBz@bBbA`BbAvA`ArA|BbCdCpCbCfDrBfD`AzAfG|J~BvDdExGxCxEnMtSpElH|E|HnBzChGvJbHfL~BlE|@lBpBlE~BjFnAfDhKrZfBfF`BnErBdFpBrFTz@Lt@Fx@DjAJdF?n@Ap@Gl@Mr@a@jBsE|KqCdHiCfGwCfHwJbU{@tBkB~EoAnDwA~DUp@_C`H{@jCkAnDgAzDiAjE{@~DmAhGaA~Fw@|FYjC[pCi@xGe@`IYxGK|EEzAQ|LQfTAbAGdFAdCAd@IzHM~MQhXO|P?DG~G?\\[zVCxBClASfNWfLc@bQc@zMw@zTk@rMs@~Nu@bOw@xMw@vMs@~J_ApLaAtLgAnLkA~Ku@nH[xCIj@wArMyBrQaCzQqCzRgCpPiCtPmE`VmOhz@YzAg@nC]hBcCtMO|@iChNa@zBq@rDo@jEk@dEg@lEOxASjBc@rFMlBQfCYvFEt@QpFQhHItFAtE@nD@rBHhHNjGV~Gp@lMt@lOxGvqAZrGV`FnCbh@TtE?DdAhS^hHpAxUVrDZlD^pD^jDd@pD`@fCh@jDvAfI`BpHhBbH|BxH`BbFx@~B`K|YtUrr@`F`O~DvLTn@lEdNf@hCV`BHv@Dp@@p@?l@Cd@Ev@GnACb@MzAYxBUfBOj@Sj@o@fAc@d@wBvAc@^c@^a@h@c@r@]l@aArBYh@Ub@WXUVYR[L[FYB[?YA_@EmDS]A[?[@]FWDYPYZYb@Wf@Qd@YdA[pAiAxEoApFa@~AoAxEmA~DU|@Un@Uj@]x@_@t@gBfDe@`Am@lAg@fAm@pAiFlNqLtWwLfWwLjW_FnKwHlPoHlOaDdFiB~BaAdAy@z@kAfAwAfAkAz@aB~@gAf@mAh@wAd@gCr@sDz@}a@xJi^hIsL~Cs@NI@IBmXnGkFfBkFpCgErDiBjBaBxB_BdCeGlKsOvZqU|c@uCzFuA|CkBpEgBhF}@zCy@bDOr@cAfF}@|Fq@nFe@nEsCbYsBnTm@dGiB`RIv@sDj_@k@|G_@lFUdGG~E@`ELpERfEV|Ch@zEXpBp@jDjAxEhApDpAdDtDjHxEpJlAhDfAvDbAvEn@`Eh@|E\\vEN`EHvC@tEC`CEhCQ|DSvC]lD_@pCYhBe@bCy@rD{CvKaCbHoB~FeB`FaDpIgT~m@sDzKkDtJkEzMkBfGy@vCq@~BiDtM}AjHeBvIiB~J}BtM_ChLcCvJiCzJaCnHkC`IiHnQkI`QsK~RoBbDuXbd@qGpKyG|I}CbEqJzJkJtHeK|H_J`IoWfTaWvSkNbLaEfD{Ad@q@Pi@Bm@C_@Iy@e@gBaBi@c@a@Sa@Ia@Ce@Ba@Ha@Ti@b@a@n@Wt@U|@SvAMtAC|@@hAFx@TlC^rEVtC|@rK\\tEz@`KnAhNBX`LvuAnH|}@p@tIPtBv@rJf@hGf@tDhCz[TnBXhATt@L\\dCfGJVzBhFf@nAhAdCpAlCv@lBn@hBd@pADJFRJR@Fn@bBFNh@rAb@bAh@rATp@`@xA~CfP`@lBJh@Rp@~@lCnAvCtBbErBrDf@~@NXHL\\j@Vh@rD|HLZj@fAXj@DH`@x@P|@BT?r@EdAKhBkA|ViAjUAPEXQTaAZKPa@TKPC\\@XBPJHVJLJdB^`B`@p@NrBh@tAXh@Jh@Jb@HhKtBpDt@tIbBxBf@zCv@XHbCp@lBh@Xp@fAlHJj@Jv@h@rDBZD^Fd@Dp@H`CBpCBfADjDHhFDpC@nA`@j]BpA@x@Ft@NxA|AbOz@`I@PBP@Ln@bG\\fEDbCEzBC`@It@QdAOv@Sz@gB`HGTW`AI\\{@hDkBlHaBtGiAlECJEVc@bCo@lCAN?F?F@DBHZd@FLbCvDj@dAnTbl@DJ|AbE\\|@h@dBP|@Jl@Db@Dn@Dr@D~@?j@AfG?bBAbF?V?Z@rF?~E@jD?V?\\?lE?d@?P?d@?~B@pB?N?nB?hD@lO?z@@lC?hD?F?NFtAK@wBj@eEfA_Cl@eAVg@NODSDc@Hg@LqD`ASFSFk@N}GfBMDMDQDSFLh@Bd@?DPlDFbAFbAD|@Dz@d@`G^dERtB`@zDTzB@FBVPdBHt@L|@Jr@FVThADRlAdETdAHv@D\\n@bQFt@Dn@Hp@VnCH~@@bAB~B@~C?R?xC?n@?n@@fF?N@tJ?z@@bH?rA@bE?`@?pB?j@@lA@`@CvC?fA?n@?h@?vQ?l@AbAAx@I\\W`@KFODYNIAQ?}@\\y@f@mAv@Y^MZGb@Ct@AlD]FkAl@{CnAgBv@a@X?pA_@Dk@C{@Ga@q@g@AaB[}AIAb@",
        "distance": 79357,
        "duration": 5822,
        "waypoints": [
            {
                "lat": 55.923545,
                "lng": 37.923401
            },
            {
                "lat": 55.884591,
                "lng": 37.96434
            },
            {
                "lat": 55.986772,
                "lng": 37.111
            }
        ]
    }
}
```







