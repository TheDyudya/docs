Платежная форма Checkout представляет собой готовое техническое решение, которое позволяет совершить платеж. Форма отдается с веб-серверов RBKmoney и открывается в iframe таким образом, чтобы создать для плательщика видимость нахождения на сайте мерчанта.

## Примеры интеграции платежной формы
Рассмотрим сценарии с использованием [инвойсов](https://developer.rbk.money/api/#tag/Invoices):
### Использование HTML API

```html
<form action="https://<your-server-side>" method="GET">
    <script src="https://checkout.rbk.money/checkout.js" class="rbkmoney-checkout"
            data-invoice-id="string"
            data-invoice-access-token="string"
            data-name="Company name"
            data-description="Some product"
            data-label="Pay with RBKmoney">
    </script>
</form>
```

С помощью `form` можно задать callback. В случае успешного платежа, будет произведен submit формы.

| data-* атрибут            | Описание                                                             | Обязательный | Возможные значения   |
| :-----------------------: | -------------------------------------------------------------------- | :-----------:| :-------------------:|
| data-invoice-id           | Идентификатор инвойса                                                | ✓            | oVU2LzUCbQ           |
| data-invoice-access-token | Токен для доступа к указанному инвойсу                               | ✓            | eyJhbGciOiJSUzI1N... |
| data-name                 | Наименование вашей компании или сайта                                |              | Company name         |
| data-description          | Описание вашего продукта или сервиса                                 |              | Some product         |
| data-label                | Текст кнопки открытия формы                                          |              | Pay with RBKmoney    |
| data-email                | Если вы знаете email вашего плательщика, вы можете его предзаполнить |              | example@mail.com     |
| data-obscure-card-cvv     | Затенять карточный cvv код                                           |              | true / false         |
| data-require-card-holder  | Требовать от плательщика заполнять поле card holder                  |              | true / false         |
| data-locale               | Настройка локализации платежной формы                                |              | auto / ru / en       |
| data-popup-mode           | Открывать checkout в новой вкладке браузера                          |              | true / false         |

### Использование JS API

```javascript
const checkout = RbkmoneyCheckout.configure({
    invoiceID: 'string',
    invoiceAccessToken: 'string',
    name: 'Company name',
    description: 'Some product',
    opened: function () {
        console.log('Checkout opened');
    },
    closed: function () {
        console.log('Checkout closed');
    },
    finished: function () {
        console.log('Payment successful finished');
    }
});

document.getElementById('customButton').addEventListener('click', function () {
    checkout.open();
});

window.addEventListener('popstate', function () {
    checkout.close();
});
```

| Свойство конфигурации    | Описание                                                             | Обязательное | Возможные значения   |
| :----------------------: | -------------------------------------------------------------------- | :-----------:| :-------------------:|
| invoiceID                | Идентификатор инвойса                                                | ✓            | oVU2LzUCbQ           |
| invoiceAccessToken       | Токен для доступа к указанному инвойсу                               | ✓            | eyJhbGciOiJSUzI1N... |
| name                     | Наименование вашей компании или сайта                                |              | Company name         |
| description              | Описание вашего продукта или сервиса                                 |              | Some product         |
| email                    | Если вы знаете email вашего плательщика, вы можете его предзаполнить |              | example@mail.com     |
| opened                   | Callback на открытие окна checkout                                   |              | function             |
| closed                   | Callback на закрытие окна checkout                                   |              | function             |
| finished                 | Callback на успешное завершение платежа                              |              | function             |
| obscureCardCvv           | Затенять карточный cvv код                                           |              | true / false         |
| requireCardHolder        | Требовать от плательщика заполнять поле card holder                  |              | true / false         |
| locale                   | Настройка локализации платежной формы                                |              | auto / ru / en       |
| popupMode                | Открывать checkout в новой вкладке браузера                          |              | true / false         |

Примечание: Checkout возвращает управление в callback только при успешном завершении платежа. С целью увеличения конверсии оплат при неуспешных попытках оплаты (например неверно введены данные или на карте недостаточно средств) мы оставляем UA плательщика на форме, позволяя исправить ошибку, использовать другую карту и т.п.

### Совершение оплаты с использованием [шаблонов инвойсов](https://developer.rbk.money/api/#tag/InvoiceTemplates)
Вместо пары `invoiceID` и `invoiceAccessToken`, необходимо использовать идентификатор шаблона инвойса и токен для доступа к указанному шаблону.

* HTML API: `data-invoice-template-id`, `data-invoice-template-access-token`.
* JS API: `invoiceTemplateID`, `invoiceTemplateAccessToken`.

В случае с шаблонами инвойса, c открытой стоймостью товаров или услуг, возможно заранее указать сумму к оплате:

| Свойство конфигурации (JS API) | data-* атрибут (HTML API) | Описание                                                                                                                 | Возможные значения |
| :----------------------------: | :------------------------:| :-----------------------------------------------------------------------------------------------------------------------:| :----------------: |
| amount                         | data-amount               | Сумма к оплате, в минорных денежных единицах, например в копейках в случае указания российских рублей в качестве валюты. | integer            |

Пример интеграции:

JS API
<script async src="//jsfiddle.net/Ildar_Galeev/2g59m5fv/19/embed/js,html/"></script>

HTML API
<script async src="//jsfiddle.net/Ildar_Galeev/xrx0qgfk/embed/html/"></script>

### Создание привязки [плательщика](https://developer.rbk.money/api/#tag/Customers)
Вместо пары `invoiceID` и `invoiceAccessToken`, необходимо использовать идентификатор плательщика и токен для доступа к указанному плательщику.

* HTML API: `data-customer-id`, `data-customer-access-token`.
* JS API: `customerID`, `customerAccessToken`.

### Совершение оплаты с удержанием денежных средств
Для настройки необходимо:

1. Указать параметр конфигурации `paymentFlowHold` со значением: `true`.
2. Выбрать политику, которая будет применена по истечении срока удержания денежных средств. Возможные значения:
    * cancel - удержание в пользу плательщика. Данная политика используется по умолчанию.
    * capture - удержание в пользу мерчанта.

| Свойство конфигурации (JS API) | data-* атрибут (HTML API) | Описание                                                | Возможные значения |
| :----------------------------: | :------------------------:| :-----------------------------------------------------: | :-----------------:|
| paymentFlowHold                | data-payment-flow-hold    | Признак совершения оплаты с удержанием денежных средств | true / false       |
| holdExpiration                 | data-hold-expiration      | Политика управления удержанием денежных средств         | cancel / capture   |

### Рекуррент по первому платежу

| Свойство конфигурации (JS API) | data-* атрибут (HTML API) | Описание                                 | Возможные значения | Значение по умолчанию  |
| :----------------------------: | :------------------------:| :--------------------------------------: | :-----------------:| :--------------------: |
| recurring                      | data-recurring            | Признак выполнения рекуррентного платежа | true / false       | false                  |

### Управление методами оплаты

| Метод оплаты         | Свойство конфигурации (JS API) | data-* атрибут (HTML API) | Возможные значения | Значение по умолчанию  |
| :-----------------:  | :----------------------------: | :------------------------:| :-----------------:| :--------------------: |
| Банковская карта     | bankCard                       | data-bank-card            | true / false       | true                   |
| Электронные кошельки | wallets                        | data-wallets              | true / false       | true                   |
| Терминалы оплаты     | terminals                      | data-terminals            | true / false       | true                   |
| Apple Pay            | applePay                       | data-apple-pay            | true / false       | true                   |
| Google Pay           | googlePay                      | data-google-pay           | true / false       | true                   |
| Samsung Pay          | samsungPay                     | data-samsung-pay          | true / false       | true                   |

Управление методом оплаты, который будет предложен плательщику в первую очередь:

* HTML API: `data-initial-payment-method`
* JS API: `initialPaymentMethod`

Возможные значения:

- `bankCard` - Банковская карта.

- `terminalEuroset` - Терминалы "Евросеть".

- `walletQiwi` - Электоронный кошелек "Qiwi".

- `applePay` - Apple Pay.

- `googlePay` - Google Pay.

- `samsungPay` - Samsung Pay.

### Темы

| Свойство конфигурации (JS API) | data-* атрибут (HTML API) | Описание                                 | Возможные значения | Значение по умолчанию  |
| :----------------------------: | :------------------------:| :--------------------------------------: | :-----------------:| :--------------------: |
| theme                          | data-theme                | Название выбранной темы                  | main / coral       | main                   |

### Предотвращение блокировки функции открытия checkout
Не вызывайте функцию открытия checkout в callback. Большинство мобильных браузеров блокируют подобное поведение. Открытие нового окна должно происходить в результате действия пользователя.

```javascript
// Будет работать:
document.getElementById("button").addEventListener("click", function() {
    checkout.open();
});

// Не будет работать:
document.getElementById("button").addEventListener("click", function() {
    someFunction().then(function() {
        checkout.open();
    });
});
```

### Ошибка `RbkmoneyCheckout is not defined`

При некорректном встраивании Checkout в код вашей страницы часто может возникать ошибка `RbkmoneyCheckout is not defined`. Она связана с тем, что вы вызываете функцию RbkmoneyCheckout.configure() до полной инициализации DOM. Используйте удобные вам практики для того, чтобы обеспечить вызов платежной формы после полной инициализации модели документа.

Частный случай решения этой задачи с использованием `jquery`, указывающий направление ее решения может выглядеть так:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://code.jquery.com/jquery-3.3.1.min.js" integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=" crossorigin="anonymous"></script>
    <script type="text/javascript" src="https://checkout.rbk.money/checkout.js"></script>
</head>
<body>
    <script>
    $(function() {
        $('#rbkmoney-checkout').on('click', function(e) {
            e.preventDefault();
            const checkout = initCheckout('{INVOICE_ID}', '{INVOICE_ACCESS_TOKEN}');
            checkout.open();
        });
    
        function initCheckout(invoiceID, invoiceAccessToken) {
            return RbkmoneyCheckout.configure({
                invoiceID: invoiceID,
                invoiceAccessToken: invoiceAccessToken,
                name: 'Product name',
                description: 'Product description',
                email: 'example@example.com',
                finished: function() {
                    console.log('Payment successful finished');
                },
                opened: function() {
                    console.log('Checkout opened');
                },
                closed: function() {
                    console.log('Checkout closed');
                }
            });
        }
    });
    </script>
    <button id="rbkmoney-checkout">Pay with RBKmoney</button>
</body>
</html>
```
