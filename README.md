從零開始搭建家庭超融合數據中心
============================

故事從去年開始，因為越來越多的數據存儲和AI計算需求，我開始重新組織家裏的計算資源，把零散的計算資源整合成更便於管理和維護的統一資源架構。

在這個過程中，我遇到了很多的問題，之前也在 X 上也和大家分享過一些，大家建議我都寫下來，經過一些考慮，我決定成體系地介紹一下如何一步一步搭建適合家庭網絡規模的超融合數據中心。

這裡面會有很多選型上的考慮，也會有很多配置上的細節，不一定很有含金量，但求給大家一些參考。

我將會從網絡搭建開始，講到存儲，再到運算，因為我覺得考慮 homelab 的配置，應該遵循這一個順序，避免來回折騰，浪費時間。

## 什麼是超融合

在傳統的數據中心，服務器節點一般分為三大類別，網絡服務器，存儲服務器，應用服務器。

- 網絡服務器包括光電線路的路由器，防火牆，交換機，IoT 網關等等。
- 存儲服務器往往以集群的方式存在，對應用服務器提供冷熱數據的訪問，二進制數據和結構化數據的存儲、備份等等，有多方式實施，如 NAS，SAN，DAS 等型態。
- 應用服務器承擔著要的計算任務，包括 CPU 為主的運算以及新興的 GPU 為主的運算，提供的服務包括 Web 服務，郵件服務，虛擬機，容器集群，密集計算服務等等。

超融合 HCI (Hyper-Converged Infrastructure) https://en.wikipedia.org/wiki/Hyper-converged_infrastructure 的概念比較新，它是隨著虛擬化的發展而提出來的一重新組織服務器資源的策略，也是所謂軟件定義數據中心的核心概念之一。
超融合數據中心將網絡，存儲，運算三種資源融合在一起，形成一個統一的計算資源池，完全由虛擬化平台提供各種服務和計算、存儲資源。
在一個超融合計算中心裡面，除了物理主機的網絡連接需要單獨的網絡設備之外，其他的計算資源都可以通過虛擬化平台來提供，每個物理主機都是全能節點，承擔部分的計算和存儲，他們之間緊密相連，共享資源，相互備援。
超融合不是一個特定的單一軟件技術，它其實脫胎於虛擬化，分佈式存儲等等一系列技術上的一個概念，在實施上，常見的超融技術棧有 Proxmox VE + Ceph, VMware ESXi, Hyper-V + S2D 等等。本文以 Proxmox VE + Ceph 為例子，介紹如何一步一步搭建一個融單櫃的超融合數據中心。
當然了，說數據中心是一個誇大的概念，但是這個過程的確已經覆蓋了搭建一個小型數據中心所需要的常見類別的設備和比較完整的步驟。

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

我們接下來會挑選一些設備，從網絡設備出發，但是對於後面各種類型的設備在插入機櫃之前，都需要大致考慮好機器的擺放順序，所以我們先簡單了解一下上架機器的一些基本準則：

- 重的機器往下放，主要是存儲和 UPS 等；
- 輕的機器往上放，主要是電源管理，網絡設備，輔助設備，KVM 等；
- 太熱的機器不要放在一起，要有間隔，盡可能讓熱的機器靠近風扇防止機機積熱；
- 根據自己的身高和維護習慣安排 KVM 的位置，大概率可能會被安排在機櫃的中上部分；

盡可能多安排風扇，保持通風，這樣可以大大減少機器因為積熱帶來的穩定性問題。通風的設計上，應該是從下往上，從前往後，比較細緻的可以上控溫設備，我還沒做，目前是保持風險以最大功率運行。

這裡有一個比較理想情況下的單櫃排列策略圖：

https://lucid.app/lucidchart/c67227d4-dbd8-4164-9005-24aabf8a3765/view?anonId=0.07fd85c819500992984&sessionDate=2025-02-13T18%3A36%3A39.778Z&sessionId=0.e8ca132219500992985&fromMarketing=true&page=0_0#

但是需要注意的是，其實每個櫃子的實際情況都是不一樣的，非一體化設計的機櫃單元，很多時候會因為線纜長度的制約，走線的需要，機器的尺寸重量，甚至個人的喜好而有所調整，這都很常講，事實上隨著業務的變化，我也調整過幾次才達到滿意的效果。

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

