# API
**API** — программный интерфейс системы для взаимодействия с системами ТСП.


Интерфейс работает по адресу https://api.cloudpayments.ru и поддерживает функции для выполнения платежа, отмены оплаты, возврата денег, завершения платежей, выполненных по двухстадийной схеме, создания и отмены подписок на рекуррентные платежи, а также отправки счетов по почте.
### Архитектура
API использует REST архитектуру. Параметры передаются методом **POST** в теле запроса в формате «ключ=значение» либо в JSON.


Выбор формата передачи параметров определяется на стороне клиента и управляется через заголовок запроса [Content-Type](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17):

* Для параметров «ключ=значение» Content-Type: application/x-www-form-urlencode.  
* Для параметров JSON Content-Type: application/json 

Ответ система выдает в JSON формате, который как минимум включает в себя два параметра: *Success* и *Message*:

```ruby
{ "Success": false, "Message": "Invalid Amount value" }
```
```python
{ "Success": false, "Message": "Invalid Amount value" }
```
```shell
{ "Success": false, "Message": "Invalid Amount value" }
```
```javascript
{ "Success": false, "Message": "Invalid Amount value" }
```


## Аутентификация запросов

Для аутентификации запроса используется [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication) — отправка логина и пароля в заголовке HTTP запроса. В качестве логина используется **Public ID**, в качестве пароля — **API Secret**. Оба этих значения можно получить в личном кабинете.


<aside class="notice"> Если в запросе не передан заголовок с данными аутентификации или переданы неверные данные, система вернет HTTP статус 401 – Unauthorized.</aside>

## Идемпотентность API
**Идемпотентность** — свойство API при повторном запросе выдавать тот же результат, что на первичный без повторной обработки. Это значит, что вы можете отправить несколько запросов к системе с одинаковым идентификатором, при этом обработан будет только один запрос, а все ответы будут идентичными. Таким образом реализуется защита от сетевых ошибок, которые приводят к созданию дублированных записей и действий.  
Для включения идемпотентности необходимо в запросе к API передавать заголовок с ключом *X-Request-ID*, содержащий уникальный идентификатор. Формирование идентификатора запроса остается на вашей стороне — это может быть guid, комбинация из номера заказа, даты и суммы или любое другое значение на ваше усмотрение.  
Каждый новый запрос, который необходимо обработать, должен включать новое значение *X-Request-ID*. Обработанный результат хранится в системе в течение 1 часа.  
Система сохраняет только успешные результаты запросов.


## Оплата по криптограмме


Для оплаты по криптограмме, сформированной скриптом Checkout, Apple Pay или Google Pay необходимо выполнить вызов метода по одному из следующих адресов:

* `POST https://api.cloudpayments.ru/payments/cards/charge` — для [одностадийного платежа](#8a2276f139)  
* `POSThttps://api.cloudpayments.ru/payments/cards/auth`  — для [двухстадийного](#8a2276f139)  

### Параметры запроса:


Параметр | Формат | Применение | Описание
--------- | ------- | ----------- | ----------- 
Amount | Numeric | Обязательный |  Сумма платежа 
Currency | String | Обязательный |  Валюта: RUB/USD/EUR/GBP (см. справочник)
IpAddress | String | Обязательный для всех платежей кроме Apple Pay и Google Pay |  IP адрес плательщика
Name | String | Обязательный |  Имя держателя карты в латинице
CardCryptogramPacket | String | Необязательный |  Криптограмма платежных данных
InvoiceId | String | Необязательный |  Номер счета или заказа
Description | String | Необязательный |  Описание оплаты в свободной форме
AccountId | String | Необязательный |  Идентификатор пользователя
Email | String | Необязательный |  E-mail плательщика на который будет отправлена квитанция об оплате
JsonData | Json | Необязательный |  Любые другие данные, которые будут связаны с транзакцией, в том числе инструкции для создания подписки или формирования онлайн-чека.Мы зарезервировали названия следующих параметров и отображаем их содержимое в реестре операций, выгружаемом в Личном Кабинете: name, firstName, middleName, lastName, nick, phone, address, comment, birthDate.

В ответ сервер возвращает JSON с тремя составляющими:  
`поле success — результат запроса,`  
`поле message — описание ошибки,`  
`объект model — расширенная информация.`  

</br>
</br>

**Возможные варианты ответа:**
</br>

* Некорректно сформирован запрос:  
		*success* — false  
		*message* — описание ошибки  
* Требуется 3-D Secure аутентификация (неприменимо для Apple Pay):  
		*success* — false  
		*model* — информация для проведения аутентификации  
* Транзакции отклонена:  
		*success* — false  
		*model* — информация о транзакции и код ошибки  
* Транзакции принята:  
		*success* — true  
		*model* — информация о транзакции  

</br>	
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

</br>
**Пример ответа:** *некорректный запрос:*

```shell
{"Success":false,"Message":"Amount is required"}
```

</br>
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

</br>

**Пример ответа:** *транзакция отклонена. В поле ReasonCode код ошибки (см. справочник):*

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

</br>

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

<aside class="warning">
Хранить криптограмму на сервере запрещено.
</aside>

##Обработка 3-D Secure
