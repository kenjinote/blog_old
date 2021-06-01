---
layout: page
title: ABOUT
---

### ブログの内容について
本ブログでは、昔ながらのWindowsの開発手法であるVisual Studio (C++)を使って、Win32APIを叩きながらガリガリとクライアントPC向けの書いていくプログラムを紹介しています。

最近では、そのようなプログラムの開発手法は、少なくなってきていて、流行りとしては、クライアントアプリ開発ではなく、Web系のプログラムが主流となってきていると思いますが、私の手元には1998年頃から当時はVisual C++ 6.0から書き溜めたプログラムが数多くあるため、そのプログラムを紹介していこうと思った次第です。

古いプログラミングスタイル（とはいっても現役で高速に動きいち早くWindowsのAPIに触れることができる）Win32API & C++の組み合わせでプログラムを作成する必要がある方の参考になれば幸いです。

### 著者
名前：山本 健児 (Yamamoto Kenji)  
名古屋出身、大阪在住

Web Page：[https://kenji.jp](https://kenji.jp)  
Twitter：[@kenjinote](https://twitter.com/kenjinote)

興味があること
- プログラミング (C / C++ / C# / アセンブリ言語 / Haskell)
- 数学 (代数 / 暗号)
- ゲーム (スプラトゥーン / 風来のシレン / Mother / パズルゲーム)

### ご連絡
お仕事のご依頼や本ブログのお問い合わせについては本ぺージの下部にあるメールかTwitterのダイレクトメッセージからお問い合わせをお願いいたします。

### 寄付
本ブログで公開しているプログラムにデジタル署名をつけたいと考えていますが、デジタル証明書（コードサイニング証明書）を発行するには料金がかかります。 デジタル署名が付与されたプログラムは、ダウンロードおよび実行時の警告メッセージ等が表示されず、手間なく利用できるようになります。本ブログから産まれたサンプルプログラムやツール類をより多くの方にお使いいただけるように資金をご支援いただけましたら幸いです。

<div id="smart-button-container">
    <div style="text-align: center"><label for="description">お名前 </label><input type="text" name="descriptionInput" id="description" maxlength="127" value=""></div>
      <p id="descriptionError" style="visibility: hidden; color:red; text-align: center;">Please enter a description</p>
    <div style="text-align: center"><label for="amount">寄付金 </label><input name="amountInput" type="number" id="amount" value="" ><span> JPY</span></div>
      <p id="priceLabelError" style="visibility: hidden; color:red; text-align: center;">Please enter a price</p>
    <div id="invoiceidDiv" style="text-align: center; display: none;"><label for="invoiceid"> </label><input name="invoiceid" maxlength="127" type="text" id="invoiceid" value="" ></div>
      <p id="invoiceidError" style="visibility: hidden; color:red; text-align: center;">Please enter an Invoice ID</p>
    <div style="text-align: center; margin-top: 0.625rem;" id="paypal-button-container"></div>
  </div>
  <script src="https://www.paypal.com/sdk/js?client-id=sb&currency=JPY" data-sdk-integration-source="button-factory"></script>
  <script>
  function initPayPalButton() {
    var description = document.querySelector('#smart-button-container #description');
    var amount = document.querySelector('#smart-button-container #amount');
    var descriptionError = document.querySelector('#smart-button-container #descriptionError');
    var priceError = document.querySelector('#smart-button-container #priceLabelError');
    var invoiceid = document.querySelector('#smart-button-container #invoiceid');
    var invoiceidError = document.querySelector('#smart-button-container #invoiceidError');
    var invoiceidDiv = document.querySelector('#smart-button-container #invoiceidDiv');

    var elArr = [description, amount];

    if (invoiceidDiv.firstChild.innerHTML.length > 1) {
      invoiceidDiv.style.display = "block";
    }

    var purchase_units = [];
    purchase_units[0] = {};
    purchase_units[0].amount = {};

    function validate(event) {
      return event.value.length > 0;
    }

    paypal.Buttons({
      style: {
        color: 'silver',
        shape: 'pill',
        label: 'paypal',
        layout: 'horizontal',
        
      },

      onInit: function (data, actions) {
        actions.disable();

        if(invoiceidDiv.style.display === "block") {
          elArr.push(invoiceid);
        }

        elArr.forEach(function (item) {
          item.addEventListener('keyup', function (event) {
            var result = elArr.every(validate);
            if (result) {
              actions.enable();
            } else {
              actions.disable();
            }
          });
        });
      },

      onClick: function () {
        if (description.value.length < 1) {
          descriptionError.style.visibility = "visible";
        } else {
          descriptionError.style.visibility = "hidden";
        }

        if (amount.value.length < 1) {
          priceError.style.visibility = "visible";
        } else {
          priceError.style.visibility = "hidden";
        }

        if (invoiceid.value.length < 1 && invoiceidDiv.style.display === "block") {
          invoiceidError.style.visibility = "visible";
        } else {
          invoiceidError.style.visibility = "hidden";
        }

        purchase_units[0].description = description.value;
        purchase_units[0].amount.value = amount.value;

        if(invoiceid.value !== '') {
          purchase_units[0].invoice_id = invoiceid.value;
        }
      },

      createOrder: function (data, actions) {
        return actions.order.create({
          purchase_units: purchase_units,
        });
      },

      onApprove: function (data, actions) {
        return actions.order.capture().then(function (details) {
          alert('Transaction completed by ' + details.payer.name.given_name + '!');
        });
      },

      onError: function (err) {
        console.log(err);
      }
    }).render('#paypal-button-container');
}
initPayPalButton();
</script>

### プライバシーポリシー
当サイト(hack.jp)では、記事の閲覧状況を把握するために、Google社のGoogleアナリティクスを利用しています。
Googleアナリティクスは、クッキー(Cookie)を利用して、当社サイトへの訪問状況を収集・記録・分析します。
Googleアナリティクスでデータが収集・処理される仕組み、Googleアナリティクスの利用規約及びGoogle社の
プライバシーポリシーについては以下のサイトをご覧ください。

- [Google アナリティクス利用規約](https://marketingplatform.google.com/about/analytics/terms/jp/)
- [GOOGLE プライバシー ポリシー](https://policies.google.com/privacy?hl=ja)
- [Google のサービスを使用するサイトやアプリから収集した情報の Google による使用](https://policies.google.com/technologies/partner-sites?hl=ja)
