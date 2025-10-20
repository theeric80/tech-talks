# 精通 Go 單元測試：從核心實踐到高階模式

對於有經驗的軟體工程師而言，單元測試不僅是品質保證的手段，更是維持程式碼健康度、提升開發效率與保障重構安全的關鍵實踐。本文將深入探討 Go 語言中單元測試的核心觀念與高階技巧，旨在提供一份乾淨、專業且具深度的實戰指南。

## TL;DR

- **隔離是核心**：單元測試應專注於單一功能，與外部依賴（網路、資料庫）徹底解耦，確保測試的快速與穩定。
- **善用內建工具**：遵循 Go 的 `_test.go` 慣例與 `testing` 套件，並以 `AAA (Arrange, Act, Assert)` 模式構建清晰的測試結構。
- **擁抱表格驅動**：採用表格驅動測試（Table-Driven Tests）來優雅地管理多組測試案例，提升可讀性與可擴充性。
- **精通依賴處理**：利用介面（Interface）實現依賴注入（Dependency Injection），並在測試中使用測試替身（Test Doubles）如 Stub 或 Mock，達成測試隔離。
- **AI 協作**：將 AI 視為輔助工具，透過提供清晰的上下文（Context）與團隊規範（Guideline），加速測試案例的產出，但開發者仍是最終的品質守門員。

---

## 單元測試的價值：不僅是找 Bug

在軟體測試金字塔中，單元測試構成了穩固的基石。一個擁有良好單元測試覆蓋的專案，其價值體現在：

- **品質內建**：在開發週期的最早階段捕捉錯誤，降低修復成本。
- **加速迭代**：自動化測試讓我們能自信地進行變更，無需手動回歸測試的繁瑣。
- **保障重構**：當需要優化或調整既有實作時，完善的測試是確保系統行為一致性的安全網。
- **即時文件**：清晰的測試案例是函式預期行為與邊界條件的最佳說明。

在 Go 的世界裡，「單元」通常指一個函式（Function）或方法（Method）。單元測試的核心精神在於 **隔離 (Isolation)**，專注於驗證單一功能的邏輯正確性，並將其與外部世界（如資料庫、網路請求、檔案系統）徹底分離。

### 核心原則：隔離、快速、穩定

一個優質的單元測試應具備三個特質：
1.  **執行快速**：毫秒級的反應速度，讓開發者能無負擔地頻繁運行。
2.  **結果穩定 (Deterministic)**：無論何時何地，測試結果都應保持一致，不受外部環境影響。
3.  **完全隔離 (Isolated)**：測試應與外部依賴（資料庫、網路、檔案系統）徹底解耦。

## Go 的測試慣例與基礎

Go 語言原生提供了強大且簡潔的 `testing` 套件，讓撰寫測試成為一種直覺。

### 命名與結構

- **檔案命名**：測試檔案必須以 `_test.go` 結尾。
- **函式簽名**：測試函式以 `Test` 開頭，並接收 `*testing.T` 作為參數。

一個結構良好的測試遵循 **AAA (Arrange, Act, Assert)** 模式，將邏輯劃分為三個清晰的階段：

1.  **Arrange (安排)**：初始化變數、設定輸入與預期結果。
2.  **Act (執行)**：呼叫待測試的函式。
3.  **Assert (斷言)**：驗證執行結果是否與預期相符。

```go
// sum.go
func Sum(a, b int) int {
    return a + b
}

// sum_test.go
import "testing"

func TestSum(t *testing.T) {
    // Arrange
    a, b := 1, 2
    expected := 3

    // Act
    result := Sum(a, b)

    // Assert
    if result != expected {
        t.Errorf("Sum(%d, %d) = %d; expected %d", a, b, result, expected)
    }
}
```

雖然 Go 原生的 `if result != expected { t.Errorf(...) }` 可行，但在複雜斷言下顯得冗長。引入 `testify/assert` 能讓斷言更具語意化且簡潔。

```go
import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestSumWithTestify(t *testing.T) {
    // Arrange
    a, b := 1, 2
    expected := 3

    // Act
    result := Sum(a, b)

    // Assert
    assert.Equal(t, expected, result, "Sum(1, 2) should be 3")
}
```

## 進階技巧：表格驅動測試 (Table-Driven Tests)

當需要驗證多組輸入與輸出時，表格驅動測試是 Go 社群公認的最佳實踐。它將測試案例組織在一個 struct 切片中，透過迴圈執行，極大提升了測試的可讀性與維護性。

結合 `t.Run` 建立子測試，能讓每個案例在測試報告中獨立顯示，精準定位問題。

```go
func TestSumTableDriven(t *testing.T) {
    testCases := []struct {
        name     string // 測試案例名稱
        a, b     int    // 輸入
        expected int    // 期望的輸出
    }{
        {"Positive numbers", 1, 2, 3},
        {"Negative numbers", -1, -2, -3},
        {"Mixed numbers", -1, 1, 0},
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            result := Sum(tc.a, tc.b)
            assert.Equal(t, tc.expected, result)
        })
    }
}
```