### 網絡組建

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

在網絡安排好之後，我建議大家先考慮存儲，存儲決定了這套設備能處理的數據的極限，這可能包括你工作需要存儲的數據，例如視頻工作者，AI 工程師等等往往都對存儲空間有較高的需求。對於 homelab 來說，可能還有媒體服務，遊戲串流服務，相冊服務等等一些個人數據的存放。
坦白說，在有條件的情況下，盡可能多安排存儲，會少走很多彎路。存儲池我個人建議至少安排一個 SSD 高速池用來支持虛擬化，數據庫，AI 運算等等，和一個 HDD 的大容量存儲池，用來存儲備份數據，非活躍項目的大體積數據等等。我個人推薦存儲方案的選擇應該滿足兩個條件：穩定是大前提，靈活可伸縮相當重要。
在花了很長時間的折騰之後，我最終的方案是：

- 高速存儲池通過 Ceph 實現，全 SATA SSD + NVME SSD，單盤 4TB * 4 盤 * 4 節點*；
- 大容量存儲池通過 Mdadm RAID 6 實現，全 HDD，單盤 18T *  10 盤；

其中的 Ceph 是超融合重要的一環，後面會詳細解釋。至於大容量的存儲池為什麼選擇 Mdadm 而不是 ZFS 或者 LVM，這個主要看個人的喜好。

我對穩定性和靈活性都有很高的要求，ZFS 的確很強大，但是對於陣列的伸縮性比較侷限，不能滿足我喜歡折騰的喜好。LVM 對快照的支持比較差，而且還有不少其他問題尚未解決，例如內建 RAID 的課恢復性等等。

所以我還是選擇了在 Linux 上廣泛使用，穩定性無可挑剔的 Mdadm，在上面使用 Btrfs 文件系統來支持快照和文件校驗，有意思的是，這個組合恰好是 Unifi UNAS Pro 所使用的方式。當然，這個主要看個人喜好，對擴展性，靈活性，穩定性，容災重建等等每個人的理解都不一樣，選擇自己喜歡的，不要陷入神仙打架的局面。

### Ceph

這裡重點所一下 Ceph，這是開源超融合方案的核心組件，如果你搭配 PVE 使用的話，選擇 Ceph 作為存儲池，可以獲得非常好的性能，擴展性和穩定性，同時 PVE 面板上整合的 Ceph 管理工具相當簡單實用，可以快速管理全局 Ceph 集群健康狀態，簡單好用。當然 PVE 對 Ceph 的集成雖然已經很好，但是 Ceph 其實是一套十分強大和複雜的分佈式存儲系統，很多高級操作依然需要命令行，這個無可避免。

為什麼說 Ceph 是超融合方案的核心？前文已經提到，在超融合數據中心裡面，說有的資源是充分共享和網絡化的。你可以理解，你的業務不是運行在單一機器，或者單一個虛擬機裡面，粗略又抽象地理解，你的服務是跑在這個集群上。這個集群一起來支撐你的業務，機器之間緊密相連，相互備援。在 PVE 正確配置 Ceph 作為存儲後端之後，你的虛擬機是可以在集群裡面隨意遷移的，一個 VM 可以在線不關機的情況下，從一個主機直接遷移到另一個主機上運行，對外甚至是無感知的。

這帶來了很多便利，例如一台機器故障或者需要維護重啟，你完全不用擔心上面的業務，PVE 可以自動把所有運行中的 VM 無縫遷移到集群中其他機器上運行。

這種無縫的遷移依賴這個集群把存儲視為一個整體，從不同的機器，都可以訪問到一樣的數據。PVE 當然也支持通過 SAN 網絡或者 NFS 等傳統方案來實現統一的存儲，但是顯然 Ceph 是更面向未來的選擇。

- 有幾點需要強調一下，為了獲得足夠理想的 IO 性能，全 SSD 或者 NVME 配置應該是優先考慮的；
- 為了獲得足夠的穩定性和冗余，個人推薦從 4 個節點起步，size 4，min 2 應該是最低的安全保障，這相當於集群中每一份數據保存在至少兩個節點上，至少滿足兩個節點在線的時候允許讀寫，掛掉兩個節點不影響業務。

