# API

**API** — программный интерфейс системы для взаимодействия с системами ТСП.

Интерфейс работает по адресу [api.cloudpayments.ru](https://api.cloudpayments.ru) и поддерживает функции для выполнения платежа, отмены оплаты, возврата денег, завершения платежей, выполненных по двухстадийной схеме, создания и отмены подписок на рекуррентные платежи, а также отправки счетов по почте.
  
## Архитектура

API использует REST архитектуру. Параметры передаются методом **POST** в теле запроса в формате «ключ=значение» либо в JSON.

Выбор формата передачи параметров определяется на стороне клиента и управляется через заголовок запроса [Content-Type](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17):

* Для параметров «ключ=значение» Content-Type: `application/x-www-form-urlencode`  
* Для параметров JSON Content-Type: `application/json`

Ответ система выдает в JSON формате, который как минимум включает в себя два параметра: *Success* и *Message*:

```shell
{ "Success": false, "Message": "Invalid Amount value" }
```

### Аутентификация запросов

Для аутентификации запроса используется [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication) — отправка логина и пароля в заголовке HTTP запроса. В качестве логина используется **Public ID**, в качестве пароля — **API Secret**. Оба этих значения можно получить в личном кабинете.

<aside class="notice"> Если в запросе не передан заголовок с данными аутентификации или переданы неверные данные, система вернет HTTP статус 401 – Unauthorized.</aside>

### Идемпотентность API

**Идемпотентность** — свойство API при повторном запросе выдавать тот же результат, что на первичный без повторной обработки. Это значит, что вы можете отправить несколько запросов к системе с одинаковым идентификатором, при этом обработан будет только один запрос, а все ответы будут идентичными. Таким образом реализуется защита от сетевых ошибок, которые приводят к созданию дублированных записей и действий.  
Для включения идемпотентности необходимо в запросе к API передавать заголовок с ключом *X-Request-ID*, содержащий уникальный идентификатор. Формирование идентификатора запроса остается на вашей стороне — это может быть guid, комбинация из номера заказа, даты и суммы или любое другое значение на ваше усмотрение.  
Каждый новый запрос, который необходимо обработать, должен включать новое значение *X-Request-ID*. Обработанный результат хранится в системе в течение 1 часа.  
Система сохраняет только успешные результаты запросов.

### Тестовый метод

Для проверки взаимодействия с API можно вызвать тестовый метод по адресу  
`https://api.cloudpayments.ru/test` без передачи параметров. В ответ метод возвращает статус запроса.

**Пример ответа:**

```shell
{"Success":true,"Message":"bd6353c3-0ed6-4a65-946f-083664bf8dbd"}
```

## Оплата по криптограмме

Для оплаты по криптограмме, сформированной скриптом [[Checkout]](), [[Apple Pay]]() или [[Google Pay]]() необходимо выполнить вызов метода по одному из следующих адресов:

* `POST https://api.cloudpayments.ru/payments/cards/charge` — для [[одностадийного платежа]]()  
* `POSThttps://api.cloudpayments.ru/payments/cards/auth`  — для [[двухстадийного]]()  

### Параметры запроса:


Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Amount | Numeric | Обязательный | Сумма платежа 
Currency | String | Обязательный | Валюта: RUB/USD/EUR/GBP ([[см. справочник]]())
IpAddress | String | Обязательный для всех платежей кроме Apple Pay и Google Pay |  IP адрес плательщика
Name | String | Обязательный | Имя держателя карты в латинице
CardCryptogramPacket | String | Необязательный | Криптограмма платежных данных
InvoiceId | String | Необязательный | Номер счета или заказа
Description | String | Необязательный | Описание оплаты в свободной форме
AccountId | String | Необязательный | Идентификатор пользователя
Email | String | Необязательный | E-mail плательщика на который будет отправлена квитанция об оплате
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для [[создания подписки]]() или формирования [[онлайн-чека]]().Мы зарезервировали названия следующих параметров и отображаем их содержимое в реестре операций, выгружаемом в Личном Кабинете: name, firstName, middleName, lastName, nick, phone, address, comment, birthDate.

В ответ сервер возвращает JSON с тремя составляющими:  
`поле success` — результат запроса, 
`поле message` — описание ошибки,  
`объект model` — расширенная информация. 

**Возможные варианты ответа:**

* Некорректно сформирован запрос:  
		`success` — false  
		`message` — описание ошибки  
* Требуется 3-D Secure аутентификация (неприменимо для Apple Pay):  
		`success` — false  
		`model` — информация для проведения аутентификации  
* Транзакции отклонена:  
		`success` — false  
		`model` — информация о транзакции и код ошибки  
* Транзакции принята:  
		`success` — true  
		`model` — информация о транзакции  
		
**Пример запроса на оплату по криптограмме:**

```shell
{
    "Amount":10,
    "Currency":"RUB",
    "InvoiceId":"1234567",
    "Description":"Оплата товаров в example.com",
    "AccountId":"user_x",
    "Name":"CARDHOLDER NAME",
    "CardCryptogramPacket":"01492500008719030128SMfLeYdKp5dSQVIiO5l6ZCJiPdel4uDjdFTTz1UnXY+3QaZcNOW8lmXg0H670MclS4lI+qLkujKF4pR5Ri+T/E04Ufq3t5ntMUVLuZ998DLm+OVHV7FxIGR7snckpg47A73v7/y88Q5dxxvVZtDVi0qCcJAiZrgKLyLCqypnMfhjsgCEPF6d4OMzkgNQiynZvKysI2q+xc9cL0+CMmQTUPytnxX52k9qLNZ55cnE8kuLvqSK+TOG7Fz03moGcVvbb9XTg1oTDL4pl9rgkG3XvvTJOwol3JDxL1i6x+VpaRxpLJg0Zd9/9xRJOBMGmwAxo8/xyvGuAj85sxLJL6fA=="
}
        
```

**Пример ответа:** *некорректный запрос:*

```shell
{"Success":false,"Message":"Amount is required"}
```

**Пример ответа:** *требуется 3-D Secure аутентификация:*

```shell
{
    "Model": {
        "TransactionId": 504,
        "PaReq": "eJxVUdtugkAQ/RXDe9mLgo0Z1nhpU9PQasWmPhLYAKksuEChfn13uVR9mGTO7MzZM2dg3qSn0Q+X\nRZIJxyAmNkZcBFmYiMgxDt7zw6MxZ+DFkvP1ngeV5AxcXhR+xEdJ6BhpEZnEYLBdfPAzg56JKSKT\nAhqgGpFB7IuSgR+cl5s3NqFTG2NAPYSUy82aETqeWPYUUAdB+ClnwSmrwtz/TbkoC0BtDYKsEqX8\nZfZkDGgAUMkTi8synyFU17V5N2nKCpBuAHRVs610VijCJgmZu17UXTxhFWP34l7evYPlegsHkO6A\n0C85o5hMsI3piNIZHc+IBaitg59qJYzgdrUOQK7/WNy+3FZAeSqV5cMqAwLe5JlQwpny8T8HdFW8\netFuBqUyahV+Hjf27vWCaSx22fe+KY6kXKZfJLK1x22TZkyUS8QiHaUGgDQN6s+H+tOq7O7kf8hd\nt30=",
        "AcsUrl": "https://test.paymentgate.ru/acs/auth/start.do"
    },
    "Success": false,
    "Message": null
}
```

**Пример ответа:** *транзакция отклонена. В поле ReasonCode код ошибки (см. [[справочник]]()):*

```shell
{
    "Model": {
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "PaymentAmount": 10.00000,
        "PaymentCurrency": "RUB",
        "PaymentCurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41", //все даты в UTC
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardExpDate": "05/19",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Declined",
        "StatusCode": 5,
        "Reason": "InsufficientFunds", // причина отказа
        "ReasonCode": 5051, //код отказа
        "CardHolderMessage":"Недостаточно средств на карте", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
    },
    "Success": false,
    "Message": null
}
```

**Пример ответа:** *транзакция принята:*

```shell
{
    "Model": {
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41", //все даты в UTC
        "AuthDate": "\/Date(1401733880523)\/",
        "AuthDateIso":"2014-08-09T11:49:42",
        "ConfirmDate": "\/Date(1401733880523)\/",
        "ConfirmDateIso":"2014-08-09T11:49:42",
        "AuthCode": "123456",
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardExpDate": "05/19",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Completed",
        "StatusCode": 3,
        "Reason": "Approved",
        "ReasonCode": 0,
        "CardHolderMessage":"Оплата успешно проведена", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
        "Token": "a4e67841-abb0-42de-a364-d1d8f9f4b3c0"
    },
    "Success": true,
    "Message": null
}
```

### Обработка 3-D Secure

Для проведения [[3-D Secure аутентификации]](), нужно отправить плательщика на адрес, указанный в параметре `AcsUrl` [[ответа сервера]]() с передачей следующих параметров:

* `MD` — параметр TransactionId из ответа сервера
* `PaReq` — одноименный параметр из ответа сервера
* `TermUrl` — адрес на вашем сайте для возврата плательщика после аутентификации

<aside class="notice">Регистр букв в названии параметров имеет значение.</aside>

Пример формы:

```shell
<form name="downloadForm" action="AcsUrl" method="POST">
    <input type="hidden" name="PaReq" value="eJxVUdtugkAQ/RXDe9mLgo0Z1nhpU9PQasWmPhLYAKksuEChfn13uVR9mGTO7MzZM2dg3qSn0Q+X\nRZIJxyAmNkZcBFmYiMgxDt7zw6MxZ+DFkvP1ngeV5AxcXhR+xEdJ6BhpEZnEYLBdfPAzg56JKSKT\nAhqgGpFB7IuSgR+cl5s3NqFTG2NAPYSUy82aETqeWPYUUAdB+ClnwSmrwtz/TbkoC0BtDYKsEqX8\nZfZkDGgAUMkTi8synyFU17V5N2nKCpBuAHRVs610VijCJgmZu17UXTxhFWP34l7evYPlegsHkO6A\n0C85o5hMsI3piNIZHc+IBaitg59qJYzgdrUOQK7/WNy+3FZAeSqV5cMqAwLe5JlQwpny8T8HdFW8\netFuBqUyahV+Hjf27vWCaSx22fe+KY6kXKZfJLK1x22TZkyUS8QiHaUGgDQN6s+H+tOq7O7kf8hd\nt30=">
    <input type="hidden" name="MD" value="504">
    <input type="hidden" name="TermUrl" value="https://example.com/post3ds?order=1234567">
</form>
<script>
    window.onload = submitForm;
    function submitForm() { downloadForm.submit(); }
</script>
```

После аутентификации, плательщик будет возвращен на `TermUrl` с параметрами `MD` и `PaRes`, переданными методом POST.

Для завершения оплаты необходимо вызвать метод по адресу  
`https://api.cloudpayments.ru/payments/cards/post3ds` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
TransactionId | Int | Значение параметра MD | Значение параметра MD
PaRes | String | Обязательный | Значение одноименного параметра

В ответ на корректно сформированный запрос, сервер вернет либо информацию об [[успешной транзакции]](), либо — [[об отклоненной]]().

## Оплата по токену (рекарринг)

Для оплаты по токену необходимо выполнить вызов метода по одному из следующих адресов:  

* `https://api.cloudpayments.ru/payments/tokens/charge` — для [[одностадийного платежа]]()  
или  
* `https://api.cloudpayments.ru/payments/tokens/auth` — для [[двухстадийного]]().

Параметры запроса:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Amount | Numeric | Обязательный | Сумма платежа
Currency | String | Обязательный | Валюта: RUB/USD/EUR/GBP (см. [[справочник]]())
AccountId | String | Обязательный | Идентификатор пользователя
Token | String | Обязательный | Токен
InvoiceId | String | Необязательный | Номер счета или заказа
Description | String | Необязательный | Назначение платежа в свободной форме
IpAddress | String | Необязательный | IP адрес плательщика
Email | String | Необязательный |  	E-mail плательщика на который будет отправлена квитанция об оплате
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для [[создания подписки]]() или формирования [[онлайн-чека]]().
Мы зарезервировали названия следующих параметров и отображаем их содержимое в реестре операций, выгружаемом в Личном Кабинете: name, firstName, middleName, lastName, nick, phone, address, comment, birthDate.

В ответ сервер возвращает JSON с тремя составляющими: поле success — результат запроса, поле message — описание ошибки, объект model — расширенная информация.

Возможные варианты:

* Некорректно сформирован запрос:
	`success` — false
	`message` — описание ошибки
* Транзакции отклонена:
	`success` — false
	`model` — информация о транзакции и код ошибки
* Транзакции принята:
	`success` — true
	`model` — информация о транзакции

**Пример запроса на оплату по токену**:

```shell
{
    "Amount":10,
    "Currency":"RUB",
    "InvoiceId":"1234567",
    "Description":"Оплата товаров в example.com",
    "AccountId":"user_x",
    "Token":"a4e67841-abb0-42de-a364-d1d8f9f4b3c0"
}
```

**Пример ответа:** *некорректный запрос*

```shell
{"Success":false,"Message":"Amount is required"}
```

**Пример ответа:** *транзакция отклонена*. В поле ReasonCode код ошибки (см. [[справочник]]())

```shell
{
    "Model": {
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41", //все даты в UTC
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Declined",
        "StatusCode": 5,
        "Reason": "InsufficientFunds", //причина отказа
        "ReasonCode": 5051,
        "CardHolderMessage":"Недостаточно средств на карте", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
    },
    "Success": false,
    "Message": null
}
```

**Пример ответа:** *транзакция принята*

```shell
{
    "Model": {
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41",  //все даты в UTC
        "AuthDate": "\/Date(1401733880523)\/",
        "AuthDateIso":"2014-08-09T11:49:42",
        "ConfirmDate": "\/Date(1401733880523)\/",
        "ConfirmDateIso":"2014-08-09T11:49:42",
        "AuthCode": "123456",
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Completed",
        "StatusCode": 3,
        "Reason": "Approved",
        "ReasonCode": 0,
        "CardHolderMessage":"Оплата успешно проведена", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
        "Token": "a4e67841-abb0-42de-a364-d1d8f9f4b3c0"
    },
    "Success": true,
    "Message": null
}
```

## Подтверждение оплаты

Для платежей, проведенных по [[двухстадийной схеме]](), необходимо подтверждение оплаты, которое можно выполнить через личный кабинет либо через вызов метода API.

Метод работает по адресу  
`https://api.cloudpayments.ru/payments/confirm` и принимает следующие параметры:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
TransactionId | Int | Обязательный | Номер транзакции в системе
Amount | Numeric | Обязательный | Сумма подтверждения в валюте транзакции
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для формирования [[онлайн-чека]]().

**Пример запроса:**

```shell
{"TransactionId":455,"Amount":65.98}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```

## Отмена оплаты

Отмену оплаты можно выполнить через личный кабинет либо через вызов метода API. Метод работает по адресу `https://api.cloudpayments.ru/payments/void` и принимает один параметр:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
TransactionId | Int |  	Обязательный | Номер транзакции в системе

**Пример запроса:**

```shell
{"TransactionId":455}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```

## Возврат денег

Возврат денег можно выполнить через личный кабинет либо через вызов метода API. Метод работает по адресу  
`https://api.cloudpayments.ru/payments/refund` и принимает следующие параметры:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
TransactionId | Int |  	Обязательный | Номер транзакции оплаты
Amount | Numeric | Обязательный | Сумма возврата в валюте транзакции
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для формирования [[онлайн-чека]]().

**Пример запроса:**

```shell
{"TransactionId":455, "Amount": 100}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```


## Выплата по криптограмме

Выплату по криптограмме можно осуществить через вызов метода API. Метод работает по адресу  
`https://api.cloudpayments.ru/payments/cards/topup` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Name | String | Обязательный | Имя держателя карты в латинице
CardCryptogramPacket | String | Обязательный | Криптограмма платежных данных
Amount | Numeric | Обязательный | Сумма платежа
AccountId | String | Обязательный | Идентификатор пользователя
Email | String | Необязательный | E-mail плательщика на который будет отправлена квитанция об оплате
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией. Мы зарезервировали названия следующих параметров и отображаем их содержимое в реестре операций, выгружаемом в Личном Кабинете: name, firstName, middleName, lastName, nick, phone, address, comment, birthDate.
Currency | String | Обязательный | Валюта: RUB
InvoiceId | String | Необязательный | Номер заказа в вашей системе
Description | String | Необязательный | Описание оплаты в свободной форме

**Пример запроса:**

```shell
{
    "Name":"CARDHOLDER NAME",
    "CardCryptogramPacket":"01492500008719030128SMfLeYdKp5dSQVIiO5l6ZCJiPdel4uDjdFTTz1UnXY+3QaZcNOW8lmXg0H670MclS4lI+qLkujKF4pR5Ri+T/E04Ufq3t5ntMUVLuZ998DLm+OVHV7FxIGR7snckpg47A73v7/y88Q5dxxvVZtDVi0qCcJAiZrgKLyLCqypnMfhjsgCEPF6d4OMzkgNQiynZvKysI2q+xc9cL0+CMmQTUPytnxX52k9qLNZ55cnE8kuLvqSK+TOG7Fz03moGcVvbb9XTg1oTDL4pl9rgkG3XvvTJOwol3JDxL1i6x+VpaRxpLJg0Zd9/9xRJOBMGmwAxo8/xyvGuAj85sxLJL6fA==",
    "Amount":1,
    "AccountId":"user@example.com",
    "Currency":"RUB"
    "InvoiceId":"1234567"
}
```

**Пример ответа:**

```shell
{
   "Model":{
      "PublicId":"pk_b9b86395c99782f0d16386d83e5d8",
      "TransactionId":100551,
      "Amount":1,
      "Currency":"RUB",
      "PaymentAmount":1,
      "PaymentCurrency":"RUB",
      "AccountId":"user@example.com",
      "Email":null,
      "Description":null,
      "JsonData":null,
      "CreatedDate":"/Date(1517943890884)/",
      "PayoutDate":"/Date(1517950800000)/",
      "PayoutDateIso":"2018-02-07T00:00:00",
      "PayoutAmount":1,
      "CreatedDateIso":"2018-02-06T19:04:50",
      "AuthDate":"/Date(1517943899268)/",
      "AuthDateIso":"2018-02-06T19:04:59",
      "ConfirmDate":"/Date(1517943899268)/",
      "ConfirmDateIso":"2018-02-06T19:04:59",
      "AuthCode":"031365",
      "TestMode":false,
      "Rrn":"568879820",
      "OriginalTransactionId":null,
      "IpAddress":"185.8.6.164",
      "IpCountry":"RU",
      "IpCity":"Москва",
      "IpRegion":null,
      "IpDistrict":"Москва",
      "IpLatitude":55.75222,
      "IpLongitude":37.61556,
      "CardFirstSix":"411111",
      "CardLastFour":"1111",
      "CardExpDate":"12/22",
      "CardType":"Visa",
      "CardTypeCode":0,
      "Status":"Completed",
      "StatusCode":3,
      "CultureName":"ru",
      "Reason":"Approved",
      "ReasonCode":0,
      "CardHolderMessage":"Оплата успешно проведена",
      "Type":2,
      "Refunded":false,
      "Name":"WQER",
      "SubscriptionId":null,
      "GatewayName":"Tinkoff Payout"
   },
   "InnerResult":null,
   "Success":true,
   "Message":null
}
```

##  Выплата по токену

Выплату по токену можно осуществить через вызов метода API. Метод работает по адресу  
`https://api.cloudpayments.ru/payments/token/topup` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Token | String | Обязательный |	Токен карты, выданный системой после первого платежа
Amount | Numeric | Обязательный | Сумма платежа
AccountId |	String | Обязательный | Идентификатор пользователя
Currency | String |	Обязательный | Валюта: RUB
InvoiceId |	String | Необязательный | Номер заказа в вашей системе

**Пример запроса:**

```shell
{
    "Token":"a4e67841-abb0-42de-a364-d1d8f9f4b3c0",
    "Amount":1,
    "AccountId":"user@example.com",
    "Currency":"RUB"
}
```

**Пример ответа:**

```shell
{
   "Model":{
      "PublicId":"pk_b9b86395c99782f0d16386d83e5d8",
      "TransactionId":100551,
      "Amount":1,
      "Currency":"RUB",
      "PaymentAmount":1,
      "PaymentCurrency":"RUB",
      "AccountId":"user@example.com",
      "Email":null,
      "Description":null,
      "JsonData":null,
      "CreatedDate":"/Date(1517943890884)/",
      "PayoutDate":"/Date(1517950800000)/",
      "PayoutDateIso":"2018-02-07T00:00:00",
      "PayoutAmount":1,
      "CreatedDateIso":"2018-02-06T19:04:50",
      "AuthDate":"/Date(1517943899268)/",
      "AuthDateIso":"2018-02-06T19:04:59",
      "ConfirmDate":"/Date(1517943899268)/",
      "ConfirmDateIso":"2018-02-06T19:04:59",
      "AuthCode":"031365",
      "TestMode":false,
      "Rrn":"568879820",
      "OriginalTransactionId":null,
      "IpAddress":"185.8.6.164",
      "IpCountry":"RU",
      "IpCity":"Москва",
      "IpRegion":null,
      "IpDistrict":"Москва",
      "IpLatitude":55.75222,
      "IpLongitude":37.61556,
      "CardFirstSix":"411111",
      "CardLastFour":"1111",
      "CardExpDate":"12/22",
      "CardType":"Visa",
      "CardTypeCode":0,
      "Status":"Completed",
      "StatusCode":3,
      "CultureName":"ru",
      "Reason":"Approved",
      "ReasonCode":0,
      "CardHolderMessage":"Оплата успешно проведена",
      "Type":2,
      "Refunded":false,
      "Name":"WQER",
      "SubscriptionId":null,
      "GatewayName":"Tinkoff Payout"
   },
   "InnerResult":null,
   "Success":true,
   "Message":null
}
```

### Просмотр информации об операции

Для просмотра информации о транзакции необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/payments/get` с указанием номера интересующей транзакции: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
TransactionId | Int | Обязательный | Номер транзакции

Если транзакция с указанным номером была найдена, система отобразит информацию о ней.

**Пример запроса:**

```shell
{"TransactionId":"504"}
```

**Пример ответа:**

```shell
{
    "Model": [{
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41", //все даты в указанной временной зоне
        "AuthDate": "\/Date(1401733880523)\/",
        "AuthDateIso":"2014-08-09T11:49:42",
        "ConfirmDate": "\/Date(1401733880523)\/",
        "ConfirmDateIso":"2014-08-09T11:49:42",
        "PayoutDate": "\/Date(1401733880523)\/", //дата возмещения
        "PayoutDateIso":"2014-08-09T11:49:42",
        "PayoutAmount": 99.61, //сумма возмещения
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardExpDate": "05/19",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Completed",
        "StatusCode": 3,
        "Reason": "Approved",
        "ReasonCode": 0,
        "CardHolderMessage":"Оплата успешно проведена", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
    }],
    "Success": true,
    "Message": null
}
```

## Проверка статуса платежа

Для поиска платежа и проверки статуса (см. [[справочник]]()) необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/payments/find` с указанием номера заказа, по которому была сформирована транзакция:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
InvoiceId | String | Обязательный | Номер заказа

Если платеж по указанному номеру заказа был найден, система отобразит либо информацию об [[успешной транзакции]](), либо — [[об отклоненной]](). Если будет найдено несколько платежей с указанным номером заказа, то система вернет информацию только о последней операции.

**Пример запроса:**

```shell
{"InvoiceId":"123456789"}
```

**Пример ответа:**

```shell
{"Success":false,"Message":"Not found"}
```

<aside class="notice">Проверка статуса платежа является избыточной и имеет смысл только в случае, если при проведении оплаты возник сбой, который привел к потере информации.</aside>

### Выгрузка списка транзакций

Для выгрузки списка транзакций за день необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/payments/list` с указанием даты по которой будут отбираться операции: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Date | Date | Обязательный | Дата | создания операций
TimeZone | String | Необязательный | Код временной зоны, по умолчанию — UTC

В выгрузку транзакций попадают все операции, зарегистрированные за указанный день. Для удобства учета вы можете указать код временной зоны ([[см. справочник]]())

**Пример запроса:**

```shell
{"Date":"2019-02-21", "TimeZone": "MSK"}
```

**Пример ответа:**

```shell
{
    "Model": [{
        "TransactionId": 504,
        "Amount": 10.00000,
        "Currency": "RUB",
        "CurrencyCode": 0,
        "InvoiceId": "1234567",
        "AccountId": "user_x",
        "Email": null,
        "Description": "Оплата товаров в example.com",
        "JsonData": null,
        "CreatedDate": "\/Date(1401718880000)\/",
        "CreatedDateIso":"2014-08-09T11:49:41", //все даты в указанной временной зоне
        "AuthDate": "\/Date(1401733880523)\/",
        "AuthDateIso":"2014-08-09T11:49:42",
        "ConfirmDate": "\/Date(1401733880523)\/",
        "ConfirmDateIso":"2014-08-09T11:49:42",
        "PayoutDate": "\/Date(1401733880523)\/", //дата возмещения
        "PayoutDateIso":"2014-08-09T11:49:42",
        "PayoutAmount": 99.61, //сумма возмещения
        "TestMode": true,
        "IpAddress": "195.91.194.13",
        "IpCountry": "RU",
        "IpCity": "Уфа",
        "IpRegion": "Республика Башкортостан",
        "IpDistrict": "Приволжский федеральный округ",
        "IpLatitude": 54.7355,
        "IpLongitude": 55.991982,
        "CardFirstSix": "411111",
        "CardLastFour": "1111",
        "CardExpDate": "05/19",
        "CardType": "Visa",
        "CardTypeCode": 0,
        "Issuer": "Sberbank of Russia",
        "IssuerBankCountry": "RU",
        "Status": "Completed",
        "StatusCode": 3,
        "Reason": "Approved",
        "ReasonCode": 0,
        "CardHolderMessage":"Оплата успешно проведена", //сообщение для покупателя
        "Name": "CARDHOLDER NAME",
    }],
    "Success": true,
}
```

## Выгрузка токенов

Для выгрузки списка платежных токенов необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/payments/tokens/list`

Пример ответ на запрос `https://api.cloudpayments.ru/payments/tokens/list`

```shell
{
    "Model": [
        {
            "Token": "tk_020a924486aa4df254331afa33f2a",
            "AccountId": "user_x",
            "CardMask": "4242 42****** 4242",
            "ExpirationDateMonth": 12,
            "ExpirationDateYear": 2020
        },
        {
            "Token": "tk_1a9f2f10253a30a7c5692a3fc4c17",
            "AccountId": "user_x",
            "CardMask": "5555 55****** 4444",
            "ExpirationDateMonth": 12,
            "ExpirationDateYear": 2021
        },
        {
            "Token": "tk_b91062f0f2875909233ab66d0fc7b",
            "AccountId": "user_x",
            "CardMask": "4012 88****** 1881",
            "ExpirationDateMonth": 12,
            "ExpirationDateYear": 2022
        }
    ],
    "InnerResult": null,
    "Success": true,
    "Message": null
}
```

### Создания подписки на рекуррентные платежи

Для подписки пользователя на рекуррентные платежи, необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/subscriptions/create` с передачей следующих параметров: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Token |	String | Обязательный |	Токен карты, выданный системой после первого платежа
AccountId |	String | Обязательный | Идентификатор пользователя
Description | String | Обязательный | Назначение платежа в свободной форме
Email |	String | Обязательный |	E-mail плательщика
Amount | Numeric | Обязательный | Сумма платежа
Currency | String |	Обязательный | Валюта: RUB/USD/EUR/GBP (см. [[[справочник]]())
RequireConfirmation | Bool | Обязательный | Если значение true — платежи будут выполняться по двустадийной схеме
StartDate |	DateTime | Обязательный | Дата и время первого платежа по плану во временной зоне UTC
Interval | String |	Обязательный | Интервал. Возможные значения: Day, Week, Month
Period | Int | Обязательный | Период. В комбинации с интервалом, 1 Month значит раз в месяц, а 2 Week — раз в две недели.
MaxPeriods | Int | Необязательный |	Максимальное количество платежей в подписке

В ответ на корректно сформированный запрос система возвращает сообщение об успешно выполненной операции и идентификатор подписки.

**Пример запроса:**

```shell
{  
   "token":"477BBA133C182267FE5F086924ABDC5DB71F77BFC27F01F2843F2CDC69D89F05",
   "accountId":"user@example.com",
   "description":"Ежемесячная подписка на сервис example.com",
   "email":"user@example.com",
   "amount":1.02,
   "currency":"RUB",
   "requireConfirmation":false,
   "startDate":"2014-08-06T16:46:29.5377246Z",
   "interval":"Month",
   "period":1
}
```

**Пример ответа:**

```shell
{
   "Model":{
      "Id":"sc_8cf8a9338fb8ebf7202b08d09c938", //идентификатор подписки
      "AccountId":"user@example.com",
      "Description":"Ежемесячная подписка на сервис example.com",
      "Email":"user@example.com",
      "Amount":1.02,
      "CurrencyCode":0,
      "Currency":"RUB",
      "RequireConfirmation":false, //true для двустадийных платежей
      "StartDate":"\/Date(1407343589537)\/",
      "StartDateIso":"2014-08-09T11:49:41", //все даты в UTC
      "IntervalCode":1,
      "Interval":"Month",
      "Period":1,
      "MaxPeriods":null,
      "StatusCode":0,
      "Status":"Active",
      "SuccessfulTransactionsNumber":0,
      "FailedTransactionsNumber":0,
      "LastTransactionDate":null,
      "LastTransactionDateIso":null, 
      "NextTransactionDate":"\/Date(1407343589537)\/"
      "NextTransactionDateIso":"2014-08-09T11:49:41"
   },
   "Success":true
}
```

## Запрос информации о подписке

Для получения информации о статусе подписки, необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/subscriptions/get` с передачей единственного параметра: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Id | String | Обязательный | Идентификатор подписки

**Пример запроса:**

```shell
{"Id":"sc_8cf8a9338fb8ebf7202b08d09c938"}
```

**Пример ответа:**

```shell
{
   "Model":{
      "Id":"sc_8cf8a9338fb8ebf7202b08d09c938", //идентификатор подписки
      "AccountId":"user@example.com",
      "Description":"Ежемесячная подписка на сервис example.com",
      "Email":"user@example.com",
      "Amount":1.02,
      "CurrencyCode":0,
      "Currency":"RUB",
      "RequireConfirmation":false, //true для двустадийных платежей
      "StartDate":"\/Date(1407343589537)\/",
      "StartDateIso":"2014-08-09T11:49:41", //все даты в UTC
      "IntervalCode":1,
      "Interval":"Month",
      "Period":1,
      "MaxPeriods":null,
      "StatusCode":0,
      "Status":"Active",
      "SuccessfulTransactionsNumber":0,
      "FailedTransactionsNumber":0,
      "LastTransactionDate":null,
      "LastTransactionDateIso":null, 
      "NextTransactionDate":"\/Date(1407343589537)\/"
      "NextTransactionDateIso":"2014-08-09T11:49:41"
   },
   "Success":true
}
```

## Поиск подписок

Для получения списка подписок для определенного аккаунта, нужно сделать запрос на адрес  
`https://api.cloudpayments.ru/subscriptions/find` с передачей единственного параметра: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
accountId | String | Обязательный | Идентификатор подписки

**Пример запроса:**

```shell
{"accountId":"user@example.com"}
```

**Пример ответа:**

```shell
{
  "Model": [
    {
      "Id": "sc_4bae8f5823bb8cdc966ccd1590a3b",
      "AccountId": "user@example.com",
      "Description": "Подписка на сервис",
      "Email": "user@example.com",
      "Amount": 1.02,
      "CurrencyCode": 0,
      "Currency": "RUB",
      "RequireConfirmation": false,
      "StartDate": "/Date(1473665268000)/",
      "StartDateIso": "2016-09-12T15:27:48",
      "IntervalCode": 1,
      "Interval": "Month",
      "Period": 1,
      "MaxPeriods": null,
      "CultureName": "ru",
      "StatusCode": 0,
      "Status": "Active",
      "SuccessfulTransactionsNumber": 0,
      "FailedTransactionsNumber": 0,
      "LastTransactionDate": null,
      "LastTransactionDateIso": null,
      "NextTransactionDate": "/Date(1473665268000)/",
      "NextTransactionDateIso": "2016-09-12T15:27:48"
    },
    {
      "Id": "sc_b4bdedba0e2bdf279be2e0bab9c99",
      "AccountId": "user@example.com",
      "Description": "Подписка на сервис",
      "Email": "user@example.com",
      "Amount": 3.04,
      "CurrencyCode": 0,
      "Currency": "RUB",
      "RequireConfirmation": false,
      "StartDate": "/Date(1473665268000)/",
      "StartDateIso": "2016-09-12T15:27:48",
      "IntervalCode": 0,
      "Interval": "Week",
      "Period": 2,
      "MaxPeriods": null,
      "CultureName": "ru",
      "StatusCode": 0,
      "Status": "Active",
      "SuccessfulTransactionsNumber": 0,
      "FailedTransactionsNumber": 0,
      "LastTransactionDate": null,
      "LastTransactionDateIso": null,
      "NextTransactionDate": "/Date(1473665268000)/",
      "NextTransactionDateIso": "2016-09-12T15:27:48"
    }
  ],
  "InnerResult": null,
  "Success": true,
  "Message": null
}
```

## Изменение подписки на рекуррентные платежи

Для изменения ранее созданной подписки, необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/subscriptions/update` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Id | String | Обязательный | Идентификатор подписки
Description | String | Необязательный | Для изменения назначение платежа
Amount | Numeric | Необязательный |	Для изменения суммы платежа
Currency | String |	Необязательный | Для изменения валюты: RUB/USD/EUR/GBP (см. справочник)
RequireConfirmation | Bool | Необязательный | Для изменения схемы проведения платежей
StartDate |	DateTime | Необязательный |	Для изменения даты и времени первого (или следующего) платежа во временной зоне UTC
Interval | String |	Необязательный | Для изменения интервала. Возможные значения: Week, Month
Period | Int | Необязательный |	Для изменения периода. В комбинации с интервалом, 1 Month значит раз в месяц, а 2 Week — раз в две недели.
MaxPeriods | Int | Необязательный |	Для изменения максимального количества платежей в подписке

В ответ на корректно сформированный запрос система возвращает сообщение об успешно выполненной операции и параметры подписки.

**Пример запроса:**

```shell
{  
   "Id":"sc_8cf8a9338fb8ebf7202b08d09c938",
   "description":"Тариф №5",
   "amount":1200,
   "currency":"RUB"
}
```

**Пример ответа:**

```shell
{
   "Model":{
      "Id":"sc_8cf8a9338fb8ebf7202b08d09c938", //идентификатор подписки
      "AccountId":"user@example.com",
      "Description":"Тариф №5",
      "Email":"user@example.com",
      "Amount":1200,
      "CurrencyCode":0,
      "Currency":"RUB",
      "RequireConfirmation":false, //true для двустадийных платежей
      "StartDate":"\/Date(1407343589537)\/",
      "StartDateIso":"2014-08-09T11:49:41", //все даты в UTC
      "IntervalCode":1,
      "Interval":"Month",
      "Period":1,
      "MaxPeriods":null,
      "StatusCode":0,
      "Status":"Active",
      "SuccessfulTransactionsNumber":0,
      "FailedTransactionsNumber":0,
      "LastTransactionDate":null,
      "LastTransactionDateIso":null, 
      "NextTransactionDate":"\/Date(1407343589537)\/"
      "NextTransactionDateIso":"2014-08-09T11:49:41"
   },
   "Success":true
}
```

## Отмена подписки на рекуррентные платежи

Для отмены подписки на рекуррентные платежи, необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/subscriptions/cancel` с передачей следующих параметров: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Id | String | Обязательный | Идентификатор подписки

В ответ на корректно сформированный запрос система возвращает сообщение об успешно выполненной операции.

**Пример запроса:**

```shell
{"Id":"sc_cc673fdc50b3577e60eee9081e440"}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```

Вы также можете предоставить покупателю ссылку на сайт системы — `https://my.cloudpayments.ru/unsubscribe`, где он самостоятельно сможет найти и отменить свои регулярные платежи. 

## Создание счета для отправки по почте

Для формирования ссылки на оплату и последующей отправки уведомления на e-mail адрес плательщика, можно воспользоваться методом  
`https://api.cloudpayments.ru/orders/create` с передачей следующих параметров: 

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Amount | Numeric | Обязательный | Сумма платежа
Currency | String |	Обязательный | Валюта: RUB/USD/EUR/GBP (см. [[справочник]]())
Description | String | Обязательный | Назначение платежа в свободной форме
Email |	String | Необязательный | E-mail плательщика
RequireConfirmation | Bool | Необязательный | Есть значение true — платеж будет выполнен по двустадийной схеме
SendEmail | Bool | Необязательный |	Если значение true — плательщик получит письмо со ссылкой на оплату
InvoiceId |	String | Необязательный | Номер заказа в вашей системе
AccountId |	String | Необязательный | Идентификатор пользователя в вашей системе
OfferUri | String | Необязательный | Ссылка на оферту, которая будет показываться на странице заказа
Phone |	String | Необязательный | Номер телефона плательщика в произвольном формате
SendSms | Bool | Необязательный | Если значение true — плательщик получит СМС со ссылкой на оплату
SendViber | Bool | Необязательный |	Если значение true — плательщик получит сообщение в Viber со ссылкой на оплату
CultureName | String | Необязательный |	Язык уведомлений. Возможные значения: "ru-RU", "en-US"
SubscriptionBehavior | String |	Необязательный | Для создания платежа с подпиской. Возможножные значения: CreateWeekly, CreateMonthly
JsonData | Json | Необязательный | Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для формирования [[онлайн-чека]](). 

<aside class="notice">В ответ на корректно сформированный запрос система возвращает параметры запроса и ссылку на оплату.</aside>

**Пример запроса:**

```shell
{
    "Amount":10.0,
    "Currency":"RUB",
    "Description":"Оплата на сайте example.com",
    "Email":"client@test.local",
    "RequireConfirmation":true,
    "SendEmail":false
}
```

**Пример ответа:**

```shell
{
    "Model":{
        "Id":"f2K8LV6reGE9WBFn",
        "Number":61,
        "Amount":10.0,
        "Currency":"RUB",
        "CurrencyCode":0,
        "Email":"client@test.local",
        "Description":"Оплата на сайте example.com",
        "RequireConfirmation":true,
        "Url":"https://orders.cloudpayments.ru/d/f2K8LV6reGE9WBFn",
    },
    "Success":true,
}
```

<aside class="notice"> Сообщение на телефон плательщика может быть отправлено только одним выбранным способом: СМС или Viber.</aside>

## Отмена созданного счета

Для отмены созданного счета необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/orders/cancel` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Id | String | Обязательный | Идентификатор счета

В ответ на корректно сформированный запрос система возвращает сообщение об успешно выполненной операции.

**Пример запроса:**

```shell
{"Id":"f2K8LV6reGE9WBFn"}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```

## Просмотр настроек уведомлений

Для просмотра настроек уведомлений необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/site/notifications/{Type}/get` с указанием типа уведомления:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Type | String | Обязательный | Тип уведомления: Check/Pay/Fail и т.д. (см. [[справочник]]())

Пример ответа на запрос для Pay уведомления на адрес  
`https://api.cloudpayments.ru/site/notifications/pay/get:`

```shell
{
    "Model": {
        "IsEnabled": true,
        "Address": "http://example.com",
        "HttpMethod": "GET",
        "Encoding": "UTF8",
        "Format": "CloudPayments"
    },
    "Success": true,
    "Message": null
}
```

## Изменение настроек уведомлений

Для изменения настроек уведомлений необходимо отправить запрос на адрес  
`https://api.cloudpayments.ru/site/notifications/{Type}/update` с передачей следующих параметров:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Type | String | Обязательный | Тип уведомления: Check/Pay/Fail и т.д. (см. [[справочник]]())
IsEnabled |	Bool |	Необязательный | Если значение true — то уведомление включено. Значение по умолчанию — false.
Address | String |	Необязательный, если `IsEnabled=false`, в противном случае обязательный |	Адрес для отправки уведомлений (для HTTPS схемы необходим валидный SSL сертификат).
HttpMethod | String | Необязательный | HTTP метод для отправки уведомлений. Возможные значения: GET, POST. Значение по умолчанию — GET.
Encoding | String | Необязательный | Кодировка уведомлений. Возможные значения: UTF8, Windows1251. Значение по умолчанию — UTF8.
Format | String | Необязательный | Формат уведомлений. Возможные значения: CloudPayments, QIWI, RT. Значение по умолчанию — CloudPayments.

**Пример запроса** для [[Pay]]() уведомления на адрес `https://api.cloudpayments.ru/site/notifications/pay/update:`

```shell
{
    "IsEnabled": true,
    "Address": "http://example.com",
    "HttpMethod": "GET",
    "Encoding": "UTF8",
    "Format": "CloudPayments"
}
```

**Пример ответа:**

```shell
{"Success":true,"Message":null}
```

## Запуск сессии для оплаты через Apple Pay

Запуск сессии необходим для приема платежей [[Apple Pay]]() на сайтах. Для оплаты в мобильных приложениях его использование не требуется.  
Метод работает по адресу  
`https://api.cloudpayments.ru/applepay/startsession` и принимает следующие параметры:

Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
ValidationUrl |	Строка | Обязательный | Адрес, полученный из Apple JS

В ответ на корректно сформированный запрос система возвращает ответ, где в объекте Model содержится сессия для оплаты Apple Pay в формате JSON.

**Пример запроса:**

```shell
{"ValidationUrl":"https://apple-pay-gateway.apple.com/paymentservices/startSession"}
```

**Пример ответа:**

```shell
{"Success":true, Model:{SomeAppleStuff:1}}
```

## Локализация

По умолчанию API выдает сообщения для пользователей на русском языке. Для получения ответов, локализованных для других языков, передайте в параметрах запроса `CultureName.`

**Список поддерживаемых языков:**

Язык | Часовой пояс  | Код
--------- | ------- | ----------- 
Русский | MSK | ru-RU
Английский | CET | en-US
Латышский |	CET | lv
Азербайджанский | AZT |	az
Русский | ALMT | kk
Украинский | EET | uk
Польский | CET | pl