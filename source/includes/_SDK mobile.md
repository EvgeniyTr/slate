# SDK для мобильных приложений

Мобильное SDK — библиотеки и инструкция для встраивания оплаты по картам в мобильные приложения на iOS и Android. Позволяет запрашивать карточные данные в нативной форме приложения, делать [[3‒D Secure аутентификацию]]() и проводить платежи.

Схема работы мобильного приложения:

1. В приложении необходимо получить данные карты: номер, срок действия, имя держателя и CVV.
2. Создать криптограмму карточных данных при помощи SDK.
3. Отправить криптограмму и все данные для платежа с мобильного устройства на ваш сервер.
4. С сервера выполнить оплату через платежное API CloudPayments.  
![mobile-schema](images/mobile-schema.png)

 Условия использования:

* Крайне нежелательно отправлять данные для оплаты с мобильного устройства напрямую в API CloudPayments. Необходимо держать учетные данные для доступа к API на вашем сервере, а в мобильном приложении поддерживать принцип тонкого клиента.
*  Для вашего сервера не потребуется сертификация PCI, так как карточные данные на него приходят в зашифрованном виде, а ключи для расшифрования есть только в платежном шлюзе CloudPayments (в приложении и библиотеке их также нет).
*  Данные карты можно вводить с клавиатуры или сканировать камерой (например, через [card.io](https://www.card.io/). Полный номер карты и CVV нельзя сохранять или записывать в лог.
*  Мобильное SDK нельзя использовать для мобильных терминалов (mPos). Единственное применение — в приложениях, которые будут установлены на телефонах и планшетах ваших покупателей.
*  По правилам [[Google Play]]() и [[AppStore]]() разработчик мобильного приложения имеет право принимать платежи с использованием сторонних платежных систем только для продажи нецифровых товаров, или цифровых товаров, которые можно использовать вне приложения.

## SDK для iOS

Демо приложение, которое показывает работу с SDK CloudPayments для платформы iOS можно скачать с [GitHub](https://github.com/cloudpayments/CloudPayments_iOSSDKDemo)

Приложение демонстрирует как получить карточные данные, сформировать криптограмму, провести [[3-D Secure авторизацию]]() и выполнить платеж на iPhone или iPad.

Для установки необходимо сделать клон репозитория:

```shell
git clone https://github.com/cloudpayments/CloudPayments_iOSSDKDemo.git
```

И установить две сторонние библиотеки (используются только в демо-приложении) 

```shell
git submodule update --init CloudPaymentsAPIDemo/Libraries/AFNetworking/
git submodule update --init CloudPaymentsAPIDemo/Libraries/SVProgressHUD/
```

### Функции SDK

SDK CloudPayments (CloudPaymentsAPI.framework) позволяет: 

* Проверить номер карты (защита от опечаток)

```shell
[CPService isCardNumberValid: cardNumberString];
```

* определять тип платежной системы (Visa, MasterCard, Maestro)

```shell
[CPService cardTypeFromCardNumber: cardNumberString];
```

* Создать криптограмму для безопасной передачи на сервер

```shell
PService *_apiService = [[CPService alloc] init];
    NSString *cryptogramPacket = [_apiService makeCardCryptogramPacket:self.cardNumberString
                                    andExpDate:self.cardExpirationDateString
                                    andCVV:self.cardCVVString
                                    andStorePublicID:_apiPublicID];
```

## SDK для Android

Демо приложение, которое показывает работу с SDK CloudPayments для платформы Android можно скачать с [GitHub](https://github.com/cloudpayments/CloudPayments_AndroidSDKDemo)

Приложение демонстрирует как получить карточные данные, сформировать криптограмму, провести [[3-D Secure авторизацию]]() и выполнить платеж на Android.

Для установки необходимо сделать клон репозитория:

```shell
git clone https://github.com/cloudpayments/CloudPayments_AndroidSDKDemo.git
```

### Функции SDK

SDK CloudPayments (CloudPaymentsAPI.framework) позволяет: 

* Проверить номер карты (защита от опечаток)

```shell
boolean ru.cloudpayments.sdk.Card.isValidNumber(java.lang.String number);
```

* определять тип платежной системы (Visa, MasterCard, Maestro)

```shell
java.lang.String ru.cloudpayments.sdk.Card.getType(java.lang.String number);
```

* Создать криптограмму для безопасной передачи на сервер

```shell
java.lang.String ru.cloudpayments.sdk.Card.cardCryptogram(java.lang.String publicId);
```



