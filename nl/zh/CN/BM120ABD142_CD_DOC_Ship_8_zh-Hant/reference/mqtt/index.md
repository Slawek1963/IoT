---

copyright:
  years: 2015, 2017
lastupdated: "2017-03-14"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# MQTT 傳訊
{: #ref-mqtt}

MQTT 是裝置及應用程式用來與 {{site.data.keyword.iot_full}} 通訊的主要通訊協定。MQTT 是一種發佈與訂閱傳訊傳輸通訊協定，其設計目的是為了在感應器與行動裝置之間，有效率地交換即時資料。
{:shortdesc}

MQTT 透過 TCP/IP 執行，雖然可以直接將程式碼編寫為 TCP/IP，您也可以選擇使用程式庫來為您處理 MQTT 通訊協定的詳細資料。有很多種 MQTT 用戶端程式庫可供您使用。IBM 協助開發及支援數種用戶端程式庫，包括可從下列網站取得的用戶端程式庫：

- [MQTT 社群 Wiki ![外部鏈結圖示](../../../../icons/launch-glyph.svg "外部鏈結圖示")](https://github.com/mqtt/mqtt.github.io/wiki){: new_window}
- [Eclipse Paho 專案 ![外部鏈結圖示](../../../../icons/launch-glyph.svg "外部鏈結圖示")](http://eclipse.org/paho/){: new_window}

## 版本支援
{: #version-support}
如需 {{site.data.keyword.iot_short_notm}} 支援之 MQTT 版本的相關資訊，請參閱[標準和需求](../standards_and_requirements.html#mqtt)。

## 應用程式、裝置及閘道用戶端
{: #device-app-clients}

在 {{site.data.keyword.iot_short_notm}} 中，主要的物件類別為裝置及應用程式。閘道是裝置的子類別。

MQTT 用戶端會向 {{site.data.keyword.iot_short_notm}} 服務自我識別為一種物件類別。物件類別可決定連接用戶端時的功能。物件類別也會決定用戶端鑑別的機制。

應用程式和裝置搭配運作的 MQTT 主題空間不同。裝置只能在某個裝置範圍的主題空間內運作，而應用程式可以完整存取整個組織的主題空間。如需相關資訊，請參閱下列主題：

- [裝置的 MQTT 傳訊](../../devices/mqtt.html)
- [應用程式的 MQTT 傳訊](../../applications/mqtt.html)
- [閘道的 MQTT 傳訊](../../gateways/mqtt.html)

### 保留的訊息
{{site.data.keyword.iot_short_notm}} 提供 MQTT 傳訊的保留的訊息特性的有限支援。如果從裝置、閘道或應用程式傳送至 {{site.data.keyword.iot_short_notm}} 的 MQTT 訊息中保留的訊息旗標設為 true，則會將訊息處理為未保留的訊息。{{site.data.keyword.iot_short_notm}} 組織未獲授權，無法發佈保留的訊息。{{site.data.keyword.iot_short_notm}} 服務會置換設為 true 的保留的訊息旗標，而且處理訊息的方式就像保留的訊息旗標設為 false 一樣。

## 服務品質水準
{: #qos-levels}

MQTT 通訊協定提供三種在用戶端與伺服器之間遞送訊息的服務品質：「最多一次」、「最少一次」，以及「正好一次」。
雖然您可以使用任何服務品質水準來傳送事件和指令，但必須謹慎考量符合您需求的正確服務水準。選擇服務品質水準 '2' 並不一定會比水準 '0' 好。

### 最多一次 (QoS0)

「最多一次」的服務品質水準 (QoS0) 是最快的傳輸模式，有時會稱為「發動並遺忘」。訊息最多遞送一次，或是根本不遞送。透過網路遞送不會進行確認，而且不會儲存訊息。如果用戶端中斷連線，或是伺服器失敗，訊息可能會遺失。

MQTT 通訊協定不需要伺服器將服務品質水準 '0' 的發佈項目轉遞至用戶端。如果用戶端在伺服器收到發佈項目時斷線，根據伺服器的實作，可能會捨棄該發佈項目。

**提示：**依時間間隔傳送即時資料時，請使用服務品質水準 0。如果有單一訊息遺失，其實並沒有關係，因為之後很快就會再傳送包含較新資料的另一則訊息。在此情況下，使用較高服務品質的額外成本，並不會產生任何實質的好處。

### 最少一次 (QoS1)

若使用服務品質水準 1 (QoS1)，一律會遞送訊息至少一次。如果在傳送者收到確認通知之前，發生失敗的狀況，可以多次遞送訊息。訊息必須儲存在傳送者本端，直到傳送者收到該訊息是由接收端發佈的確認通知。訊息會儲存起來，以防必須重新傳送訊息。

### 正好一次 (QoS2)

「正好一次」的服務品質水準 2 (QoS2) 最安全，但為最慢的傳送模式。訊息一律遞送正好一次，而且也必須儲存在傳送者本端，直到傳送者收到該訊息已由接收端發佈的確認通知。訊息會儲存起來，以防必須重新傳送訊息。若使用服務品質水準 2，所使用的信號交換和確認通知順序會比水準 1 更準確，以確保訊息不重複。

**提示：**在傳送指令時，如果您想要確認只會執行所指定的指令，而且只會執行一次，請使用服務品質水準 2。這個範例顯示水準 2 的額外需要比起其他水準更有利的情況。

## 訂閱緩衝區及全新階段作業
{: #subscription-buffers-and-clean-session}

無論是裝置或應用程式的每一個訂閱，都配置有 5000 則訊息的緩衝區。緩衝區允許任何應用程式或裝置落後於正在處理的即時資料，而且還可以針對其已建立的每一個訂閱，累積最多 5000 則擱置訊息的待辦事項。緩衝區已滿時，就會在收到新訊息時，捨棄最舊的訊息。

請使用 MQTT 全新階段作業選項來存取訂閱緩衝區。若全新階段作業設為 false，訂閱者會收到來自緩衝區的訊息。若全新階段作業設為 true，緩衝區會重設。

**附註：**不論使用的服務品質設定為何，都會適用訂閱緩衝區限制。如果應用程式跟不上其訂閱的傳訊速率，以水準 1 或 2 傳送的訊息可能不會遞送到該應用程式。

## 訊息有效負載限制
{: #message-payload}

{{site.data.keyword.iot_short_notm}} 支援傳送及接收 MQTT 標準允許之任何格式的訊息。MQTT 不限制資料內容，因此能夠傳送影像、任何編碼的文字、加密的資料，以及幾乎所有類型的二進位格式資料。不過，特定使用案例有某些限制。   

{{site.data.keyword.iot_short_notm}} 上的訊息有效負載也有大小限制。

### 訊息有效負載格式限制

訊息有效負載可以包含任何有效字串，不過，比較常用的格式類型為 JSON ("json")、文字 ("text") 和二進位 ("bin") 格式。

下表概述不同格式類型的訊息有效負載限制：

有效負載格式| 特定使用案例的準則

--------- | ----------  
JSON| JSON 是 {{site.data.keyword.iot_short_notm}} 的標準格式。如果您計劃使用內建的 {{site.data.keyword.iot_short_notm}} 儀表板、板和卡片，以及分析，請確定訊息有效負載格式符合形式完整的 JSON 文字。

文字| 使用有效的 UTF-8 字元編碼。

二進位| 沒有限制。





### 訊息有效負載大小上限

**重要事項：**{{site.data.keyword.iot_short_notm}} 上的有效負載大小上限為 131072 個位元組。有效負載大於限制的訊息將會被拒絕。連線中的用戶端也會斷線，而且診斷日誌中會出現訊息，如下列裝置訊息範例所概述：

`Closed connection from x.x.x.x. The message size is too large for this endpoint.`

## MQTT 保持作用中間隔
{: #mqtt-keep-alive}

MQTT 保持作用中間隔（以秒為測量單位）定義用戶端與分配管理系統之間可持續無通訊的時間上限。MQTT 用戶端必須確保，在沒有與分配管理系統進行任何其他通訊的情況下，會傳送 PINGREQ 封包。保持作用中間隔讓用戶端和分配管理系統都能夠偵測網路失敗，並導致連線中斷，而不需要等到 TCP/IP 逾時。

如果 {{site.data.keyword.iot_short_notm}} MQTT 用戶端使用共用訂閱，保持作用中間隔值就只能設為 1 到 3600 秒之間。如果是要求值 0 或大於 3600 的值，{{site.data.keyword.iot_short_notm}} 分配管理系統會將保持作用中間隔設為 3600 秒。