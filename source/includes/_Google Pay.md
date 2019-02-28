# Google Pay  

![image-1-1](images/image-1-1.png)

Google Pay — это быстрый и простой способ оплаты в мобильных приложениях и браузере Chrome на всех устройствах Android. Google Pay предоставляет все удобства для оплаты и сохраняет данные покупателя в безопасности.

Google Pay работает с картами Visa и MasterCard и доступен всем организациям и индивидуальным предпринимателям, подключенным к системе CloudPayments без дополнительных соглашений и без изменений в условиях работы.

[[тут пример оплаты через Google PAY]]()

**Условия использования**

Вы можете применять технологию Google Pay с системой CloudPayments на сайтах и в мобильных приложениях при соблюдении следующий требований:

* Необходимо соблюдать [условия использования от компании Google](https://payments.developers.google.com/terms/sellertos), в том числе:
  <br>[Список](https://payments.developers.google.com/terms/aup) запрещенных товаров и услуг;
  <br>[Требования](https://developers.google.com/pay/api/brand-guidelines) по брендированию.
* Для приема платежей в мобильном приложении или при размещении кнопки GPay на вашем сайте (как в примере выше), необходимо зарегистрировать приложение и сайт в Google и пройти процедуру проверки.

**Принцип работы Google Pay**

Google Pay объединяет в себе возможности оплаты через приложение Google Pay (ранее Android Pay) в магазинах и обычной оплаты картой, сохраненной в аккаунте пользователя Google.

Если покупатель производит оплату в приложении или на сайте с мобильного устройства, поддерживающего Google Pay, ему будет предложено подтвердить оплату способом, зависящим от возможностей устройства.

При оплате с устройства без установленного приложения Google Pay, покупателю будет предложено выбрать сохраненную карту из его Google аккаунта и, на усмотрение процессора, пройти [3-d Secure](#3-d-secure) аутентификацию.


<div id="integraciya"></div>
**Интеграция**

_Регистрация сайтов и приложений_

Для приема платежей в приложении или на сайте с прямой интеграцией:

1. Проверьте, что соблюдены все [требования по брендированию](https://developers.google.com/pay/api/brand-guidelines);
2. Заполните [форму регистрации](https://services.google.com/fb/forms/googlepayAPIenable/), после чего с вами свяжется представитель Google и проинструктирует по дальнейшим шагам;
3. Будьте готовы отправить на проверку сборку приложения (.apk) или ссылку на сайт со страницей оплаты.

Для приема платежей на сайте через [[виджет]](#platezhnyy-vidzhet)

<aside class="notice">Регистрация и проверка сайта не требуются.</aside>

**Прием платежей с Google Pay**

Схема оплаты включает в себя 3 этапа:

* Проверка совместимости устройства. Если устройство поддерживается, то нужно показать покупателю кнопку GPay;  
* Авторизация платежа – подтверждение покупателем оплаты;  
* Обработка платежа. После авторизации Google формирует токен, который необходимо передать в API системы CloudPayments.  

**Google Pay в мобильных приложениях**

Используйте [Google Pay API](https://developers.google.com/pay/api/setup) для получения PaymentData и метод [оплаты по криптограмме](#oplata-po-kriptogramme) в API для проведения платежа.

При формирования запроса на платежные данные укажите тип оплаты через шлюз:
`Wallet-Constants.PAYMENT_METHOD_TOKENIZATION_TYPE_PAYMENT_GATEWAY`
и добавьте два параметра:

1. `gateway: cloudpayments`;
2. `gatewayMerchantId: Ваш Public ID из личного кабинета CloudPayments.`

```shell
PaymentMethodTokenizationParameters params =
        PaymentMethodTokenizationParameters.newBuilder()
            .setPaymentMethodTokenizationType(
                WalletConstants.PAYMENT_METHOD_TOKENIZATION_TYPE_PAYMENT_GATEWAY)
            .addParameter("gateway", "cloudpayments")
            .addParameter("gatewayMerchantId", "Ваш Public ID")
            .build();
```

Смотрите также:

* [Пример](https://github.com/cloudpayments/CloudPayments_AndroidCheckout) использования Google Pay API для оплаты через CloudPayments;
* [Пример](https://github.com/google-pay/android-quickstart) использования Google Pay API от Google.

**Google Pay на сайте**

Есть два варианта приема платежей Google Pay на вашем сайте: используя встроенные возможности [виджета](#platezhnyy-vidzhet) или прямая интеграция — размещение кнопки непосредственно на вашей страничке без дополнительных форм. В первом случае, никакой дополнительной интеграции не требуется. Во втором — ваш сайт должен работать по схеме `HTTPS` и поддерживать протокол `TLS версии 1.2`. Домен сайта должен быть предварительно [зарегистрирован и подтвержден в Google](#integraciya).

Прямая интеграция предполагает использование клиентской части (javascript) и серверной. На клиенте вы проверяете совместимость устройства и получаете платежные данные, а на сервере отправляете [запрос на оплату](#oplata-po-kriptogramme) в API.

**Пример js кода:**

```shell
//в примере используется библиотека jquery
var allowedPaymentMethods = ['CARD', 'TOKENIZED_CARD'];
var allowedCardNetworks = ['MASTERCARD', 'VISA']; 
var tokenizationParameters = {
    tokenizationType: 'PAYMENT_GATEWAY',
    parameters: {
        'gateway': 'cloudpayments',
        'gatewayMerchantId': 'pk_b9fa79f3d759c56a9b856d8ac59b1' //ваш public id
    }
}
function getGooglePaymentsClient() {
    return (new google.payments.api.PaymentsClient({ environment: 'PRODUCTION' }));
}

//обработчик загрузки Google Pay API
function onGooglePayLoaded() {
    var paymentsClient = getGooglePaymentsClient();
    //проверка устройства
    paymentsClient.isReadyToPay({ allowedPaymentMethods: allowedPaymentMethods })
        .then(function (response) {
            if (response.result) {
                $('#gpay-checkout').show(); //включение кнопки
                $('#gpay-checkout').click(onGooglePaymentButtonClicked);
            }
        });
}

//настройки
function getGooglePaymentDataConfiguration() {
    return {
        merchantId: '04349806409183181471', //выдается после регистрации в Google
        paymentMethodTokenizationParameters: tokenizationParameters,
        allowedPaymentMethods: allowedPaymentMethods,
        cardRequirements: {
            allowedCardNetworks: allowedCardNetworks
        }
    };
}

//информация о транзакции
function getGoogleTransactionInfo() {
    return {
        currencyCode: 'RUB',
        totalPriceStatus: 'FINAL',
        totalPrice: '10.00'
    };
}

//обработчик кнопки
function onGooglePaymentButtonClicked() {
    var paymentDataRequest = getGooglePaymentDataConfiguration();
    paymentDataRequest.transactionInfo = getGoogleTransactionInfo();

    var paymentsClient = getGooglePaymentsClient();
    paymentsClient.loadPaymentData(paymentDataRequest)
        .then(function (paymentData) {
            processPayment(paymentData);
        });
}            

//обработка платежа
function processPayment(paymentData) {
    var data = {
        cryptogram: JSON.stringify(paymentData.paymentMethodToken.token)
    };

    // отправьте запрос на ваш сервер, а далее запросите API CloudPayments
    // для проведения оплаты 
    $.post("/GooglePay/Pay", data).then(function (result) {
        if (result.Success) {
            //оплата успешно завершена
        } else {
            if (result.Model.PaReq) { 
                //требуется 3-d secure
            } else {
                //оплата отклонена
            }
        }
    });
}
```

В конце страницы необходимо прописать скрипт для вызова функций Google Pay API:

```shell
<script async src="https://pay.google.com/gp/p/js/pay.js" onload="onGooglePayLoaded()"></script>
```

**Смотрите также:**

* [Документацию](https://developers.google.com/pay/api/web/setup) Google Pay API для сайтов.
* [Как обрабатывать](#obrabotka-3-d-secure) 3-d Secure.




