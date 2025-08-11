

# **XCP-on-Ethernet：現代汽車嵌入式系統中的作用綜合分析**

## **I. XCP-on-Ethernet 簡介**

### **XCP（通用測量與校準協議）的定義與演進**

XCP（eXchange Control Protocol），即通用測量與校準協議，是一個標準化的協議，旨在實現對嵌入式系統內部記憶體的讀寫存取。該協議已作為一種通用測量協議確立多年，並允許在運行時對嵌入式系統中的演算法進行參數化。其主要應用是測量和校準內部電子控制單元（ECU）變數。該協議能夠以與ECU內部進程「事件同步」的方式記錄測量資料，從而確保資料之間的一致性 1。

XCP協議最初起源於汽車工業，由ASAM（自動化和測量系統標準化協會）工作組於2003年標準化 1。儘管其根源在汽車領域，但XCP在所有涉及嵌入式系統的領域都非常有用，例如航空航太、運輸或醫療行業 1。XCP是CAN校準協議（CCP）的繼任者，旨在將測量和校準功能擴展到CAN以外的各種傳輸介質 3。XCP名稱中的「X」代表其可變且可互換的傳輸層，強調其與匯流排系統的獨立性 1。

XCP從CCP演進而來，以支援「其他匯流排系統」 3。XCP名稱中的「X」明確表示其「可變且可互換的傳輸層」 1。這不僅僅是漸進式的更新，而是一種根本性的架構轉變。這種設計選擇表明ASAM及其成員（包括汽車原始設備製造商、供應商和工具生產商 8）採取了積極主動的策略，旨在建立一個能夠適應快速發展的車載網路環境的協議。這種協議能夠滿足對更高頻寬和多樣化通訊介質（如乙太網、FlexRay、USB等）的需求，超越了CAN的限制。這種前瞻性對於在複雜、異構的車輛架構中保持標準化和效率至關重要。

### **乙太網作為XCP傳輸層的重要性**

乙太網是汽車領域中XCP特別相關的傳輸層，與CAN、CAN FD、FlexRay和SxI（SPI、SCI）並列 12。採用乙太網作為傳輸層，滿足了現代ECU和車輛網路對更高資料速率日益增長的需求，這主要由先進駕駛輔助系統（ADAS）、資訊娛樂系統和複雜控制演算法所驅動 10。XCP-on-Ethernet利用乙太網基礎設施的廣泛可用性和高性能，使其適用於車載和測試台環境 8。

XCP的核心目的是「測量和校準內部ECU變數」以及「運行時演算法參數化」 1。這個過程是迭代的，旨在「加速開發過程」 1。乙太網的高頻寬使得「兆位元組的資料」傳輸成為可能 10，並實現「高資料速率和微秒級的短測量週期時間」 17。乙太網為XCP通訊提供的資料量和速度，不僅僅是增量改進，更是實現更複雜和快速開發週期的根本推動力。它促進了即時除錯、複雜的旁路情境 10，以及將高保真感測器資料 18 直接整合到校準迴路中。這將開發從一個較慢、需要大量重新編譯的過程轉變為一個動態、互動式且資料豐富的環境，直接影響ECU軟體優化的速度和品質。

## **II. XCP 的核心原則與架構**

### **主從通訊模型**

XCP採用單主機/多從機概念實現 1。主機通常是測量和校準工具，例如電腦應用程式（如Vector CANape）、資料記錄器或基於PC的模擬 1。從機則整合在嵌入式系統中，最常見的是ECU 1。一個主機可以同時連接多個從機，但一個從機一次只能有一個主機 1。這種「軟主從原則」允許從機在建立通訊通道後自主發送某些訊息（事件、服務請求、資料採集訊息） 19。

現代車輛具有日益分散和複雜的ECU架構 18。主從模型中，單個主機可以同時與「多個」從機通訊 1，為管理這種分散式複雜性提供了集中控制點。這對於跨多個互連ECU進行連貫的測量和校準至關重要。主機充當協調者，確保跨各種控制單元的資料一致性和同步操作，這對於整體系統優化至關重要。這種架構不僅是一種通訊模式，更是一種設計理念，簡化了開發過程中複雜多ECU系統的管理。它允許對分散式嵌入式系統網路進行單一、統一的視圖和控制，這對於現代車輛中高效的系統級校準和診斷至關重要。

### **兩層協議設計：協議層與傳輸層分離**

XCP始終將協議層與傳輸層分離 1。協議層定義了通用的測量和校準功能，獨立於底層網路類型 1。傳輸層則定義了XCP如何透過特定網路類型傳輸，例如CAN、FlexRay、乙太網（TCP/IP和UDP/IP）、USB和串行介面（SxI：SPI、SCI） 1。這種模組化設計允許XCP擴展到新的傳輸層，例如XCP-on-FlexRay於2005年首次亮相就證明了這一點 8。

協議與傳輸層的分離意味著核心XCP邏輯（命令、資料處理等）在任何通訊匯流排上都保持一致 3。這顯著減少了ECU製造商的實施工作，因為他們只需為每個新的匯流排開發一個傳輸層特定的驅動程式，而無需重新發明一個全新的測量和校準協議。這也意味著在整個開發過程中始終使用「相同的機制、工具、描述等」，減少了「工作步驟之間的劇變」以及購買/培訓「更少工具」的需求 1。兩層架構是一種戰略設計選擇，在快速發展的汽車領域提供了卓越的敏捷性。它允許行業採用新的、更高頻寬的通訊技術（如乙太網），而無需徹底改造整個測量和校準生態系統。這大大降低了開發成本，加速了工具的採用，並確保了從模擬到車載除錯等不同車輛平台和開發階段的一致性。這真正實現了其名稱所暗示的「通用」協議。

### **ASAM 標準化的作用（ASAM MCD-1 XCP、ASAM MCD-2 MC、A2L 文件）**

XCP被標準化為「ASAM MCD-1 XCP」 2。ASAM在定義ECU的獨立於匯流排的通訊協議方面發揮著關鍵作用 5。ASAM MCD-1 XCP以記憶體位址導向的方式存取參數和測量變數 1。這些資料的屬性和記憶體位址在A2L文件格式中描述，該格式透過ASAM MCD-2 MC進行標準化 1。

A2L文件包含存取和正確解釋傳輸資料所需的所有資訊，無需將對特定參數或變數的存取硬編碼到ECU應用程式中 3。這意味著XCP主機的不同配置可以執行不同的校準和測量任務，而無需重新編譯ECU應用程式碼 5。這種機制從根本上將軟體開發（ECU程式碼）與校準和測量配置分離。工程師只需更新A2L文件和主機工具的配置，即可修改參數和採集資料，而無需為每次更改重新編譯和刷新ECU韌體 1。這顯著加速了演算法優化和系統調優的迭代過程。這種分離是高效汽車開發的基石。它實現了軟體工程師和校準工程師的並行工作流程，減少了依賴性和瓶頸。它還促進了「即插即用」配置 1，因為主機可以查詢可用的XCP功能並動態解釋資料結構，從而縮短了設定時間並提高了測試環境的靈活性。