## 安裝虛擬化平台 PVE

Proxmax VE 基於 Debian，這其實也是我選擇它的原因之一，我是多年 Debian 用戶，對於 Debian 的穩定性相當有信心。它唯一的缺點是保守，Stable 分支的軟件包相對較舊，需要嘗新可以用其他分支，但是我的理解是宿主機的穩定性是最重要的，嘗新完全可以在 VM 中折騰。

事實上我推薦的安裝方式並不是直安裝 PVE，而是先安裝 Debian，然後安裝 PVE 軟件包。這個主要的原因在於，我在幾個服務器上，嘗試直接安裝 PVE 都失敗，可能是我環境的問題，不深究，但是從 Debian 上安裝，表現更符合預期。

- 直接安裝 PVE：https://www.proxmox.com/en/products/proxmox-virtual-environment/get-started
- 從 Debian 安裝：https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm

PVE 的官方文檔上要求 Debian 版本是 12 Bookworm，事實上大家可以放心直接在 Debian stable 分支上運行。有一些包比 PVE 預設的要更新，但是沒有任何兼容性問題。

簡單總結一下安裝的步驟：

- 安裝 Debian 12 Bookworm，切換到 stable 分支；

- 在 /etc/host 中加入當前主機的 IP 地址

```/etc/host
[本機本地 IP]    [當前主機名]
```
例如：
```/etc/host
192.168.1.64    Enlightenment
```
確保地址正確：
```
hostname --ip-address
```
確保返回的地址是 /etc/host 中配置的地址，而不是 127.0.0.1 或者 ::1 等。

- 添加 PVE 的 apt 源：

```
# echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
```

- 添加 PVE 的密鑰：

        ```
        wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
        # verify
        sha512sum /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
        7da6fe34168adc6e479327ba517796d4702fa2f8b4f0a9833f5ea6e6b48f6507a6da403a274fe201595edc86a84463d50383d07f64bdde2e3658108db7d6dc87 /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
        ```

- 更新 apt 源，升級系統：

        ```
        apt update && apt upgrade -y
        ```

- 安裝 PVE 內核 ：

        ```
        apt install proxmox-default-kernel
        systemctl reboot
        ```

- 安裝 PVE 套件：

        ```
        apt install proxmox-ve postfix open-iscsi chrony
        ```

- 刪除 Debian 默認內核：

        ```
        apt remove linux-image-amd64 'linux-image-6.1*'
        update-grub
        ```

- 刪除 os-prober，避免虛擬機被錯誤加到引導菜單：
```
apt remove os-prober
```
- 刪除企業版 apt 源，僅使用 開源版組件（如果你打算購買企業版，忽略此步驟）：
```
# rm /etc/apt/sources.list.d/pve-install-repo.list
```
- 重啟，確保可以正常引導，如果你的 Debian 是啟動到命令行的，你應該會看到 PVE 的歡迎介面，如果你是啟動到圖形介面的，你不會看到任何變化，可以執行以下命令查看 PVE 是不是已經正確安裝：
```bash
pveversion
```
如果看到類似以下的輸出，說明 PVE 已經正確安裝：
```
pve-manager/8.3.3/f157a38b211595d6 (running kernel: 6.11.0-1-pve)
```
注意，如果無法看到 PVE 歡迎介面，先檢查一下上面 IP 地址的步驟。
- 訪問 PVE 管理面板：
```
https://[本機本地 IP]:8006
```
例如：
```
https://192.168.1.64:8006
```
注意，這裡是 https，然後需要瀏覽器忽略一下自簽證書的安全檢查。

### 配置網橋，確保虛擬機能訪問網絡

詳情可以參考這裡：https://pve.proxmox.com/wiki/Network_Configuration

簡單來說，你需要配置一個網橋，便於虛擬機可以方便橋接到主網絡中：

```/etc/network/interface
auto lo
iface lo inet loopback

iface eno1 [網卡的設備號，如 eno1, enp1s0 等等，通過 ifconfig -a 查看] manual

auto vmbr0
iface vmbr0 inet static
        address [配置為上面步驟中的 IP 地址]/24 (一般是這個網段，如果需要其他網段，可以自行修改，也可以配置 netmask 來配置) // 待定細節
        gateway [配置為上面路由器地址]
        bridge-ports [網卡的設備號，如前所述]
        bridge-stp off
        bridge-fd 0
```

