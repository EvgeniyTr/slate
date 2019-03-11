# Apple Pay 
 
![image1](images/image1.png)

Apple Pay — современный, удобный и безопасный способ оплаты от компании Apple. Покупатель один раз привязывает карту к телефону, а далее при оплате только подтверждает платеж отпечатком пальца. Технология работает в мобильных приложениях и браузере Safari на iPhone, iPad, Apple Watch и последних MacBook.

Apple Pay работает с картами Visa и MasterCard и доступен всем организациям и индивидуальным предпринимателям, подключенным к системе CloudPayments без дополнительных соглашений и без изменений в условиях работы.

Протестировать оплату через Apple Pay можно в нашем [демомагазине](https://show.cloudpayments.ru/).
Использовать можно как [тестовые](#testirovanie), так и реальные карточные данные.   
Деньги списываться не будут.


## Классическая интеграция

**Apple Pay Merchant ID, сертификаты и домены**

Для использования технологии Apple Pay вам необходимо зарегистрировать Merchant ID, сформировать платежный сертификат, сертификат для веб-платежей и подтвердить владение доменами сайтов, на которых будет производиться оплата.

**Регистрация Merchant ID:**  

1. Зайдите в консоль [Apple Developer Account](https://developer.apple.com/), далее в раздел `«Certificates, IDs & Profiles»`, далее `«Merchant IDs»`. Зарегистрируйте новый Merchant ID:  
	* В поле `Description` укажите произвольное описание;  
	* В поле `Identifier` укажите адрес вашего основного сайта в обратном порядке и с префиксом `«merchant»`. Например, если адрес вашего основного сайт `shop.domain.ru`, то `Identitfier` — `merchant.ru.domain.shop`

2. Сохраните результат.

**Создание сертификатов:**  

1. Напишите письмо на адрес `support@cloudpayments.ru` с указанием зарегистрированного в Apple Merchant ID. Наша служба поддержки сформирует два запроса на сертификаты и отправит вам обратным письмом. Процесс занимает не более 10 минут, но в зависимости от загруженности может растянуться на целый день.  

2. Сфомируйте в консоли [Apple Developer](https://developer.apple.com/) два сертификата: Payment Processing Certificate и Merchant Identity Certificate и передайте в нашу службу поддержки.

**Подтверждение доменов:**  

1. Добавьте домены в консоли [Apple Developer](https://developer.apple.com/) для каждого сайта, где планируете принимать оплату через Apple Pay. Обратите внимание, что сайты должны использовать схему `HTTPS` и поддерживать протокол `TLS 1.2`.  

2. Подтвердите владение доменами.

В [виджете](#platezhnyy-vidzhet) появится возможность оплачивать через Apple Pay.

**Прием платежей с Apple Pay**

Схема оплаты включает в себя 3 этапа:

* Проверка совместимости устройства. Если устройство поддерживается, то нужно показать покупателю кнопку Apple Pay.
* Авторизация платежа — подтверждение покупателем оплаты вводом кода или отпечатком пальца.
* Обработка платежа. После авторизации Apple формирует зашифрованный токен, который необходимо передать в API системы CloudPayments.

**Apple Pay в мобильных приложениях**

Используйте SDK PassKit от Apple для получения PaymentToken и метод оплаты по криптограмме в API для проведения платежа.

![image4-2](images/image1-4.png)

<div id="applepayself"></div>
**Самостоятельное размещение Apple Pay на сайте**  


Если вы хотите разместить кнопку Apple Pay непосредственно на вашем сайте по примеру ниже (не через платёжный [виджет](#platezhnyy-vidzhet)), то следуйте дальнейшей инструкции.

Интеграция предполагает использование клиентской части (javascript) и серверной. На клиенте вы проверяете совместимость устройства и обрабатываете события: создание сессии, авторизация платежа, обработка платежа.

**На серверной части необходимо выполнять вызовы API:**  

1. [Запуск сессии](#zapusk-sessii-dlya-oplaty-cherez-apple-pay) Apple Pay;

2. [Проведение оплаты по криптограмме](#oplata-po-kriptogramme)

**Пример js кода:**

```shell
//в примере используется библиотека jquery
//дополнительные скрипты прописывать не требуется, все объекты для платежей Apple Pay уже есть в браузере Safari

if (window.ApplePaySession) { //проверка устройства
    var merchantIdentifier = 'Ваш Apple Merchant ID';
    var promise = ApplePaySession.canMakePaymentsWithActiveCard(merchantIdentifier);
    promise.then(function (canMakePayments) {
        if (canMakePayments) {
            $('#apple-pay').show(); //кнопка Apple Pay
        }
    });
}
$('#apple-pay').click(function () { //обработчик кнопки
    var request = {
        // requiredShippingContactFields: ['email'], //Раскомментируйте, если вам нужен e-mail. Также можно запросить postalAddress, phone, name.
        countryCode: 'RU',
        currencyCode: 'RUB',
        supportedNetworks: ['visa', 'masterCard'],
        merchantCapabilities: ['supports3DS'],
        //Назначение платежа указывайте только латиницей!
        total: { label: 'Test', amount: '1.00' }, //назначение платежа и сумма
    }
    var session = new ApplePaySession(1, request);

    // обработчик события для создания merchant session.
    session.onvalidatemerchant = function (event) {

        var data = {
            validationUrl: event.validationURL
        };

        // отправьте запрос на ваш сервер, а далее запросите API CloudPayments
        // для запуска сессии
        $.post("/ApplePay/StartSession", data).then(function (result) {
            session.completeMerchantValidation(result.Model);
        });
    };

    // обработчик события авторизации платежа
    session.onpaymentauthorized = function (event) {

        //var email = event.payment.shippingContact.emailAddress; //если был запрошен адрес e-mail
        //var phone = event.payment.shippingContact.phoneNumber; //если был запрошен телефон
        //все варианты смотрите на сайте https://developer.apple.com/reference/applepayjs/paymentcontact

        var data = {
            cryptogram: JSON.stringify(event.payment.token)
        };

        //отправьте запрос на ваш сервер, а далее запросите API CloudPayments
        //для проведения оплаты
        $.post("/ApplePay/Pay", data).then(function (result) {
            var status;
            if (result.Success) {
                status = ApplePaySession.STATUS_SUCCESS;
            } else {
                status = ApplePaySession.STATUS_FAILURE;
            }

            session.completePayment(status);
        });
    };

    // Начало сессии Apple Pay
    session.begin();
});
```

<aside class="notice">Для приема платежей через Apple Pay ваш сайт должен работать по схеме HTTPS и поддерживать протокол TLS версии 1.2.</aside>

## Упрощенная интеграция (только для  веб-сайтов)

Мы упростили процедуру регистрации в Apple. Для веб-сайтов создание учетной записи в консоли [Apple Developer](https://developer.apple.com/) при использовании наших платёжных решений больше не является необходимостью. Теперь достаточно в [личном кабинете](https://merchant.cloudpayments.ru/login) в настройках сайта: 

* Скачать верификационный файл, разместить его на вашем сайте по указанному в настройках пути;
* Включить опцию `"Apple Pay"`.

Для успешного включения опции ваш сайт должен работать по схеме HTTPS и поддерживать протокол TLS версии 1.2.  
![image4](images/image4.png)

Если вы используете [Checkout скрипт](#skript-checkout), вам  необходимо [самостоятельно разместить](#applepayself)  ApplePay.  
В случае использования [виджета](#platezhnyy-vidzhet) возможность оплаты Apple Pay появится автоматически после включения одноимённой опции в нашем ЛК. 



## Условия использования Apple Pay

Вы можете применять технологию Apple Pay с системой CloudPayments на сайтах и в мобильных приложениях при соблюдении условий использования от компании Apple:

*  Вам необходим аккаунт в программе [Apple Developer](https://developer.apple.com/) для регистрации индивидуального `Merchant ID`(при упрощенной интеграции `Merchant ID` можно уточнить у вашего менеджера);
*  Apple Pay нельзя использовать для оплаты табачной продукции, реплики, товаров для взрослых, покупки виртуальной валюты, пополнения кошельков;
*  Apple Pay не заменяет In App Purchase в мобильных приложениях;
*  При [самостоятельном размещении](#applepayself) в веб и для мобильного приложения вам необходимо придерживаться рекомендации по использованию от [Apple](https://developer.apple.com/apple-pay/Apple-Pay-Identity-Guidelines.pdf);
*  Для использования Apple Pay с целью сбора благотворительных пожертвований необходимо зарегистрироваться на портале [Benevity](https://causes.benevity.org/causes/claim-cause-search).  

![image4-1](images/image1-3.png)
 