### **基本操作：位址導向的讀寫存取與最小資源消耗**

XCP採用位址導向的工作方式，對記憶體中物件的讀寫存取基於位址的指定 1。XCP的一個主要設計目標是在嵌入式系統（ECU）中實現最小的資源消耗（RAM、ROM、運行時），高效通訊，簡單的從機實施，少量參數的即插即用配置以及可擴展性 1。這種精簡的實施允許XCP在從8位微控制器（用於CAN/SCI）到利用FlexRay或乙太網的高性能平台等多種平台上使用 3。

ECU是資源受限的環境（有限的CPU、RAM、快閃記憶體）。在實現「高效通訊」和「高資料吞吐量」 5 的同時，將ECU資源影響降至最低，這是一個重大的工程挑戰。設計原則明確優先考慮這種平衡，使XCP能夠廣泛應用於各種ECU能力，從簡單的8位微控制器到複雜的32位處理器 3。這種在高性能資料交換和低ECU開銷之間仔細的平衡對於XCP作為通用協議的成功至關重要。它確保協議不會成為目標系統的瓶頸或不必要的負擔，從而允許ECU的主要功能（控制演算法）在提供穩健的測量和校準能力的同時最佳運行。這種設計理念直接促進了協議的廣泛適用性和行業接受度。

## **III. XCP-on-Ethernet：技術深入探討**

### **TCP/IP 和 UDP/IP 作為傳輸層的利用**

XCP-on-Ethernet支援TCP/IP和UDP/IP兩種資料交換協議 5。UDP/IP通常用於非同步資料傳輸，特別是資料採集（DAQ）封包，在這種情況下，每個單獨封包的保證傳遞不如高吞吐量和最小開銷重要。然而，值得注意的是，使用非確認鏈路（如UDP/IP）可能無法保證主機設備接收到非同步發送的封包 23。TCP/IP提供可靠的、面向連接的通訊，具有保證傳遞和流量控制，適用於命令-回應序列（CTO）和快閃記憶體編程，其中資料完整性至關重要 25。XCPlite是一個XCP實施方案，針對使用巨型幀的XCP-on-Ethernet傳輸層（TCP或UDP）進行了優化 27。

TCP提供可靠性、排序和流量控制，而UDP提供速度和較低的開銷但沒有保證 25。兩者之間的選擇並非隨意，而是策略性的。對於同步資料採集（DAQ），當大量時間敏感資料被串流傳輸時，通常優先選擇UDP的較低開銷和較高速度，即使這可能意味著潛在的封包丟失，因為重點在於捕獲連續的串流而不是確保每個單個樣本都到達 30。對於校準參數更改或快閃記憶體編程等關鍵操作，TCP的保證傳遞對於防止資料損壞至關重要 25。這種雙協議支援反映了一種複雜的設計，允許XCP根據不同資料交換的特定可靠性和效能要求進行優化。它實現了網路資源的靈活高效利用，確保了測量的高吞吐量，同時維護了關鍵校準和編程任務的資料完整性。這種適應性是XCP-on-Ethernet在各種開發階段多功能性的關鍵。

### **XCP 封包結構：命令傳輸物件（CTO）和資料傳輸物件（DTO）**

XCP通訊區分命令傳輸物件（CTO）和資料傳輸物件（DTO） 1。

* **CTO（命令傳輸物件）**：用於主機向從機交換命令，以及從機向主機回應和訊息。這包括命令封包（CMD）、命令回應封包（RES）、錯誤封包（ERR）、事件封包（EV）和服務請求封包（SERV）。主機發送的CMD必須始終由RES或ERR回應 1。  
* **DTO（資料傳輸物件）**：用於交換同步測量（DAQ）和刺激（STIM）資料 1。

XCP封包格式由一個識別欄位、一個可選的時間戳欄位和一個資料欄位組成。這個結構獨立於傳輸協議 1。識別欄位（PID）明確識別封包的類型和內容。對於DTO，它識別DAQ列表和ODT（物件描述表） 1。時間戳欄位（可選）存在於DTO封包中，包含從機在資料傳輸時的內部時鐘值 1。這對於重新排序封包和確保時間相關性至關重要 28。資料欄位包含CTO的特定參數或DTO的實際測量/刺激值 1。

CTO和DTO的明確分離允許對控制邏輯和資料流進行不同的處理，從而優化各自的目的。DTO中包含可選的時間戳欄位 1 尤其重要。這意味著從機（ECU）可以在採集資料時對其進行時間戳，將其直接連結到ECU內部事件 1。這種內部時間戳克服了網路傳輸延遲或可變匯流排負載引起的不確定性 10，確保了高度精確的「事件同步」測量。這種精細的封包結構，特別是時間戳功能，對於XCP提供對ECU行為的真實即時洞察力至關重要。它使工程師能夠準確地將外部測量與內部ECU進程相關聯，這對於診斷複雜的時間相關問題和高精度優化控制演算法是不可或缺的。

### **通訊模式：標準、塊和交錯通訊**

XCP支援多種通訊模式以優化資料傳輸：

* **標準通訊模式**：主機發送請求並等待從機回應 1。  
* **主機塊傳輸模式（可選）**：主機可以連續發送多個命令，無需等待單個回應，從而加速大資料傳輸（例如，上傳/下載） 1。  
* **從機塊傳輸模式**：從機可以連續向主機發送多個幀 1。  
* **交錯通訊模式（可選）**：主機可以連續發送多個請求，同時考慮從機的接收緩衝區大小，從機可以無限制地發送多個回應。此模式不應與塊模式同時使用 1。

這些模式不僅是傳輸資料的不同方式，它們還針對不同的操作場景進行了優化。標準模式確保了控制的嚴格請求-回應。塊模式（主機和從機）旨在透過最小化確認開銷來最大化大資料傳輸（如快閃記憶體編程或記憶體轉儲）的吞吐量 1。交錯模式允許並行操作，提高了複雜測量和校準會話中的響應能力 1。這些不同的通訊模式的可用性使得XCP能夠根據手頭任務的特定需求調整其資料傳輸策略。這種靈活性對於最大化各種ECU開發活動的效率至關重要，從微調單個參數到刷新整個韌體映像，確保乙太網底層頻寬的最佳利用。

### **同步資料交換：資料採集（DAQ）和資料刺激（STIM）機制**