### 配置鏈路聚合

如果你的交換機支持 LACP，應該配置鏈路聚合，這樣可以提高網絡的穩定性和帶寬，對集群的穩定性，特別是 Ceph 的穩定性有很大的幫助。

```
╰ $ > cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto enp130s0f4
iface enp130s0f4 inet manual

auto enp130s0f4d1
iface enp130s0f4d1 inet manual

auto enp130s0f4d2
iface enp130s0f4d2 inet manual

auto enp130s0f4d3
iface enp130s0f4d3 inet manual

auto bond0
iface bond0 inet manual
    bond-slaves enp130s0f4 enp130s0f4d1 enp130s0f4d2 enp130s0f4d3
    bond-mode 802.3ad
    bond-xmit-hash-policy layer2+3
    bond-miimon 100
    bond-downdelay 200
    bond-updelay 200
    bond-lacp-rate 1

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.204
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge-ports bond0
    bridge-stp off
    bridge-fd 0
```

測速，確保聚合有效

```
待補充
```



配置完成後，重啟網絡：
```
systemctl restart networking
```

### 開啟 Jumbo Frame

Jumbo Frame 是 10GE 對於高速網絡至關重要，更大的包可以減少包的數量，降低 CPU 在封包和解包過程的負擔，顯著提升網絡的吞吐量。

```bash
# /usr/bin/ip link set [port] mtu 9000
```

根據需要修改 `[port]` 為你的網絡接口名稱，如果單網卡，在 PVE 環境應該修改 `vmbr0` 的 MTU：

```bash
# /usr/bin/ip link set vmbr0 mtu 9000
```

如果你有使用上面的 LACP 鏈路聚合，這裡需要使用聚合的接口名稱，例如：

```bash
# /usr/bin/ip link set bond0 mtu 9000
```

你可以把類似的配置放在 `/etc/network/interfaces` 中，也可以直接把上面的命令放在 crontab 通過 `@reboot` 和 `sleep` (等待網絡接口初始化完成) 來執行，完全是個人喜好，此處不做贅述。

注意！請保證你的每個節點，使用相同的 MTU 包尺寸設置，這將是集群穩定性的關鍵，如果 MTU 不一致，PVE 集群會出現不穩定，某個節點突然斷開的情況。對於 Ceph 更是如此，不一致的 MTU 會導致 OSD 心跳受阻，進而導致 OSD 頻繁從集群中離線。我有一個機器出現這個情況花了很長的時間才發現這是一個致命的問題。

### 配置顯卡直通

如果有 AI 運算需求，遊戲需求或者視頻編輯需求，顯卡直通是必不可少的，這裡面往上有很多的介紹，有對有錯，這裡簡單總結一下步驟：

- BIOS 中開啟 IOMMU，這一點尤其重要：
有一些主板是默認關閉的，也有一些主板是默認開啟的，需要手動確認一下：
```
dmesg | grep -e DMAR -e IOMMU
```
確保輸出中有：
```
DMAR: IOMMU enabled
```
- 確保 IOMMU interrupt remapping 可用：

