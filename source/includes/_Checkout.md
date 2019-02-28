# Скрипт checkout

Checkout — скрипт, который прописывается на вашем сайте, собирает из указанной формы карточные данные и составляет из них криптограмму для оплаты через наш API.

Криптограмма формируется алгоритмом RSA с длиной ключа в 2048 бит и удовлетворяет стандарту по защите карточных данных. При соблюдении описанных ниже требований карточные данные к вам не попадают, но ваш сервер всё равно влияет на их безопасность.

## Требования

### Требования к форме:

* Должна работать по HTTPS соединению с валидным SSL сертификатом.
* На полях не должно быть атрибута "name" — это предотвращает попадание карточных данных на сервер при отправке формы.
* Поле для ввода номера карты должно поддерживать ввод от 14 до 19 цифр

### Требования к криптограмме:

* Должна формироваться только оригинальным скриптом checkout, загруженным с адресов системы.
* Криптограмму нельзя хранить после оплаты и использовать повторно.

### Требования к безопасности по PCI DSS:

С точки зрения PCI DSS, подобный способ подключения классифицируется как "E-commerce merchants who outsource all payment processing to PCI DSS validated third parties, and who have a website(s) that doesn’t directly receive cardholder data but that can impact the security of the payment transaction. No electronic storage, processing, or transmission of any cardholder data on the merchant’s systems or premises.", то есть обработка платежных данных выполняется третьей стороной, но сайт влияет на безопасность карточных данных.

Для соблюдения требований стандарта, необходимо заполнять лист самооценки [SAQ-EP](https://www.pcisecuritystandards.org/documents/SAQ_A-EP_v3.docx) и ежеквартально проходить ASV тестирование.

Подробнее про соответствие требованиям PCI читайте в разделе [[PCI DSS]]().

## Установка

Для создания криптограммы необходимо прописать на странице с платежной формой скрипт `checkout` 

```shell
<script src="https://widget.cloudpayments.ru/bundles/checkout"></script>
```

Создать форму для ввода карточных данных 

```shell
<form id="paymentFormSample" autocomplete="off">
    <input type="text" data-cp="cardNumber">
    <input type="text" data-cp="expDateMonth">
    <input type="text" data-cp="expDateYear">
    <input type="text" data-cp="cvv">
    <input type="text" data-cp="name">
    <button type="submit">Оплатить 100 р.</button>
</form>
```

Поля ввода карточных данных должны быть помечены атрибутами: 

* `data-cp="cardNumber"` — поле с номером карты
* `data-cp="expDateMonth"` — поле с месяцем срока действия
* `data-cp="expDateYear"` — поле с годом срока действия
* `data-cp="cvv"` — поле с кодом CVV
* `data-cp="name"` — поле с именем и фамилией держателя карты

Прописать скрипт для создания криптограммы

```shell
this.createCryptogram = function () {
    var result = checkout.createCryptogramPacket();

    if (result.success) {
        // сформирована криптограмма
        alert(result.packet);
    }
    else {
        // найдены ошибки в введённых данных, объект `result.messages` формата: 
        // { name: "В имени держателя карты слишком много символов", cardNumber: "Неправильный номер карты" }
        // где `name`, `cardNumber` соответствуют значениям атрибутов `<input ... data-cp="cardNumber">`
       for (var msgName in result.messages) {
           alert(result.messages[msgName]);
       }
    }
};
                
$(function () {
    /* Создание checkout */
    checkout = new cp.Checkout(
    // public id из личного кабинета
    "test_api_00000000000000000000001",
    // тег, содержащий поля данных карты
    document.getElementById("paymentFormSample"));
});
```

Отправить криптограмму и имя держателя карты на сервер и вызывать метод оплаты через [[API]]()

<!--Для заполнения формы используйте тестовый номер карты 4925 0000 0000 0087, остальные данные — произвольные.-->

[[Для тестирования используйте тестовые карточки]](#490e809eef)

[`Пример платежной формы`](https://show.cloudpayments.ru/main)

При разработке собственной формы обратите внимание на следующие моменты:

* длина номера карты составляет от 16 до 19 цифр
* скрипт checkout не работает в устаревших, небезопасных браузерах, которые не поддерживают протоколы шифрования TLS версии 1.1 или выше. Например, в Internet Explorer 7
* окно 3DS можно показывать как в новом окне, так и во фрейме поверх страницы. Размер окна должен быть не менее 500х500 пикселей