* **DAQ（資料採集）**：資料從從機發送到主機，與ECU內部事件（例如任務執行、計時器訊號）同步 1。DAQ列表（總結測量變數）分配給選定的事件。物件描述表（ODT）定義了RAM內容如何打包到匯流排幀中 1。DAQ列表可以是靜態的、預定義的或動態配置的，其中動態配置在ECU開發中是首選 1。XCP允許在資料從ECU記憶體傳輸到發送緩衝區時對採集到的資料進行時間戳 1。  
* **STIM（資料刺激）**：主機向從機發送資料，與從機中的事件同步。這對於向控制單元提供輸入資料非常有用，例如用於旁路演算法或輸入記錄的原始感測器資料 1。

DAQ和STIM的結合促進了「旁路」 7，其中部分ECU演算法在外部、更強大的計算平台（例如，運行Simulink模型的PC 9）上執行。ECU透過DAQ將資料發送到模型，模型處理後，結果透過STIM發送回ECU。這創建了一個閉環開發環境。這種能力實現了快速控制原型（RCP）和複雜的硬體在環（HIL）和軟體在環（SIL）測試 1。它允許工程師快速迭代控制演算法，測試新功能，並在實際環境中驗證系統行為，而無需為每次更改重新刷新ECU。這顯著壓縮了開發時間表，並提高了最終軟體的品質。它還支援虛擬化開發環境的趨勢，減少了早期階段對實體原型的依賴。

## **IV. XCP-on-Ethernet 的優勢**

### **高速資料傳輸與頻寬利用**

乙太網提供比傳統汽車匯流排（如CAN的1 Mbps和FlexRay的10 Mbps）顯著更高的資料傳輸速率，其中汽車乙太網每節點提供100 Mbps，每幀資料大小可達1500 B，而CAN為8 B，FlexRay為254 B 16。XCP-on-Ethernet可以使用32位處理器透過高速網路發送「兆位元組的資料」 10。這種高頻寬對於即時應用、大規模資料記錄以及處理來自先進感測器（LIDAR、攝影機）的大量資料至關重要 2。它允許最佳利用可用頻寬，而不會影響正常的車輛通訊 10。

這種能力直接支援ADAS、自動駕駛和先進資訊娛樂系統等新興汽車技術的資料需求。這些功能產生大量的原始感測器資料（LIDAR、攝影機、雷達），並需要高解析度測量和快速校準調整 16。傳統匯流排（如CAN和FlexRay）根本不足以應對這些資料量 16。XCP-on-Ethernet不僅僅是一個更快的協議，它更是下一代資料密集型汽車功能的推動者。它處理大量資料的能力以及高速傳輸的能力，對於支撐這些先進功能的複雜演算法的開發、測試和驗證至關重要，直接促進了現代智慧汽車的可行性和性能。

### **增強的效率與可擴展性**

XCP旨在實現高效通訊和可擴展性，能夠適應從簡單應用到需要大量資料處理的複雜系統 1。其驅動程式大小可以透過包含強制和可選功能進行調整，以優化可用的ROM/快閃記憶體大小 10。XCP-on-Ethernet，特別是透過塊通訊模式，顯著加速了記憶體上傳和下載 28。

這種可擴展性不僅適用於資料量，也適用於ECU的內部資源佔用。為XCP驅動程式選擇強制和可選功能的能力 10 意味著即使是資源受限的8位微控制器也可以實現基本的XCP從機，而高性能ECU則可以充分利用乙太網的潛力 3。這種適應性確保了XCP可以部署在車輛內的各種ECU上，從簡單的感測器節點到複雜的域控制器，而不會對任何特定硬體造成過度開銷。這種優化資源分配策略使XCP成為一個真正通用的協議。它避免了「一刀切」的陷阱，允許根據不同ECU的計算和記憶體限制進行量身定制的實施，同時仍然提供統一的測量和校準介面。這有助於提高車輛開發和生產的整體系統效率和成本效益。

### **即時性能能力**

XCP的主要任務包括ECU內部變數的即時測量和校準 14。它確保了即時應用和快速校準調整所需的高速資料傳輸 2。其一大優勢是能夠從RAM中採集與ECU內部進程或事件同步變化的測量資料，允許使用者推斷時間序列與變化值之間的直接關聯 1。XCP支援時間戳資料傳輸，這對於主機重新排序接收到的封包並確保資料一致性至關重要，尤其是在高吞吐量場景中 1。

儘管乙太網提供高頻寬，但它本質上不如FlexRay或CAN等時間觸發匯流排具有確定性，尤其是在複雜的網路拓撲中 16。XCP透過實施內部ECU時間戳來解決這種潛在的非確定性，用於採集到的資料 1。這意味著資料的時間相關性在測量時於「ECU內部」建立，在網路傳輸之前。這減輕了可變網路延遲（抖動）對時間關鍵資料分析準確性的影響 10。這種設計選擇是解決將高速但可能不那麼確定的網路（如乙太網）用於即時嵌入式系統的根本挑戰的複雜解決方案。透過確保資料的時間一致性在源頭得到保留，XCP-on-Ethernet即使在網路延遲和抖動可能存在問題的環境中，也能提供高度準確和可靠的即時洞察力。這對於安全關鍵應用和精確演算法調優至關重要。

### **統一介面與簡化開發流程**

XCP的一個顯著優勢是單一標準化協議滿足所有應用需求，無需為每個通訊通道開發專有驅動程式 2。相同的機制、工具和描述文件（A2L）可以在整個開發過程中使用，從模擬環境到硬體在環（HIL）和嵌入式系統 1。這減少了工具的擴散、培訓工作和手動配置錯誤 1。

在複雜的汽車開發中，不同的團隊和階段（例如，模型在環、軟體在環、硬體在環、車輛測試）通常使用不同的工具和介面。XCP的通用性旨在彌合這些差距，確保測量和校準配置（如A2L文件中定義的）在整個V模型開發週期中是可移植且一致的 1。這最大限度地減少了「工作步驟之間的劇變」 1，並減少了因手動資料傳輸或工具不相容而導致錯誤的可能性。這種由XCP統一介面實現的一致工具鏈方法是汽車軟體開發效率和品質的強大推動力。它加速了迭代迴圈，簡化了跨不同環境的除錯，並最終縮短了新車輛功能的上市時間。這代表著向真正整合的開發生態系統邁出了重要一步。

### **支援大容量資料（例如快閃記憶體編程）**

XCP支援ECU中快閃記憶體的編程，這對於更新韌體和控制演算法至關重要 2。乙太網的高頻寬使其特別適用於快閃記憶體編程等應用，因為它處理大量資料的速度比傳統匯流排快得多 31。快閃記憶體編程通常包括編程前管理（例如，檢查軟體版本）、執行（寫入新內容）和後處理（例如，校驗和檢查） 7。

