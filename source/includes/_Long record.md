# Длинная запись

Длинная запись для авиа (airline addendum) — расширенная информация о маршрутной квитанции, которая передается вместе с транзакцией на обработку в платежную систему. Использование длинной записи позволяет сократить риски мошеннических операций и понизить стоимость обработки платежа.

Длинная запись состоит из информации о маршрутшной квитанции, информации о сегментах (перелетах) и информации о пассажирах.

Информация о маршрутной квитанции включает в себя:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
BookingRef |	String |	Обязательный, если не указан номер билета |	Номер брони
TicketNumber |	String |	Обязательный, если не указан номер брони |	Номер билета

Под сегментом понимается один авиаперелет: взлет и посадка. Необходимо указать все сегменты маршрута с перечнем следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
FlightNumber |	String |	Обязательный |	Номер рейса
DepartureDateTime |	DateTime |	Обязательный |	Дата и время отправления
ArrivalDateTime |	DateTime |	Обязательный |	Дата и время прибытия
OriginatingCountry |	String |	Обязательный |	Страна вылета на русском или английском языке
OriginatingCity |	String |	Обязательный |	Город вылета на русском или английском языке
OriginatingAirportCode |	String(3) |	Обязательный |	Код аэропорта вылета — 3 буквы по классификации IATA
DestinationCountry |	String |	Обязательный |	Страна прилета на русском или английском языке
DestinationCity |	String |	Обязательный |	Город прилета на русском или английском языке
DestinationAirportCode |	String(3) |	Обязательный |	Код аэропорта прилета — 3 буквы по классификации IATA

Для передчи информации о пассажирах, необходимо по каждому указать имя и фамилию латиницей:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
FirstName |	String |	Обязательный |	Имя пассажира
LastName |	String |	Обязательный |	Фамилия пассажира

Длинную запись можно передать в систему в параметре `AirlineAddendum` при вызове метода оплаты через [[API]]() или в ответе на [[запрос проверки платежа]]().

Пример формирования длинной записи:

```shell
{
   "TicketNumber":"390 5241025377",
   "BookingRef":null,
   "Legs":[
      {
         "FlightNumber":"A3 971",
         "DepartureDateTime":"2014-05-26T05:15:00",
         "ArrivalDateTime":"2014-05-26T07:30:00",
         "OriginatingCountry":"Россия",
         "OriginatingCity":"Москва",
         "OriginatingAirportCode":"DME",
         "DestinationCountry":"Греция",
         "DestinationCity":"Афины",
         "DestinationAirportCode":"ATH"
      },
      {
         "FlightNumber":"A3 204",
         "DepartureDateTime":"2014-05-26T09:45:00",
         "ArrivalDateTime":"2014-05-26T10:50:00",
         "OriginatingCountry":"Греция",
         "OriginatingCity":"Афины",
         "OriginatingAirportCode":"ATH",
         "DestinationCountry":"Греция",
         "DestinationCity":"Родос",
         "DestinationAirportCode":"RHO"
      },
      {
         "FlightNumber":"A3 980",
         "DepartureDateTime":"2014-06-06T09:00:00",
         "ArrivalDateTime":"2014-06-06T13:45:00",
         "OriginatingCountry":"Греция",
         "OriginatingCity":"Родос",
         "OriginatingAirportCode":"RHO",
         "DestinationCountry":"Россия",
         "DestinationCity":"Москва",
         "DestinationAirportCode":"DME"
      }
   ],
   "Passengers":[
      {
         "FirstName":"KONSTANTIN",
         "LastName":"IVANOV"
      },
      {
         "FirstName":"JULIA",
         "LastName":"IVANOVA"
      }
   ]
}
```
