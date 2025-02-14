從零開始搭建家庭超融合數據中心
============================

故事從去年開始，因為越來越多的數據存儲和AI計算需求，我開始重新組織家裏的計算資源，把零散的計算資源整合成更便於管理和維護的統一資源架構。

在這個過程中，我遇到了很多的問題，之前也在 X 上也和大家分享過一些，大家建議我都寫下來，經過一些考慮，我決定成體系地介紹一下如何一步一步搭建適合家庭網絡規模的超融合數據中心。

這裡面會有很多選型上的考慮，也會有很多配置上的細節，不一定很有含金量，但求給大家一些參考。

我將會從網絡搭建開始，講到存儲，再到運算，因為我覺得考慮 homelab 的配置，應該遵循這一個順序，避免來回折騰，浪費時間。

## 先要有個機櫃

首先要有一個機櫃。選型上，除了通風設計的考慮之外，我認為首先要測量一下計畫存放的房間或者地下室的高度，在能容納的高度範圍內選擇一個最高的。高度決定了這套設備將來的擴展性，不要猶豫，越高越好。

我的地下室有一根懸梁，限制了最大高度，所以只能選擇的最大32U的，具體是這款：StarTech.com 4-Post 32U Server Rack Cabinet (https://www.amazon.ca/dp/B099986PZF?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_5&th=1)。

下面這個表列出了多少 Unit 的櫃子大概有多少高度，範圍從 1U 到 48U，也可以直接在這個頁面上動態估算： https://www.penn-elcom.com/ca/rack-unit-calculator。

| Rack Units | Height (in) | Height (ft) | Height (cm) |
|------------|------------|------------|------------|
| 1U         | 1.75″      | 0.15′      | 4.4 cm     |
| 2U         | 3.5″       | 0.29′      | 8.9 cm     |
| 3U         | 5.25″      | 0.44′      | 13.3 cm    |
| 4U         | 7″         | 0.58′      | 17.8 cm    |
| 5U         | 8.75″      | 0.73′      | 22.2 cm    |
| 6U         | 10.5″      | 0.88′      | 26.7 cm    |
| 7U         | 12.25″     | 1.02′      | 31.1 cm    |
| 8U         | 14″        | 1.17′      | 35.6 cm    |
| 9U         | 15.75″     | 1.31′      | 40 cm      |
| 10U        | 17.5″      | 1.46′      | 44.5 cm    |
| 11U        | 19.25″     | 1.6′       | 48.9 cm    |
| 12U        | 21″        | 1.75′      | 53.3 cm    |
| 13U        | 22.75″     | 1.9′       | 57.8 cm    |
| 14U        | 24.5″      | 2.04′      | 62.2 cm    |
| 15U        | 26.25″     | 2.19′      | 66.7 cm    |
| 16U        | 28″        | 2.33′      | 71.1 cm    |
| 17U        | 29.75″     | 2.48′      | 75.6 cm    |
| 18U        | 31.5″      | 2.63′      | 80 cm      |
| 19U        | 33.25″     | 2.77′      | 84.5 cm    |
| 20U        | 35″        | 2.92′      | 88.9 cm    |
| 21U        | 36.75″     | 3.06′      | 93.3 cm    |
| 22U        | 38.5″      | 3.21′      | 97.8 cm    |
| 23U        | 40.25″     | 3.35′      | 102.2 cm   |
| 24U        | 42″        | 3.5′       | 106.7 cm   |
| 25U        | 43.75″     | 3.65′      | 111.1 cm   |
| 26U        | 45.5″      | 3.79′      | 115.6 cm   |
| 27U        | 47.25″     | 3.94′      | 120 cm     |
| 28U        | 49″        | 4.08′      | 124.5 cm   |
| 29U        | 50.75″     | 4.23′      | 128.9 cm   |
| 30U        | 52.5″      | 4.38′      | 133.4 cm   |
| 31U        | 54.25″     | 4.52′      | 137.8 cm   |
| 32U        | 56″        | 4.67′      | 142.2 cm   |
| 33U        | 57.75″     | 4.81′      | 146.7 cm   |
| 34U        | 59.5″      | 4.96′      | 151.1 cm   |
| 35U        | 61.25″     | 5.1′       | 155.6 cm   |
| 36U        | 63″        | 5.25′      | 160 cm     |
| 37U        | 64.75″     | 5.4′       | 164.5 cm   |
| 38U        | 66.5″      | 5.54′      | 168.9 cm   |
| 39U        | 68.25″     | 5.69′      | 173.4 cm   |
| 40U        | 70″        | 5.83′      | 177.8 cm   |
| 41U        | 71.75″     | 5.98′      | 182.2 cm   |
| 42U        | 73.5″      | 6.13′      | 186.7 cm   |
| 43U        | 75.25″     | 6.27′      | 191.1 cm   |
| 44U        | 77″        | 6.42′      | 195.6 cm   |
| 45U        | 78.75″     | 6.56′      | 200 cm     |
| 46U        | 80.5″      | 6.71′      | 204.5 cm   |
| 47U        | 82.25″     | 6.85′      | 208.9 cm   |
| 48U        | 84″        | 7′         | 213.4 cm   |

需要注意的是，櫃子會有上蓋和底座，有一些底座下面還有輪子，所以一定要根據自己的實際需求來選擇，並且多做幾次測量避免尷尬，因為要退換的話，將會非常麻煩，一個比較正式的幾架，是相當重的。

## 網絡規劃

### 網絡設備選型

網絡設備的選擇上，首先要考慮的是速率上的規劃。我建議所有用戶，都從 10GE 開始考慮，低於 10GE 的設備，強烈不建議購買了。有條件的話，40GE 或者 100GE 都可以考慮。

另一點，就是考慮傳統 RJ45 電口還是 SFP/SFP+/SFP28/QSFP+/QSFP28 等光模塊接口。

在這個問題上，我考慮了很久，主要的原因是，我已經有一部分的設備在很早的時候，就已經是 10GE Ethernet 接口的，如果我選擇 SFP+ 路線，我將無法兼容已有的設備。但是 10G 以上電口在我之前的使用中，的確遇到一些問題，網卡的載荷比較大，發熱比較高。我的確想趁機會演進到光口路線。

我花了很多時間來抉擇，最後的決定是，都要有，事實上到目前為止，我依然覺得這是一個很好的決定，我目前的網絡容量讓我在後期的工作中獲得了很大的彈性，並預留了足夠的性能空間。

### 選擇一個交換機

正如我前面說的，我需要有足夠的電口和光口，可管理，可堆疊，可擴展，高性能，穩定可靠的交換機。我最終的選擇是：

#### UniFi USW-Pro-Aggregation 作為聚合交換機

我選擇這款路由器作為核心網的基礎。它有 32 個 28 個 10G SFP+ 接口和 4 個 25G SFP28 接口，交換容量是 760 Gbps，非阻塞 IO 能到 380 Gbps，支持 VLANs 和鏈路聚合，Layer 3 和 Layer 2 都是可管理的。

https://ca.store.ui.com/ca/en/products/usw-pro-aggregation

#### UniFi USW-EnterpriseXG-24 作為以太網交換機

我使用這一款交換機來兼容原有的設備，支持 Wi-Fi 熱點，遠程 KVM 的應用。它有 24 個 10 GbE RJ45 電口和 2 個 25G SFP28 光口，交換容量是 580 Gbps，非阻塞 IO 能到 290 Gbps，同樣支持 VLANs， Layer 3 和 Layer 2 可管理。

https://ca.store.ui.com/ca/en/products/usw-enterprisexg-24

### 選擇入口路由，防火牆

根據自己的需要選擇一個入口路由，或者防火牆路由來接入網絡，這需要考慮的主要是，它的 Uplink 能不能滿足你家的網絡接入速率，Downlink 能不能滿足你核心交換機的吞吐。我選的是 Dream Machine Special Edition (UDM-SE (180W))。

這款路由器一個 10G SFP+ Uplink 作為主要入口，一個 2.5 GbE RJ45 Uplink 作為備用網絡入口，有 1 個 10G SFP+ Downlink 和 8 個 GbE RJ45 Downlink，其中 2 個電口支持 PoE+， 其餘 6 個是電口都支持 PoE。

它唯一讓我不滿意的是 Uplink 只開放了 3.5Gbps，雖然對於我只有 3Gbps 對等光線接入的用戶來說是足夠的，但是總覺得餘量不足，比較懊悔的是，我下單之後不久，Dream Machine Pro Max (UDM-Pro-Max) 就上市了，它的 Uplink 能到 5 Gbps，就好不少。

### 關於 Wi-Fi

順帶說一下無線網絡的延伸。我是多年的 Google Wi-Fi (Nest Wi-Fi) 用戶，趁著網絡的改造，我也順便換上了支持 Wi-Fi 6 的 UniFi AP 6 Lite，這款 Wi-Fi 7 的 U7-Pro 熱點。當然，主要的考慮是更便於整合到 Unifi 的生態，簡化管理。但是體驗下來，Wi-Fi 7 的確對於經常需要視頻通話和串流遊戲帶來了更好的體驗。

### 組建網絡

在所有的硬件都到位之後，我開始組建網絡。在把機器插入機櫃之前，需要大致考慮好機器的擺放順序，一般來說按照以下一些準則：

- 重的機器往下放，主要是存儲和 UPS 等；
- 輕的機器往上放，主要是電源管理，網絡設備，輔助設備，KVM 等；
- 太熱的機器不要放在一起，要有間隔，盡可能讓熱的機器靠近風扇防止機機積熱；
- 根據自己的身高和維護習慣安排 KVM 的位置，大概率可能會被安排在機櫃的中上部分；

盡可能多安排風扇，保持通風，這樣可以大大減少機器因為積熱帶來的穩定性問題。通風的設計上，應該是從下往上，從前往後，比較細緻的可以上控溫設備，我還沒做，目前是保持風險以最大功率運行。

這裡有一個比較理想情況下的單櫃排列策略圖：

https://lucid.app/lucidchart/c67227d4-dbd8-4164-9005-24aabf8a3765/view?anonId=0.07fd85c819500992984&sessionDate=2025-02-13T18%3A36%3A39.778Z&sessionId=0.e8ca132219500992985&fromMarketing=true&page=0_0#

但是需要注意的是，其實每個櫃子的實際情況都是不一樣的，非一體化設計的機櫃單元，很多時候會因為線纜長度的制約，走線的需要，機器的尺寸重量，甚至個人的喜好而有所調整，這都很常講，事實上隨著業務的變化，我也調整過幾次才達到滿意的效果。

在大致決定好設備的位置之後，就可以開始上架機器和接線了。我有三路網絡入口，主網絡是 Bell Fibe 3Gbps 的對等光纖，主備線路是 Starlink，次備是 Telus 的無限流 5G+ 蜂窩網，次備網絡是通過 Nighthawk M5 5G WiFi 6 Mobile Router(MR5200) 實現的，這款路由可以擴展天線，支持 5G Sub-6 band，1.8Gbps Wi-Fi 6，最重要的是有一個 Gigabit Ethernet WAN 口，這樣可以直接接入網關，作為網用網絡入口。

簡單展示一下組網的拓撲：

```
+----------------------------------------------------+
|                       INTERNET                     |
+----------------------------------------------------+
        |                   |                  |
+----------------+ +------------------+ +------------+
| Bell Fibe (3G) | | Starlink (Bak 1) | | 5G (Bak 2) |
+----------------+ +------------------+ +------------+
        |                   |                  |
       10G                2.5G                1G
        |                   |                  |
+----------------------------------------------------+
|           Dream Machine Special Edition            |
+----------------------------------------------------+
                            |
                           10G
                            |
+----------------------------------------------------+
|                  USW-Pro-Aggregation               |
+----------------------------------------------------+
                            |
                        50G (25G * 2)
                            |
+----------------------------------------------------+
|                  USW-EnterpriseXG-24               |
+----------------------------------------------------+
```


## 存儲規劃
