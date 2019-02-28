# Тест окна оплаты в рассрочку

<html>
  <head>
  <script src="https://widget.cloudpayments.ru/bundles/cloudpayments"></script>
    <meta charset="utf-8">
    <title>оплаты в рассрочку</title>
    <style>
      #zatemnenie {
        background: rgba(102, 102, 102, 0.5);
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        display: none;
      }
      #okno {
        width: 800px;
        height: 100px;
        text-align: left;
        padding: 15px;
        border: 3px solid #0000cc;
        border-radius: 10px;
        color: #0000cc;
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        margin: auto;
        background: #fff;
      }
      #zatemnenie:target {display: block;}
      .close {
        display: inline-block;
        border: 1px solid #0000cc;
        color: #0000cc;
        padding: 0 12px;
        margin: 10px;
        text-decoration: none;
        background: #f2f2f2;
        font-size: 14pt;
        cursor:pointer;
      }
      .close:hover {background: #e6e6ff;}
    </style>
  </head>
  
  <body>
   
    <div id="zatemnenie">
		<div id="okno">
        <div class="form-group">
            <p>Стоимость: 12 000 рублей. Пожалуйста, выберите вариант оплаты:</p>
            <div>
                <label for="select-sample-5-now">
                    <input name="select-sample-5" id="select-sample-5-now" checked="checked" type="radio">
                    оплатить 12 000 рублей сейчас
                </label>
            </div>
            <div>
                <label for="select-sample-5-later">
                    <input name="select-sample-5" id="select-sample-5-later" type="radio">
                    оплатить 6 000 рублей сейчас, остальное в рассрочку на 6 месяцев
                </label>
            </div>
        </div>
			<input id="checkout-sample-5" value="Оплатить" class="btn" type="button">
		</div>
    </div>
     
    <a href="#zatemnenie">Вызвать всплывающее окно</a>
 
  </body>
</html>