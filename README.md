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






#