```
dmesg | grep 'remapping'
```
確保輸出中有：
```
AMD-Vi: Interrupt remapping enabled
```
或者:
```
DMAR-IR: Enabled IRQ remapping in ******* mode
```
如果找不到，可以嘗試強制開啟：
```
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
```
確保 /etc/modules 中有以下的條目：
```
vio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
如果計畫需要更好虛擬 macOS，運行：
```
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```
- 在啟動的時候，讓宿主機忽略需要分配給 VM 的顯卡：
不加載 AMD 顯卡的驅動：
```
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
```
不加載 NVIDIA 顯卡的驅動：
```
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia*" >> /etc/modprobe.d/blacklist.conf
```
不加在 Intel 顯卡的驅動：
```
echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf
```
- 更新啟動鏡像：
```
update-initramfs -u
```

做以上操作之後，大部分的 AMD 顯卡 NVIDIA 顯卡和 Intel 顯卡都可以直通給 VM 使用了，這裡需要注意的是，如果宿主機需要 GUI 的話，不要忽略 Intel 的顯卡驅動，這樣可以保留板載顯卡用來良機和調試。

如果上面的操作依然出現問題，個人建議換支持度更高和更新的顯卡，雖然網上有很多奇技淫巧可以一定程度解決問題，但是穩定性和兼容性很難保證，同時需要額外的 hack 來解決的顯卡一般都比較老了，算力上其實意義不大，可以趁機升級了。

更多信息可以參考這裡：https://pve.proxmox.com/wiki/PCI_Passthrough


## PVE 集群配置

基本上每一個 PVE 節點都需要經歷上面的配置，在完成單獨的配置之後，可以開始把 PVE 節點組合成集群。

注意，這裡強調一下，為了穩定可用，集群應該保證最少 3 個節點，考慮到後面需要 Ceph 的話，最少需要 4 個節點更為合適。兩個節點，當其中一個節點 down 掉之後，會有投票無法過半數的問題，需要手動修改投票規則才能啟動虛擬機。

在組合成集群之前，先確定是那個節點加入到那個節點，這裡暫且叫做主從節點，但是其實 PVE 並沒有主從的概念，集群中的主機是通過 quorum 投票來取的一致性的。確定誰加入誰的一個原因是，作為從節點，也就是新加入集群的節點，是不能有虛擬機的，如果有，需要先備份並銷毀，因為 PVE 中的 VM id 是全局唯一的，所以已經有 VM 的機器是不允許加入集群的。

在“主”節點上創建集群：

```
pvecm create [CLUSTERNAME]
```

在“從”節點上加入集群：
```
pvecm add [IP-ADDRESS-CLUSTER]
```

查看集群的狀態：
```
╰ $ > sudo pvecm status
[sudo] password for leask:
Sorry, try again.
[sudo] password for leask:
Cluster information
-------------------
Name:             Dream
Config Version:   8
Transport:        knet
Secure auth:      on

Quorum information
------------------
Date:             Sun Feb 16 03:32:55 2025
Quorum provider:  corosync_votequorum
Nodes:            4
Node ID:          0x00000001
Ring ID:          1.b1f
Quorate:          Yes

Votequorum information
----------------------
Expected votes:   4
Highest expected: 4
Total votes:      4
Quorum:           3
Flags:            Quorate

Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 192.168.1.204 (local)
0x00000002          1 192.168.1.201
0x00000003          1 192.168.1.202
0x00000004          1 192.168.1.79
```

查看集群節點和投票權重：
```
sudo pvecm nodes

Membership information
----------------------
    Nodeid      Votes Name
         1          1 Enlightenment (local)
         2          1 Juan
         3          1 Rio
         4          1 Rubao