快閃記憶體編程涉及將可能兆位元組的資料傳輸到ECU 10。在CAN等較慢的匯流排上，這個過程可能非常耗時，阻礙了開發過程中的快速迭代。乙太網優越的頻寬大大縮短了快閃記憶體編程所需的時間，使得在開發、測試甚至預生產車輛中頻繁進行韌體更新成為可能。這對於具有大型軟體映像的複雜ECU尤其重要。透過XCP-on-Ethernet加速快閃記憶體編程的能力直接影響了開發週期的速度。它允許工程師快速部署和測試新軟體版本、錯誤修復或演算法增強功能，從而縮短每次迭代的周轉時間。這在敏捷開發環境中是一個關鍵優勢，其中持續整合和頻繁更新很常見。

## **V. 主要應用與用例**

### **ECU 校準與參數化**

XCP的主要應用是調整ECU的內部參數並獲取內部變數的當前值 1。工程師可以調整參數（標量、曲線、映射）並優化性能以實現所需的系統行為 1。更改可以透過XCP主機的使用者介面直接在ECU的RAM中快速完成，避免了每次更改都進行重新編譯和重新刷新 1。

現代ECU演算法高度複雜，涉及眾多參數之間錯綜複雜的關係 18。在運行時動態調整這些參數，而無需中斷ECU的操作或需要完整的軟體構建週期，對於優化控制策略至關重要。這使工程師能夠觀察系統對參數變化的即時響應，無論是在實際環境還是模擬環境中，從而促進性能、燃油效率、排放和安全方面的微調 2。這種動態校準能力將優化過程從靜態的離線練習轉變為互動式的即時活動。這對於實現先進汽車功能所需的精確和優化行為至關重要，允許快速迭代並收斂到最佳控制參數。

### **測量與資料記錄（事件同步採集）**

XCP能夠以與ECU內部進程「事件同步」的方式記錄測量資料，確保資料一致性和時間相關性 1。它促進了高速資料記錄，從ECU採集即時資料用於性能分析和開發 2。測量資料可以從與ECU進程同步變化的RAM中採集，從而可以直接推斷時間序列與變化值之間的關聯 1。XCP可以將物理感測器資料整合到測量、校準和診斷（MCD）設定中，如QuantumX資料採集解決方案所示 18。

透過乙太網的高頻寬，以與ECU內部事件同步的方式採集資料的能力，可以收集大量精確時間相關的資料。這對於理解軟體執行、ECU內部變數和外部物理現象之間複雜的相互作用至關重要。透過XCP-on-Ethernet將物理感測器資料 18 與ECU內部資料整合，可以形成對車輛行為的真正整體視圖。這種同步資料融合能力對於除錯、性能分析和驗證複雜控制演算法來說是無價的。它使工程師能夠診斷細微的時間問題，識別意外的相互作用，並更深入地理解整個系統的物理特性，從而產生更穩健和優化的車輛功能。

### **診斷與故障排除**

XCP支援診斷功能，用於檢測和排除車輛系統中的問題 2。雖然UDS（統一診斷服務）主要用於發布後的診斷，但XCP提供輕量級診斷功能，主要用於開發期間的發布前階段 7。XCP-on-Ethernet可用於除錯已永久安裝在車輛中的ECU，透過VX1000硬體將除錯軟體連接到XCP從機 7。VX1000硬體允許多個XCP從機同時用於除錯、測量和調整，最大限度地利用ECU除錯介面 7。

透過XCP-on-Ethernet進行除錯，即使是對於永久安裝的ECU，也標誌著一個強大的轉變 7。傳統上，除錯可能需要與校準工具分離的專用除錯介面。XCP透過單一、穩健的物理介面整合了這些功能 7。這種統一的方法，特別是像VX1000這樣提供多個XCP從機以同時進行除錯、測量和調整的硬體 7，簡化了診斷工作流程。這種整合的除錯能力使得在實際車輛環境中進行故障排除更加高效和有效。工程師可以同時監控內部ECU狀態、調整參數和除錯程式碼，從而更快地識別和解決可能僅在特定操作條件下才會出現的複雜問題。這顯著增強了ECU開發的整合後驗證和精煉階段。

### **快速原型（MIL、SIL、HIL、RCP）與旁路**

XCP用於各種開發環境，包括模擬環境（MIL \- 模型在環）、硬體在環（HIL）、軟體在環（SIL）和快速控制原型（RCP） 1。

* **旁路**：透過同時使用同步資料採集（DAQ）和同步資料刺激（STIM）實現。ECU將訊號發送到PC上運行的模型（透過DAQ），模型處理後，結果發送回ECU（透過STIM） 7。  
* 這允許在旁路模式下測試ECU上的控制演算法 9。  
* 運行MCD工具（如CANape）的普通PC平台足以進行旁路和建模，為專用即時硬體提供了經濟高效的替代方案 10。

執行複雜旁路情境的能力，其中ECU邏輯的一部分被卸載到運行模擬模型的PC上，顯著加速了新控制功能的開發和驗證。這種操作可以在「普通PC平台」上完成 10，而無需昂貴的專用即時硬體 14，這使得先進原型設計的門檻降低。這降低了小型團隊或專案的進入障礙，並允許更廣泛的實驗和迭代。這種能力從根本上改變了快速原型設計的經濟效益和可及性。它將重點從昂貴的硬體設定轉移到靈活的軟體驅動解決方案，從而實現更快、更具成本效益的開發週期。這在開發的早期階段尤其有利，因為頻繁的更改和迭代很常見，最終導致更穩健和優化的控制演算法。

### **整合到測試台自動化中**

XCP用於ECU的功能和環境測試系統，以及各種汽車部件（例如內燃機、變速箱、氣候控制）的測試台 2。它能夠將任何感測器整合到測量、校準和診斷（MCD）設定中的決策和資料融合過程中 18。INCA軟體透過支援既定的ASAM介面（包括ASAP3協議和ASAM MCD-3 MC物件模型）與測試台自動化系統無縫整合 25。

測試台對於在受控、可重複條件下驗證ECU行為至關重要。XCP-on-Ethernet的高頻寬和即時功能允許同時採集內部ECU資料和外部物理感測器資料（例如來自QuantumX DAQ系統 18）。這種全面的資料採集能夠更深入地理解ECU與物理世界之間的相互作用，促進控制演算法的穩健驗證和優化。這種整合能力允許進行全面和自動化的測試，減少了早期階段對大量道路車輛測試的需求。它提供了一個受控環境，用於識別和解決問題，確保ECU在廣泛的操作條件下可靠運行，然後再整合到車輛中。

## **VI. XCP-on-Ethernet 實施的挑戰與考量**

### **即時性能與確定性**

儘管XCP-on-Ethernet提供高資料速率，但在標準乙太網網路中實現嚴格的即時確定性（保證時序）可能是一個挑戰，因為其固有的非確定性 16。網路擁塞、硬體性能不佳和缺乏封包優先級可能導致封包傳遞時間的可變性（抖動） 33。XCP旨在實現高資料速率和微秒級的短測量週期時間 17，但這可能會受到網路條件的影響。文件指出，XCP在共享鏈路（如CSMA乙太網或802.11無線網路）上可能表現不佳，因為單個佇列的排空速率是共享介質負載的函數 34。

