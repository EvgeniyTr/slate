# PCI DSS

PCI DSS — стандарт информационной безопасности, принятый в индустрии платежных карт Visa и MasterCard. Соблюдать требования стандарта обязаны все компании, которые принимают карты к оплате. Некоторым компаниям необходимо подтверждать свое соответствие.

## Соответствие требованиям

Основное требование стандарта — максимально ограничить доступ к данным платежных карт. Оптимальным решением является вообще не иметь к ним доступа, а вместо этого использовать сертифицированных провайдеров для приема платежей. На практике это означает, что нельзя ни запрашивать, ни передавать номера карт. Если звонит клиент, говорит, что не проходит платеж и начинает диктовать номер, нужно его перебивать. Если присылает по почте/скайпу — удалять сообщение и просить, чтоб больше так не делал.

К охраняемым данным относятся полный номер карты и код CVV2/CVC2 (последние 3 цифры на обратной стороне). Имя владельца, срок действия и маскированный номер карты (первые 6 и последние 4 цифры) в защите по требованиям стандарта не нуждаются поэтому их можно использовать в разумных пределах.

## Соответствие PCI DSS вместе с CloudPayments

CloudPayments — сертифицированный сервис провайдер с максимальным уровнем соответствия PCI DSS, предоставляющим право хранить данные платежных карт и обрабатывать более 6 миллионов платежей ежегодно. Подтверждение соответствия проходит каждый год в рамках сертификационного аудита.

Все платежные инструменты CloudPayments спроектированы таким образом, что при их использовании вы автоматически соответствуете требованиям по безопасности. Дополнительных мер предпринимать не нужно.

Исключением является прием платежей по технологии [Checkout](#skript-checkout). Для использования необходимо подтверждать соответствие: заполнить лист самооценки и ежеквартально проверять сайт на уязвимости специальным сканером.

## Технология Checkout

[Checkout](#skript-checkout) — уникальная технология токенизации карт для приема платежей на вашем сайте, в вашей форме без встроенных iframe элементов, что дает максимальный контроль и конверсию прохождения платежей. Данные платежных карт шифруются в браузере покупателя, поэтому ваш сайт не принимает участие в обработке и хранении номеров, что значительно сокращает область применения требований PCI DSS. Тем не менее, сайт влияет на безопасность карточных данных и для его защиты необходимо выполнять сканирование не менее одного раза в квартал для поиска вирусов и уязвимостей. Сканирование должно проводится аккредитованным вендором (ASV) из списка, представленного на сайте совета PCI.

## ASV сканирование

ASV сканирование — автоматизированная проверка вашего сайта на наличие уязвимостей. Сканер проверяет наличие вирусов, известных уязвимостей, таких как XSS, SQL Injections и так далее, после чего составляет детальный отчет с инструкцией по устранению проблем, если они были обнаружены.

Использование сканера необходимо для приема платежей по технологии [Checkout](#skript-checkout), для остальных инструментов: виджет, мобильное SDK, рекарринг и рекуррент его использование не требуется.

Выбор вендора для сканирования остается на ваше усмотрение, но он должен быть из [списка на сайте совета PCI](https://www.pcisecuritystandards.org/approved_companies_providers/approved_scanning_vendors.php). Если у вас нет предпочтений, мы рекомендуем использовать Trustwave — международную компанию, признанного лидера в информационной безопасности.

## Инструкция для подключения ASV сканера Trustwave

Годовая стоимость пакета услуг составляет 229 долларов и включает в себя подписку на сканирование, помощь в заполнении листа самооценки, учебные материалы и политику безопасности.

Для подключения необходимо:

1. Зарегистрируйтесь по адресу `https://pci.trustwave.com/pci` (кнопка Get Started).
    Во время регистрации нужно будет указать Merchant ID, для уточнения обратитесь к своему менеджеру или напишите письмо на адрес `support@cloudpayments.ru`.
2. На втором шаге укажите, что вы принимаете платежи на своем сайте.
3. На третьем шаге также укажите вариант «My Website».
4. Оплатите 229$ (принимают только карты).
5. Настройте IP адреса вашего сайта и расписание для сканирования.