```

順帶說一下，如果需要移除一個節點的話，可以用這個命令：
```
pvecm delnode [NODE-NAME]
pvecm delnode hp4
```


## Ceph 配置

一句話概括 Ceph 是什麼：一種分佈式的網絡存儲，aka 跨機器的高性能高可用性的磁盤陣列。

下面介紹一下 Ceph 的一些基本概念：

- Ceph 的集群由 Monitor，OSD，MDS 組成。
- Ceph 通過幾種常見的方式來提供存儲：
    - RBD (Ceph Block Device)：塊設備，提供塊級別的存儲，可以作為虛擬機的磁盤使用。
    - RGW (Ceph Object Gateway)：對象存儲，提供對象級別的存儲，可以作為對象存儲使用，可直接導出為 S3 兼容的對象存儲，NFS 兼容的文件系統，可以作為文件系統使用。
    - CephFS (Ceph File System)：POSIX 兼容的文件系統，可以作為文件系統使用。
- Monitor 負責集群的元數據管理，包括集群的拓撲結構，節點的健康狀態等。
- OSD (Object Storage Daemon) 負責數據的存儲和檢索，每個機器的每一塊物理存儲，都需要配置一個 OSD，他們各自獨立，互不干擾，只對 Monitor 負責。
- MDS (Metadata Server) 負責文件系統的元數據管理，如果需要使用 CephFS 的話，需要配置 MDS，CephFS 服務的 overhead 比較大，根據自己的需要開啟，但是作為跨節點的文件共享服務相當好用，我推薦大家嘗試一下。
- PG (Placement Group) 是 Ceph 的數據邏輯管理單元，Ceph 通過 PG 來管理數據的分配，每個 OSD 通常會包含若干個 PG，每個 PG 會根據 CRUSH map 的策略，在不同 OSD 上放置副本，對於每個存儲池需要切割成多少個 PG，後面會介紹。

Ceph 存儲池的重要參數：

- 副本數：副本數表示數據的冗餘度，默認是 3，表示數據有 3 份，分佈在不同的節點上，這個和你有多少節點沒關係，你有 5 個節點，如果副本數是 3，那麼它以然只把數據放到 3 個節點上；
- 最小副本數：最小副本數表示數據的最小可用冗餘度，默認是 2，表示數據至少需要 2 份副本這份數據才可以被讀寫。

Ceph 是副本冗余，它和 RAID 有一點像，但是並不完全一樣，更接近 Btrfs 的冗餘設計。

### 安裝 Ceph 所需要的軟件包。
Ceph 的配置很簡單，直接在 PVE 的 Web 界面中安裝最簡單。它會自動處理 Ceph 的集群和密鑰配置，不需要自己操心。
最簡單的方法是照著這個視頻的前 9 分鐘，做一次，一般十幾分鐘可以完成： https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/install-ceph-server-on-proxmox-ve

### Ceph 高可用的前提條件

- 穩定快速的局域網是 Ceph 的基礎，如果局域網不穩定，Ceph 的穩定性和性能都會受到很大的影響，這就是為什麼建議 10GE 起步的原因。
- 過得去的 CPU，Ceph 很多操作都需要跨節點校對，因為在超融合框架中，你的 CPU 還需要承擔其他的業務，所以，一定的空閒核心數比較重要，但是可以放心，現代的消費級別的中高端 CPU 完全可以滿足需求。
- 需要特別注意的，其實是內存，對於高性能的網絡存儲來說，因為需要緩存更多的數據，特別是元數據的映射信息。至少需要為 Monitor 進程預留內存至少需要 4GB 內存，更大的集群可能需要 8GB，16GB 甚至更多。
- 對於存儲節點，根據每個 OSD 所管理的數據的大小，理想的情況下，1TB 數據需要 1GB 的內存以滿足各種訪問的需求，例你節點上有 10TB 的SSD，這個節點就需要預留 10GB 的內存來保障集群的效率。


### 關於 Monitor 節點的個數選擇

- Monitor 節點的個數至少需要 3 個起步，這是因為 Ceph 的集群需要通過 paxos 協議來取得一致性，所以至少需要 3 個節點確保可用，它具體的規則是：
```
floor(N/2 + 1)
```
如果是 2 個節點，那麼 `floor(2/2 + 1) = 2`，也就是說，任何一個節點掛掉，都沒辦法取得一致，集群會卡住，一直等到故障節點重新上線才會恢復運作。
- 最佳實踐是選擇奇數作為節點數，避免因為腦裂等原因導致重組之後投票失敗，對於家用集群，3 個節點已經足夠了。5 個節點可以提供多掛掉一台機器的保障，但所增加的開銷對於，家庭規模集群可能不划算，因為5個節點會導致網絡上心跳包的開銷更大，對整體網絡性能的要求會更高。
- Monitor 節點是不是可以和 OSD 節點放在一起？其實是可以的，特別是在 homelab 環境，節點數相對較少的集群來說，十分實用，但是對於大規模的集群，還是應該獨立出來。

### 關於 MDS 節點的個數選擇

- CephFS 至少需要 1 個 MDS 節點來提供元數據服務來支持 POSIX 兼容的文件系統。
- MDS 服務器個數的選擇上，不需要考慮一致性投票的問題。但是需要注意的是在小型集群裡面，Ceph 只會在一個時間內選擇激活一個 MDS 節點，所以更多的節點並不會帶來直接的性能提升，但是會帶來可用性冗余，因為需要考慮如果 MDS 所在的節點 Down 掉或者需要維護，集群裡面的數據以然可以被訪問，我自己的選擇是在每個節點上都開啟了 MDS，這是一個國度部書的選擇，但是這樣可以讓我隨便下線節點來維護，相當方便。

### OSD 是不是需要統一大小的磁盤？

並不是，這也是大部分所謂科普文章的錯誤所在，事實上組成存儲陣列的所有磁盤大小並不重要，但是，應該保證的是一個節點多有磁盤加起來大致容量是相近的。否則會造成大容量節點還沒滿的時候，小容量磁盤就會報警寫滿，造成陣列無法繼續寫入。

對於尺寸不想等的磁盤，其實可以通過 reweight 來調整，這樣可以讓大容量磁盤承擔更多的數據，小容量磁盤承擔更少的數據，這樣可以保證每個磁盤的利用率大致相同。

臨時 reweight 的命令，其中 [osd_id] 是 OSD 的 ID，如對於 `osd.1`，只需要輸入 `1` 即可，[reweight_value] 是重權值，默認是 1，越大表示承擔的數據越多，有效範圍是一個 0 到 1 之間的浮點數，注意直接執行 reweight 命令是一次性的，當磁盤重新上線 (in) 的時候，reweight 的值會被重置為 1：

```
ceph osd reweight [osd_id] [reweight_value]
```

需要持久化上main的配置，可以通過以下命令寫入 CRUSH map，其中 [osd_name] 是 OSD 的名字，和上面的 `OSD ID` 不一樣，它包含前面的 `osd.` 前綴，例如 `osd.1`，[reweight_value] 是重權值，默認是 1，越大表示承擔的數據越多，有效範圍是一個 0 到 1 之間的浮點數，這裡提到的 CRUSH map 是 Ceph 的核心配置，是 Ceph 的數據分佈策略，後面我會詳細介紹一下：

```
ceph osd crush reweight [osd_name] [reweight_value]
```

自動根據 osd 的利用率來調整 reweight 的值，這樣可以保證每個磁盤的利用率大致相同，[threshold] 是可選的閾值，默認是 120，表示如果磁盤的利用率超過集群平均利用率的 120%，則進行 reweight：

Dry run:
```
ceph osd test-reweight-by-utilization [threshold]
```

Apply:
```
ceph osd reweight-by-utilization [threshold]
```

自動根據 PG 數量來調整 reweight 的值，這樣可以保證每個磁盤的 PG 數量大致相同，[threshold] 是 PG 的門檻值，默認是 120，表示如果磁盤的 PG 數量超過集群平均 PG 數量的 129%，則進行 reweight：

Dry run:

```
ceph osd test-reweight-by-pg [threshold]
```

Apply:
```
ceph osd reweight-by-pg [threshold]
```

### Ceph 的空間計算

為什麼 Reweight 很重要，因為它會影響 Ceph 集群的實際可用空間，下面我一步步來介紹 Ceph 如何計算空間。

Ceph 如何計算空間，由於 Ceph 的副本冗餘設計，所以 Ceph 集群的理論可用空間是通過以下公式計算出來的：

```
Total_space = sum(OSD_Capacity) / size
```

也就是所有 OSD 的容量加起來 / 副本數。

對於均勻集群，也就是說，每個服務器的磁盤大小一致，每個服務器的磁盤數量也一致，上面的公式等價於：

```
Total_space = OSD_Capacity * OSD_Count / size
```

所理論上，集群認為每個節點能寫入的容量是一樣的，在默認所有節點的 weight 都是 1 的情況下，Ceph 在請求 OSD 的時候，會均勻分佈每個節點每個 OSD 均等的機會來寫入數據。

這就造成一個問題，如果節點不均勻，某些小磁盤會更快寫滿，從而導致集群停止接受寫入任務。Reweight 可高容量的 OSD 接受更多的寫入任務，低容量的 OSD 減少寫入任務，更充分使用高容量 OSD 的空間，提高集群總體的可寫入容量。

在 PVE 的 Ceph 管理介面，我們能看到的集群容量，就是上面公式計算的理論容量。那麼已經寫入的數據體積，是如何計算的呢？為什麼我只啟動了一個 1TB 的虛擬機，但是卻佔用了幾 TB 的容量？這是因為 Ceph 中，空間的計算是考慮了副本數量的：

```
Used_space = sum(data_size) * size
```

也就是所有數據的體積加起來 * 副本數。

這和 RAID 不一樣，RAID 的容量是磁盤 RAID 之後，實際可以使用的容量，而不是裸磁盤的容量，RAID 查看數據的體積是數據的實際體積，而不考慮冗余和糾錯。

### 關於 PG 的個數選擇

PG 數量會影響 Ceph 的性能和數據分佈的均勻程度。

同等數據體積下，PG 數量越多，數據越容易均勻分佈，分佈式訪問數據的時候理論性能更高，但是過多的 PG 會導致元數據過大，從而影響性能。

反過來如果 PG 數量過小，數據分佈不均勻，會導致某些 OSD 負擔過重，從而影響集群的穩定性。

每個存儲池需要多少個 PG，通常可以通過以下公式來計算：

```
PG = (OSD * [OSD_TARGET_PG_NUM, best practice is 100] ) / (pool_num *  size)
```

然後一般會取一個最近 2 的冪值作為 PG 的數量，這樣可以保證將來重新分配的時候，PG 更容易均勻分佈。

其中，`OSD` 是集群中 OSD 的數量，`OSD_TARGET_PG_NUM` 是每個 OSD 的目標 PG 數量，廣泛接受的經驗最佳實踐值是 100，`pool_num` 是存儲池的數量，`size` 是副本數。

例如我的集群每個節點 5 個 SSD，一共 4 個節點，副本數是 4，有4個存儲池，那麼：

```
PG = (4 * 5 * 100) / (2 * 4) = 250 ～= 256 (2^8)
```

偷懶的話也可以直接訪問 PVE 的 Ceph 管理介面，找到 `Optimal # of PGs` 值，直接寫到對應的存儲池的 PG 數配置上。