這裡的核心挑戰是標準乙太網的高吞吐量與某些即時汽車應用所需的嚴格確定性之間固有的權衡。儘管XCP的內部時間戳 1 有助於減輕網路抖動對資料相關性的「影響」，但它並不能消除抖動本身。對於需要「透過網路」可預測、低延遲通訊的控制迴路（例如線控驅動），標準乙太網可能不足以滿足要求，除非有時間敏感網路（TSN）等額外機制，而這些機制在此處並未明確詳細說明XCP。這突顯了一個細微的限制，即傳輸層的特性可能會影響協議在某些要求苛刻的用例中的實際適用性。

### **延遲與抖動問題**

抖動是訊號傳輸和接收之間時間延遲的變化，通常由網路擁塞、硬體性能不佳或缺乏封包優先級引起 33。高延遲和低頻寬會增加抖動 33。封包丟失，需要重新傳輸，也會導致抖動 33。特定的網路控制器問題（例如Realtek NIC）可能會導致人為的丟失/抖動 35。解決方案包括服務品質（QoS）設定、限制高頻寬應用程式和使用抖動緩衝區 33。

乙太網可能存在延遲和抖動，影響即時性能 33。即使頻寬很高，資料傳輸的一致性也可能受到影響，這對於需要精確計時的即時測量和校準來說是一個關鍵問題。研究資料指出了各種原因（擁塞、硬體、封包丟失）和潛在的緩解策略，如QoS 33。這意味著在要求嚴格的環境中成功部署XCP-on-Ethernet需要仔細的網路工程和配置，而不僅僅是擁有一個乙太網埠。僅僅轉向乙太網並不能自動保證XCP的理想即時性能。底層網路基礎設施及其配置（例如，受管交換機、QoS實施、網路介面卡的選擇）成為確保高保真測量和校準所需穩定性和可預測性的關鍵因素。這增加了系統設計的複雜性，而這種複雜性在CAN等更簡單、更具確定性的匯流排系統中可能不那麼突出。

### **網路複雜性與配置**

向同質的全乙太網網路發展可以簡化節點之間的位址分配和資料傳輸 26。然而，異構網路（例如，橋接CAN FD和100BASE-T1）需要閘道器，這些閘道器是帶有內置交換機的強大ECU 26。XCP-on-FlexRay需要透過FIBEX文件了解FlexRay網路叢集 17。雖然這與乙太網沒有直接關係，但它表明存在傳輸層特定的配置複雜性。XCP通訊需要透過配置分配唯一的PDU-ID進行路由 28。XCP主機和從機配置需要保持一致，這通常透過A2L IF\_DATA部分生成來促進 28。

向汽車乙太網的過渡並不意味著立即全面取代現有的匯流排系統 16。車輛在可預見的未來可能會採用異構網路，將乙太網與CAN、FlexRay等結合 16。這需要複雜的閘道器功能來橋接不同的網路域，增加了整體系統複雜性和潛在的故障點 26。在這種混合環境中配置XCP需要仔細管理PDU-ID並確保A2L在潛在不同網路段之間的一致性。現代汽車架構的網路複雜性對XCP-on-Ethernet的部署提出了重大挑戰。儘管XCP的傳輸層獨立性提供了靈活性，但將其整合到混合網路中的實際情況意味著工程師必須應對閘道器設計、路由複雜性和跨多種網路技術的一致配置。這需要對網路架構和穩健配置管理實踐有更深入的理解。

### **安全漏洞與保護處理（「Seed\&Key」）**

XCP主要用於校準和開發，應在車輛出廠前停用或移除，因為存在潛在的攻擊面 9。XCP提供類似UDS的ECU解鎖功能，使用「Seed & Key」安全機制限制授權人員的存取 2。XCP的一般安全問題包括中間人攻擊、隱蔽資料通道和惡意來源/接收器 34。這些問題類似於TCP中的問題，但由於XCP的明確性，在XCP中實施可能更容易 34。為了解決重放、篡改和欺騙漏洞，建議對XCP協議安全性進行增強，包括時間戳和隨機數 36。

XCP的安全性至關重要，特別是隨著車輛越來越互聯並容易受到網路攻擊 36。儘管「Seed & Key」提供了基本的身份驗證機制 2，但研究資料強調了更複雜的威脅，如中間人攻擊、資料篡改和欺騙 34。建議在生產後停用或移除XCP 9 凸顯了其在部署的車輛中暴露時固有的風險。提及建議的增強功能（時間戳、隨機數）表明正在努力加強協議以應對不斷變化的網路威脅 36。由於XCP-on-Ethernet促進了對ECU內部的更深層次存取，它固有地擴大了攻擊面。安全考量不僅限於簡單的存取控制，還包括資料完整性、真實性以及針對複雜網路級攻擊的保護。這需要在整個車輛生命週期中制定強大的安全策略，包括安全實施、嚴格測試和生產車輛中開發介面的仔細管理。XCP安全功能的持續演進反映了汽車網路安全領域的動態性質。

## **VII. 與其他汽車通訊協議的比較分析**

### **XCP-on-Ethernet 與 CAN 和 FlexRay（頻寬、延遲、確定性）**

| 特徵/標準 | CAN (Classic/FD) | FlexRay | XCP-on-Ethernet |
| :---- | :---- | :---- | :---- |
| **主要目的/應用** | 骨幹、車身、底盤、動力總成、分散式控制 | 高性能動力總成、安全、主動底盤控制、線控驅動 | 診斷、ECU編程、資訊娛樂、高頻寬測量與校準 |
| **典型速度/頻寬** | 1 Mbps (Classic), 8 Mbps (FD) 16 | 10 Mbps 16 | 100 Mbps (每節點) 16 |
| **資料負載大小** | 8 B (Classic), 64 B (FD) 16 | 254 B 16 | 1500 B 16 |
| **拓撲** | 匯流排 16 | 匯流排/星形/混合 16 | 星形/樹形/環形 (10BASE-T1S支援多點) 16 |
| **確定性** | 非固有，使用仲裁位 16 | 是，使用時分多址 (TDMA) 16 | 標準乙太網非固有，XCP內部時間戳緩解抖動 1 |
| **成本** | 低 ($$) 16 | 較高 ($$$) 16 | 中等 ($$)，交換機增加成本 16 |
| **冗餘** | 無 16 | 有 16 | 協議層非固有 16 |