## 處理依賴：介面與測試替身

真實世界的程式碼充滿了外部依賴。為了達成測試隔離，我們利用 Go 的 **介面 (Interface)** 來實現依賴注入（Dependency Injection），並在測試時傳入一個 **測試替身 (Test Double)**。

與其讓程式碼直接依賴具體的 `http.Client` 或 `sql.DB`，不如讓它依賴一個我們定義的介面。

假設我們有一個 `UserProfileFetcher`，它依賴一個 `HTTPGetter` 介面來獲取資料，而非具體的 `http.Client`。

```go
// user_profile_fetcher.go
type HTTPGetter interface {
    Get(url string) (*http.Response, error)
}

type UserProfileFetcher struct {
    client HTTPGetter
}

func (f *UserProfileFetcher) GetUserName(userID string) (string, error) {
    // ... 實作邏輯 ...
}
```

在測試中，我們可以建立一個 `StubHTTPClient` 來模擬 `HTTPGetter` 的行為，它不發送真實網路請求，而是回傳預設的回應。

```go
// user_profile_fetcher_test.go
type StubHTTPClient struct {
    StatusCode int
    Body       string
}

func (s *StubHTTPClient) Get(url string) (*http.Response, error) {
    r := io.NopCloser(bytes.NewReader([]byte(s.Body)))
    return &http.Response{StatusCode: s.StatusCode, Body: r}, nil
}

func TestGetUserName_Success(t *testing.T) {
    // Arrange: 準備一個成功回應的 Stub
    stub := &StubHTTPClient{
        StatusCode: http.StatusOK,
        Body:       "Bob",
    }
    fetcher := &UserProfileFetcher{client: stub}

    // Act
    name, err := fetcher.GetUserName("1")

    // Assert
    assert.NoError(t, err)
    assert.Equal(t, "Bob", name)
}
```

這種模式將 `UserProfileFetcher` 的核心邏輯與網路依賴完全解耦，使測試變得快速、穩定且獨立。

### Stub vs. Mock

- **Stub** 用於 **狀態驗證 (State Verification)**：提供固定的「罐頭」回應，讓測試可以驗證函式的回傳狀態是否正確。
- **Mock** 用於 **行為驗證 (Behavior Verification)**：除了提供回應，它還會記錄與驗證互動過程，例如「某個方法是否被以特定參數呼叫了指定的次數」。這在驗證沒有回傳值的函式時特別有用。

## 測試策略與覆蓋率

一個務實的測試策略，是將精力集中在刀口上：

- **測試目標**：優先測試公開（Public）的函式與方法、核心商業邏輯，以及各種邊界條件（`nil`、空值、負數等）。
- **衡量標準**：測試覆蓋率（`go test -cover`）是一個有用的指標，它能幫助我們發現未被測試覆蓋的程式碼路徑。但切忌將其作為唯一目標。100% 的行數覆蓋不等於 100% 的邏輯覆蓋。使用 `go tool cover -html=coverage.out` 產生的視覺化報告，能更直觀地分析覆蓋情況。

## AI 輔助測試：提升效率的現代方法

AI 工具（如 GitHub Copilot）能加速測試撰寫，但前提是我們必須提供高品質的**上下文 (Context)** 與**指引 (Guideline)**。

一個與 AI 高效協作的流程如下：

1.  **建立指引**：在專案中建立一份 `TESTING_GUIDELINES.md`，將團隊的測試共識（如 AAA、表格驅動、斷言庫使用）標準化。
2.  **精準提示**：在 Prompt 中清晰地提供「待測程式碼」、「測試指引」與「具體任務」，並明確要求需要覆蓋的測試場景。
3.  **嚴格審核**：**AI 產出的程式碼永遠只是草稿**。開發者必須扮演 **審核者 (Reviewer)** 的角色，仔細檢查其邏輯正確性、案例完整性（特別是邊界條件）以及程式碼風格。

### 進階應用：利用 AI 進行系統性測試分析

對於邏輯複雜、充滿多重條件判斷的函式，可要求 AI 扮演「測試分析師」的角色，協助系統性地探索所有邏輯分支與邊界條件。

與其直接要求產出測試碼，不如先讓 AI 分析函式並產出「決策表 (Decision Table)」。

**Prompt 範例：**
>
> **Task**: Act as a test analyst. Analyze the following function and generate a **Decision Table** that maps all input conditions to their expected outcomes.
>
> **Context**:
> ```go
> // (貼上一個邏輯複雜的函式)
> ```

透過這種方式，AI 會產出一份結構化的決策表，列出所有可能的條件組合及其對應的預期結果。這份表格不僅能幫助我們在動手前就識別出被忽略的測試情境，更可直接作為撰寫表格驅動測試的藍圖，將測試設計從「直覺驅動」提升到「系統化分析」的層次。

## 結論

在 Go 中撰寫優質的單元測試，是一門結合了原則與實用模式的技藝。透過掌握內建工具、表格驅動測試與依賴注入等技巧，我們可以打造出穩健、可維護的程式碼。同時，將 AI 工具視為強大的輔助，能在遵循團隊規範的前提下，顯著提升開發效率，讓我們在面對日益複雜的系統時，依然能充滿信心。