這裡額外說一下幾個常用的配置：

- PG Auto Scale：自動調整 PG 數量，根據集群的負載自動調整 PG 數量，建議打開。
- Target Ratio: 目標佔比，也就是當前存儲池會自動擴展，一直達到佔用整集群空間的目標佔比為止。
- Target Max PG: 目標最大 PG 尺寸，就是當 PG 中的數據變超過這個尺寸，將會自動進行重分配到新的 PG 上。
- Min. # of PGs: 這個存儲池最少佔用多少個 PG，默認為 0。

### 關於 CRUSH map

幾乎所有對於 Ceph 集群中所有重要的配置都可以通過修改 CRUSH map 來實現，它控制數據的分配策略，你可以在 PVE 的 Ceph 管理介面上看到你的 CRUSH map 配置。

你可以放心修改你的 CRUSH map，Ceph 會根據新的 CRUSH map 重新平衡數據的分佈，整個過程是在線完成的，無需停機，不影響業務，而且效率很高。

所以，其實你可以隨意往集群裡面添加刪除 OSD，然後修改 CHUSH map 讓數據自動平衡，這是我最喜歡 Ceph 的地方，比任何磁盤陣列都方便，任何磁盤陣列做這類操作都無法做到這麼平滑。



修改 CRUSH map 的基本步驟：