這些協議並非簡單的替代品，而是通常扮演互補的角色。CAN對於簡單系統的分散式控制仍然具有成本效益和穩健性 16。FlexRay為安全關鍵、時間觸發的應用提供高確定性和容錯性 16。乙太網憑藉其高頻寬，非常適合資料密集型任務，如診斷、快閃記憶體編程和感測器資料傳輸 2。行業正在向異構網路發展，其中這些協議共存，通常透過閘道器橋接 26。這意味著現代車輛中的分層網路架構，其中每個匯流排系統都根據其在成本、頻寬、確定性和安全要求之間的最佳權衡進行選擇。XCP的傳輸層獨立性使其能夠利用每個匯流排的優勢進行測量和校準，適應不同ECU域或開發階段的特定需求。對於XCP而言，乙太網為複雜資料提供了必要的吞吐量，而CAN/FlexRay可能仍用於低頻寬或更具確定性的車載通訊。

### **XCP-on-Ethernet 與 UDS、DoIP 和 SOME/IP（目的、應用、協議棧）**

| 特徵/標準 | XCP-on-Ethernet | UDS (ISO 14229-2) | DoIP (ISO 13400-2) | SOME/IP |
| :---- | :---- | :---- | :---- | :---- |
| **主要目的/應用** | 開發期間測量、校準、快閃記憶體編程、除錯 2 | ECU快閃記憶體編程與診斷操作 7 | 透過IP網路傳輸UDS訊息 9 | 域控制器間資料交換 9 |
| **典型使用階段** | 開發期間 (發布前)，通常生產後停用 7 | 發布後，整個車輛生命週期 7 | 發布後，與UDS結合使用 9 | 正常車輛運行期間 9 |
| **傳輸層** | TCP/IP 或 UDP/IP 5 | ISO-TP (CAN), 或 DoIP/HSFZ (乙太網) 9 | TCP/IP 或 UDP/IP (乙太網/WLAN) 9 | IP (乙太網) 9 |
| **主從/服務導向** | 單主機/多從機 1 | 應用層協議，客戶端-伺服器 9 | 閘道器-終端點模型 9 | 服務導向，訂閱/通知 9 |
| **安全機制** | 「Seed & Key」用於存取限制 2 | 依賴應用層，DoIP可使用TLS 9 | 不指定，委託給應用層 9 | 易受網路攻擊 9 |

這項比較突顯了車輛生命週期中職責的明確劃分。XCP是「開發主力」，在生產前階段為工程師提供深入的即時存取 7。UDS（通常透過DoIP在乙太網上分層）是服務技術人員在發布後使用的「服務和診斷協議」 7。SOME/IP是車輛正常運行期間用於車載通訊的「操作資料交換協議」 9。每個協議都針對其特定階段和功能要求進行了優化，從校準所需的高度侵入性存取到現場診斷和操作資料的穩健、安全通訊。這種功能專業化避免了功能膨脹，並優化了每個協議以實現其預期目的，從而產生了更高效、更安全的汽車系統。儘管所有這些協議都可以利用乙太網的頻寬，但它們獨特的角色意味著它們不能直接互換，而是構成了車輛設計、開發和操作壽命期間的全面通訊生態系統。

## **VIII. 行業格局：工具與供應商**

### **主要軟體工具概述**

業界領先的軟體工具廣泛支援XCP，這表明XCP不僅是一個標準，更是汽車開發生態系統的基礎元素。

* **Vector CANape/CANoe**：CANape是測量資料採集和ECU校準的主要工具。它在運行時調整參數值並採集測量訊號，支援所有標準化傳輸協議上的XCP，包括乙太網 10。CANoe則透過XCP擴展了ECU記憶體存取能力 12。Vector在XCP標準化中發揮了關鍵作用 4。  
* **ETAS INCA**：用於校準、診斷和驗證汽車電子系統的綜合解決方案。它支援透過TCP/IP的XCP連接通用模擬環境，並從ECU和車輛匯流排（如乙太網和FlexRay）採集資料 23。它還與測試台自動化系統整合 25。  
* **dSPACE ControlDesk**：用於設計dSPACE硬體的系統實施和介面、下載軟體以及與全域變數介面。它支援XCP以實現應用程式和ECU之間的高效資料交換 29。  
* **National Instruments (NI) ECU測量與校準工具包**：支援XCP校準協議規範1.0版，抽象化XCP通訊層以實現使用者透明，同時允許低層次存取 19。  
* **MathWorks Simulink® Real-Time™**：XCP介面允許Vector CANape或ETAS INCA等工具與Speedgoat即時目標機配合使用，進行校準和測量。它支援從Simulink® Coder™生成原生ASAP2文件 21。

這些工具提供了全面的功能，從低層次記憶體存取到高層次圖形介面，用於校準、測量和模擬整合。它們與XCP-on-Ethernet的相容性確保工程師可以在其首選開發環境中利用高頻寬通訊。與Simulink的整合 21 尤其重要，因為基於模型的設計是現代ECU開發的基石。這種強大的工具生態系統顯著降低了XCP-on-Ethernet採用的門檻。它為工程師提供了成熟、整合的解決方案，抽象化了大部分底層協議複雜性，使他們能夠專注於演算法開發和校準。這些供應商之間的競爭與合作也推動了XCP領域的持續改進和功能擴展，進一步鞏固了其作為行業標準的地位。

### **主要硬體解決方案**

專業硬體解決方案與軟體工具相輔相成，為深入的ECU存取提供了物理介面。

* **Vector VX1000 測量與校準硬體**：提供為控制器配備XCP-on-Ethernet介面的選項。隨插即用設備（POD）透過除錯介面（DAP、JTAG、Nexus、Aurora）直接連接到ECU，將資料傳輸到作為XCP從機的基礎模組，然後透過XCP-on-Ethernet將資料提供給XCP主機 7。它支援同時除錯、測量和調整 7。  
* **ETAS ETK（ECU測試與校準）設備**：某些型號提供標準XCP-on-Ethernet相容性，可與CANape等第三方工具配合使用，儘管ETAS專有工具通常用於完整功能 24。X-ETK和F-ETK設備可以透過標準XCP-on-Ethernet進行通訊 24。  
* **Speedgoat 即時目標機**：透過內置乙太網埠支援XCP主機和從機。I/O模組提供更大的靈活性，包括支援XCP-on-CAN 23。用於汽車行業的ECU校準和旁路 23。  
* **HBK QuantumX 資料採集解決方案**：可透過標準化的XCP-on-Ethernet協議整合到測量、校準和診斷（MCD）設定中，允許整合物理感測器資料 18。

這些專業硬體解決方案與軟體工具相輔相成，為深入的ECU存取提供了物理介面。它們實現了高保真資料採集和即時互動，這對於在實際條件下驗證複雜控制演算法至關重要。這種硬體與軟體的協同作用對於穩健高效的汽車開發至關重要。

## **結論**