---

## Q&A

**Q: 表格驅動測試中的子測試 (`t.Run`) 是並行執行的嗎？**

預設為循序執行。若要啟用並行，需在 `t.Run` 的閉包函式內明確呼叫 `t.Parallel()`。Go 的測試運行器會自動管理這些標記為並行的子測試，以最大化執行效率。

**Q: 測試應深入到 Private 方法嗎？**

不建議。單元測試的核心是驗證「公開行為 (Public Behavior)」，而非「實作細節 (Implementation Detail)」。應專注於測試模組的公開 API（在 Go 中指大寫開頭的函式/方法）。當公開方法被測試時，其內部使用的私有方法自然會被間接驗證。直接測試私有方法會導致測試與實作過度耦合，使重構變得脆弱。

**Q: 如果一個共用的 `helper` 函式被多處使用，需要為它寫獨立測試嗎？**

取決於其複雜度。如果它包含關鍵的業務邏輯、解析或複雜演算法，為其建立獨立的、專注於邊界條件的測試是明智的投資。反之，如果只是簡單的格式轉換，則可由上層測試覆蓋。

**Q: 如果物件 A 依賴我們自己開發的物件 B，測試 A 時需要 Mock B 嗎？**

這是一個典型的權衡。
- **不 Mock (使用真實物件)**：當物件 B 的建立與行為相對簡單且穩定時，使用真實物件能讓測試更貼近整合情境。
- **Mock (使用測試替身)**：當物件 B 的初始化過程複雜、或依賴其他你不想在 A 的測試中引入的資源（如資料庫連線）時，就應該使用測試替身。這能確保測試 A 的焦點完全放在 A 的邏輯上，並保持測試的獨立與快速。

經驗法則是：**只 Mock 那些會讓測試變慢或不穩定的依賴**（如網路、資料庫）。如果物件 B 穩定且初始化簡單，直接使用真實物件。如果 B 很複雜或有自己的依賴，就用測試替身將其隔離。

**Q: 在 `Get` 方法的測試中，使用 `Put` 方法來準備前置狀態，這是否違反了測試獨立性？**

這是一個關於「測試狀態設定」的設計權衡。
- **優點**：透過公開 API (`Put`) 來設定狀態，使測試與底層儲存實作解耦。未來若內部資料結構改變，只要 `Put` 行為不變，測試就無需修改。
- **缺點**：`Get` 的測試依賴於 `Put` 的正確性。若 `Put` 存在 Bug，`Get` 的測試也可能失敗，增加問題定位的複雜度。

**結論**：這種做法在 `Get` 和 `Put` 是緊密綁定的公開 API（如 KV store）時是可接受的。關鍵在於權衡：是選擇與公開行為綁定，還是與內部實作細節綁定。

**Q: 測試覆蓋率 (Coverage) 的意義是什麼？100% 就夠了嗎？**

覆蓋率是個有用的「指標」，但不該是「目標」。100% 的行覆蓋率不等於 100% 的邏輯覆蓋。與其追求數字，不如專注於確保所有關鍵業務邏輯、複雜分支和邊界條件都已被驗證。

**Q: `testify/mock` 如何驗證函式呼叫的順序？**

`testify/mock` 的設計哲學是驗證「狀態 (State)」而非「順序 (Sequence)」。它擅長驗證某個方法是否被呼叫、呼叫次數及參數是否正確。

刻意驗證函式間的呼叫順序通常被視為反模式，因為這會讓測試與實作細節過度耦合，稍有重構就可能導致測試失效。如果函式執行的順序至關重要，這可能意味著該行為應由更高層級的整合測試來驗證，或者該單元的職責過於複雜，需要進一步拆分。

**Q: 當 AI 產生的測試不符預期時，該手動修改還是優化 Prompt？**

這是在「短期效率」與「長期投資」之間的權衡。
- **優化 Prompt/Guideline**：將其視為一次改善系統的機會。如果 AI 的錯誤源於指引的模糊或缺失，完善 `TESTING_GUIDELINES.md` 是一項長期投資，能提升未來所有 AI 產出程式碼的品質，並惠及整個團隊。
- **手動修改**：對於一次性的、或描述起來複雜但修改起來簡單的錯誤，直接手動編輯顯然更務實高效。

**結論**：兩者都要。把優化 Prompt/Guideline 看作是修復一個系統級的 Bug，這是一項長期投資。對於簡單的一次性錯誤，直接手動修改更有效率。目標是找到最高效的解決路徑。

**Q: 如何與團隊協作，維護給 AI 的 `TESTING_GUIDELINES.md`？**

將 `TESTING_GUIDELINES.md` 納入版本控制，視為團隊的共享資產。透過 Code Review 和日常討論持續迭代，讓它成為凝聚團隊共識、沉澱最佳實踐的動態文件。這不僅能確保 AI 產出的程式碼品質穩定，也能促進團隊內部在測試標準上達成一致。