- 獲得 CRUSH map 的二進制文件：

```
# ceph osd getcrushmap -o crushmap.compiled
```
- 解碼 CRUSH map 文件：

```
$ crushtool -d crushmap.compiled -o crushmap.text
```

- 修改 CRUSH map 文件：

```
$ vim crushmap.text
```

- 編碼 CRUSH map 文件：

```
$ crushtool -c crushmap.text -o crushmap.compiled.new
```

- 測試新的 CRUSH map 二進制文件（可選）：

```
# crushtool -i crushmap.compiled.new --test --show-X
```

- 應用新的 CRUSH map 二進制文件：

```
# ceph osd setcrushmap -i crushmap.compiled.new
```

一些常用的 CRUSH map 配置：

- `min_size`：最小副本數，默認為 1，這個設置優先級低於存儲池的配置。
- `max_size`：最大副本數，默認為 10，這個設置優先級低於存儲池的配置。
- `step take default class [class-name]`：優先選擇指定類型的 OSD 類如 `hdd`，`ssd`，`nvme` 等，這個對於需要分開不存儲池，例如機械池和 SSD 池的場景非常有用。

### 網絡對於 Ceph 集群至關重要


簡單總結，Ceph 完全是軟件定義數據中心概念的典範，你完全可以先根據自己的喜好來創建初始存儲池，然後根據不斷變化的業務需求中，不斷調整集群的配置，包括冗余，PG 數，介質類型，數量等等，它都會自動根據新的配置重新組織物理數據，這個過程相當平滑和安全。



#