XCP-on-Ethernet已成為現代汽車嵌入式系統開發不可或缺的協議，其核心優勢在於其通用性、高性能和對複雜開發流程的支援。作為ASAM標準化的協議，XCP從其前身CCP演進而來，透過其「X」所代表的可變傳輸層，展現了對未來網路技術的戰略性適應能力。這項設計決策不僅降低了開發開銷，也確保了工具鏈在從模擬到實體測試的整個V模型開發週期中的一致性。

XCP-on-Ethernet利用TCP/IP和UDP/IP的雙重支援，為不同類型的資料交換提供了量身定制的可靠性，確保了測量資料的高吞吐量和關鍵校準資料的完整性。其精細的封包結構，特別是內置的時間戳功能，為工程師提供了對ECU內部行為的精確即時洞察，即使在乙太網固有的非確定性環境中，也能透過內部時間戳緩解網路抖動的影響，確保資料的時間一致性。此外，DAQ和STIM機制共同實現了強大的閉環開發和旁路功能，推動了快速控制原型設計和虛擬化測試，從而加速了演算法迭代並降低了對昂貴專用硬體的依賴。

儘管XCP-on-Ethernet在頻寬和效率方面具有顯著優勢，但在實施過程中仍需應對挑戰。標準乙太網的即時性能和確定性問題，以及潛在的延遲和抖動，要求仔細的網路工程和配置。現代汽車網路的異構性也帶來了複雜性，需要透過閘道器來管理不同協議之間的互操作性。此外，由於XCP在開發過程中對ECU內部具有深層次存取權限，因此必須實施強大的安全措施（如「Seed & Key」），並在車輛生產後停用相關功能，以應對不斷演變的網路威脅。

與CAN、FlexRay、UDS、DoIP和SOME/IP等其他汽車通訊協議相比，XCP-on-Ethernet在車輛生命週期的開發階段扮演著獨特的、不可或缺的角色。它專注於高頻寬的測量、校準和韌體編程，與用於診斷和操作資料交換的協議形成互補。領先的軟體工具（如Vector CANape、ETAS INCA、dSPACE ControlDesk）和專業硬體解決方案（如Vector VX1000、HBK QuantumX）的廣泛支援，共同構建了一個強大而成熟的XCP生態系統，進一步鞏固了其在汽車嵌入式系統開發中的核心地位。總體而言，XCP-on-Ethernet是實現現代汽車複雜控制系統高效、高保真開發和驗證的關鍵推動因素。

#### **引用的著作**

1. XCP The standard protocol for the embedded Development \- Vector, 檢索日期：8月 8, 2025， [https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP\_Book\_V1.5\_EN.pdf](https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP_Book_V1.5_EN.pdf)  
2. XCP protocol | Learn about Universal Calibration Protocol \- Automotive Vehicle Testing, 檢索日期：8月 8, 2025， [https://automotivevehicletesting.com/measurement-calibration/xcp-protocol/](https://automotivevehicletesting.com/measurement-calibration/xcp-protocol/)  
3. XCP (protocol) \- Wikipedia, 檢索日期：8月 8, 2025， [https://en.wikipedia.org/wiki/XCP\_(protocol)](https://en.wikipedia.org/wiki/XCP_\(protocol\))  
4. XCP Fundamentals: Measuring, Calibrating and Bypassing Based on the ASAM Standard, 檢索日期：8月 8, 2025， [https://www.youtube.com/watch?v=uEmgbCXij-w](https://www.youtube.com/watch?v=uEmgbCXij-w)  
5. SOLUTIONS GUIDE ASAM \- ASAM eV, 檢索日期：8月 8, 2025， [https://www.asam.net/fileadmin/Solutions\_Guides/ASAM\_SG\_20170720\_2.pdf](https://www.asam.net/fileadmin/Solutions_Guides/ASAM_SG_20170720_2.pdf)  
6. CCP / XCP on CAN Explained \- A Simple Intro \[2025\] \- CSS Electronics, 檢索日期：8月 8, 2025， [https://www.csselectronics.com/pages/ccp-xcp-on-can-bus-calibration-protocol](https://www.csselectronics.com/pages/ccp-xcp-on-can-bus-calibration-protocol)  
7. Simtestlab, 檢索日期：8月 8, 2025， [https://simtestlab.se/xcp-protocol](https://simtestlab.se/xcp-protocol)  
8. XCP – The Standard Protocol for ECU Development \- Vector, 檢索日期：8月 8, 2025， [https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP\_ReferenceBook\_V3.0\_EN.pdf](https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP_ReferenceBook_V3.0_EN.pdf)  
9. Automotive-specific Documentation, 檢索日期：8月 8, 2025， [https://scapy.readthedocs.io/en/latest/layers/automotive.html](https://scapy.readthedocs.io/en/latest/layers/automotive.html)  
10. XCP at the Focal Point of Measurement and Calibration Applications \- Vector, 檢索日期：8月 8, 2025， [https://cdn.vector.com/cms/content/know-how/\_technical-articles/XCP\_UseCases\_ElektronikAutomotive\_200705\_PressArticle\_EN.pdf](https://cdn.vector.com/cms/content/know-how/_technical-articles/XCP_UseCases_ElektronikAutomotive_200705_PressArticle_EN.pdf)  
11. XCP Protocol | Sorion technologies, 檢索日期：8月 8, 2025， [https://www.sorion-group.com/support/technologies/xcp-protocol/](https://www.sorion-group.com/support/technologies/xcp-protocol/)  
12. XCP | Measurement and Calibration Protocol \- Vector, 檢索日期：8月 8, 2025， [https://www.vector.com/gb/en/know-how/protocols/xcp-measurement-and-calibration-protocol/](https://www.vector.com/gb/en/know-how/protocols/xcp-measurement-and-calibration-protocol/)  
13. XCP | Measurement and Calibration Protocol \- Vector, 檢索日期：8月 8, 2025， [https://www.vector.com/at/en/know-how/protocols/xcp-measurement-and-calibration-protocol/](https://www.vector.com/at/en/know-how/protocols/xcp-measurement-and-calibration-protocol/)  
14. XCP Focus for Calibration and Measurement Applications ... \- EEWorld, 檢索日期：8月 8, 2025， [https://en.eeworld.com.cn/news/Test\_and\_measurement/eic262262.html](https://en.eeworld.com.cn/news/Test_and_measurement/eic262262.html)  
15. XCP Use Cases \- Vector, 檢索日期：8月 8, 2025， [https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP\_Use\_Cases.pdf](https://cdn.vector.com/cms/content/application-areas/ecu-calibration/xcp/XCP_Use_Cases.pdf)  
16. What Is Can Bus (Controller Area Network) \- Dewesoft, 檢索日期：8月 8, 2025， [https://dewesoft.com/blog/what-is-can-bus](https://dewesoft.com/blog/what-is-can-bus)  
17. ASAM MCD-1 XCP \- Wiki, 檢索日期：8月 8, 2025， [https://www.asam.net/standards/detail/mcd-1-xcp/wiki/](https://www.asam.net/standards/detail/mcd-1-xcp/wiki/)  
18. Optimize Vehicle Testing using XCP-on-Ethernet | HBK \- HBKWorld.com, 檢索日期：8月 8, 2025， [https://www.hbkworld.com/en/knowledge/resource-center/recorded-webinars/optimize-vehicle-testing-using-xcp-on-ethernet](https://www.hbkworld.com/en/knowledge/resource-center/recorded-webinars/optimize-vehicle-testing-using-xcp-on-ethernet)  
19. Universal Measurement and Calibration Protocol (XCP) Overview \- NI \- National Instruments, 檢索日期：8月 8, 2025， [https://www.ni.com/docs/en-US/bundle/ecu-measurement-calibration-toolkit/page/universal-measurement-and-calibration-protoco.html](https://www.ni.com/docs/en-US/bundle/ecu-measurement-calibration-toolkit/page/universal-measurement-and-calibration-protoco.html)  
20. XCP \- Universal (X) Measurement and Calibration (C) Protocol (P). \- DEV Community, 檢索日期：8月 8, 2025， [https://dev.to/arvind\_b\_n\_luxoft/xcp-universal-x-measurement-and-calibration-c-protocol-p-1f0m](https://dev.to/arvind_b_n_luxoft/xcp-universal-x-measurement-and-calibration-c-protocol-p-1f0m)  
21. XCP Protocol for ECU Measurement and Calibration \- MATLAB & Simulink \- MathWorks, 檢索日期：8月 8, 2025， [https://www.mathworks.com/discovery/xcp-protocol.html](https://www.mathworks.com/discovery/xcp-protocol.html)  
22. XCP and CCP in LabVIEW (and X-NET) \- Hardware \- LAVA, 檢索日期：8月 8, 2025， [https://lavag.org/topic/20685-xcp-and-ccp-in-labview-and-x-net/](https://lavag.org/topic/20685-xcp-and-ccp-in-labview-and-x-net/)  
23. XCP Master & Slave Support for Simulink Real-Time | Speedgoat, 檢索日期：8月 8, 2025， [https://www.speedgoat.com/products-services/i-o-connectivity/protocols/xcp-over-ethernet](https://www.speedgoat.com/products-services/i-o-connectivity/protocols/xcp-over-ethernet)  
24. How to Use an ETK Device in CANape \- Vector Support, 檢索日期：8月 8, 2025， [https://support.vector.com/sys\_attachment.do?sys\_id=135e25102bd06610b847f49dbe91bf3c](https://support.vector.com/sys_attachment.do?sys_id=135e25102bd06610b847f49dbe91bf3c)  
25. ETAS INCA software products, 檢索日期：8月 8, 2025， [https://www.etas.com/ww/en/products-services/data-acquisition-processing-tools/software-products/inca-software-products/](https://www.etas.com/ww/en/products-services/data-acquisition-processing-tools/software-products/inca-software-products/)  
26. Why use 10BASE-T1S instead of CAN? | Keysight Blogs, 檢索日期：8月 8, 2025， [https://www.keysight.com/blogs/en/tech/2024/02/8/how-is-10base-t1s-different-from-can](https://www.keysight.com/blogs/en/tech/2024/02/8/how-is-10base-t1s-different-from-can)  
27. vectorgrp/XCPlite: Simple implementation of the ASAM XCP on Ethernet protocol \- GitHub, 檢索日期：8月 8, 2025， [https://github.com/vectorgrp/XCPlite](https://github.com/vectorgrp/XCPlite)  
28. Requirements on Module XCP AUTOSAR CP R21-11, 檢索日期：8月 8, 2025， [https://www.autosar.org/fileadmin/standards/R21-11/CP/AUTOSAR\_SRS\_XCP.pdf](https://www.autosar.org/fileadmin/standards/R21-11/CP/AUTOSAR_SRS_XCP.pdf)  
29. How to use the XCP protocol with RTMaps \- YouTube, 檢索日期：8月 8, 2025， [https://www.youtube.com/watch?v=-HDaMlOr3hM](https://www.youtube.com/watch?v=-HDaMlOr3hM)  
30. ASAM \- XCP \- Part2 Protocol Layer Specification \- V1 1 0 | PDF ..., 檢索日期：8月 8, 2025， [https://www.scribd.com/document/485213044/ASAM-XCP-Part2-Protocol-Layer-Specification-V1-1-0](https://www.scribd.com/document/485213044/ASAM-XCP-Part2-Protocol-Layer-Specification-V1-1-0)  
31. Guide to the ISO13400 Protocol \- UDS on automotive DoIP protocol \- Embien Technologies, 檢索日期：8月 8, 2025， [https://www.embien.com/automotive-insights/automotive-insights/guide-to-the-iso13400-protocol-uds-on-automotive-doip-protocol](https://www.embien.com/automotive-insights/automotive-insights/guide-to-the-iso13400-protocol-uds-on-automotive-doip-protocol)  
32. CAN Bus Explained \- A Simple Intro \[2025\] \- CSS Electronics, 檢索日期：8月 8, 2025， [https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial](https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial)  
33. Network Jitter \- Common Causes and Best Solutions | IR, 檢索日期：8月 8, 2025， [https://www.ir.com/guides/what-is-network-jitter](https://www.ir.com/guides/what-is-network-jitter)  
34. draft-falk-xcp-spec-03 \- Datatracker \- IETF, 檢索日期：8月 8, 2025， [https://datatracker.ietf.org/doc/html/draft-falk-xcp-spec-03](https://datatracker.ietf.org/doc/html/draft-falk-xcp-spec-03)  
35. Why you have packet loss/jitter/missdelivery on CS2 and how to resolve the issue (Recently discovered fault with Realtek Ethernet Controllers) : r/GlobalOffensive \- Reddit, 檢索日期：8月 8, 2025， [https://www.reddit.com/r/GlobalOffensive/comments/1gsbgvk/why\_you\_have\_packet\_lossjittermissdelivery\_on\_cs2/](https://www.reddit.com/r/GlobalOffensive/comments/1gsbgvk/why_you_have_packet_lossjittermissdelivery_on_cs2/)  
36. Security Analysis and Improvement of Vehicle Ethernet SOME/IP Protocol \- PMC, 檢索日期：8月 8, 2025， [https://pmc.ncbi.nlm.nih.gov/articles/PMC9503536/](https://pmc.ncbi.nlm.nih.gov/articles/PMC9503536/)  
37. dSpace Solutions for Control Tutorial, 檢索日期：8月 8, 2025， [https://my.ece.utah.edu/\~ece3510/dSpace%20Tutorial%2001172007.pdf](https://my.ece.utah.edu/~ece3510/dSpace%20Tutorial%2001172007.pdf)
